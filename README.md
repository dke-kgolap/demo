# Big KG-OLAP Appendix

## Dataset

In our experiments we used generated datasets based on AIXM data model[^1]. We developed an AIXM data generator tool[^2] to allow us to generate a realistic dataset. The tool is based on the donlon[^3]. The provided link explains how to build and use the tool to generate the desired dataset. In our case, we used the configuration file provided[^4], we specify the start and end time stamp, storage location, and storage type.

```shell
start: "2000-01-01T00:00:00.000Z"
end: "2022-12-31T23:59:59.999Z"
outputOptions:
    mode: "path"
    path: "./files"
```

Once we the configuration is fixed you can run the generator:

```shell
$ java -jar app.jar
```

## Queries 

To evaluate our implementation, we design a set of queries that vary in result size and roll-up operations to verify the system response against different scenarios. In our implementation, we used an SPARQL-like query syntax for ease of use. In the following, we explain each query and the expected result of the cube.  

* **Q1**: creates a KG-OLAP cube of all locations and topics during the month of January 2000. The resulting cube contains 6975 contexts or 269700 quads.

```shell
SELECT time_month=2000-01
```

* **Q2**: creates a KG-OLAP cube of all locations and topics during the month of February 2000 and rolls up (aggregate) the results to all. The resulting cube contains 225 contexts or 197025 quads.

```shell
SELECT time_month=2000-02 ROLLUP ON time_all
```

* **Q3**: creates a KG-OLAP cube of all topics for the territory of France during the month of January 2000. The resulting cube contains 2790 contexts or 107880 quads.

```shell
SELECT time_month=2000-01 AND location_territory=France
```

* **Q4**: creates a KG-OLAP cube of all topics for the territory of France during the month of January 2000 and rolls up topic, location, and time to all. The resulting cube contains 1 context or 50220 quads.

```shell
SELECT time_month=2000-01 AND location_territory=France 
ROLLUP ON topic_all, location_all, time_all
```

* **Q5**: creates a KG-OLAP cube of the topic family *EnRoute* for the territory of France during the year 2000. The resulting cube contains 4392 contexts or 177144 quads.

```shell
SELECT time_year=2000 AND location_territory=France 
AND topic_family=EnRoute
```

* **Q6**: creates a KG-OLAP cube of the topic family *EnRoute* for the territory of France during the year 2000 and rolls up topic, location, and time to all. The resulting cube contains 1 contexts or 83448 quads.

```shell
SELECT time_year=2000 AND location_territory=France 
AND topic_family=EnRoute 
ROLLUP ON topic_all, location_all, time_all
```

* **Q7**: creates a KG-OLAP cube of the topic category *Routes* and the location FIR *LOVV* during the year 2000. The resulting cube contains 4392 contexts or 166164 quads.

```shell
SELECT time_year=2000 AND location_fir=LOVV 
AND topic_category=Routes
```

* **Q8**: creates a KG-OLAP cube of the topic category *Routes* and the location FIR *LOVV* during the year 2000 and is added from topic to category, location to location, and time to month. The resulting cube contains 36 contexts or 128304 quads.

```shell
SELECT time_year=2000 AND location_fir=LOVV 
AND topic_category=Routes 
ROLLUP ON topic_category, location_location, time_month
```

[^1]: <https://www.aixm.aero>
[^2]: <https://anonymous.4open.science/r/aixm-gen-F5E1>
[^3]: <https://github.com/aixm/donlon>
[^4]: <https://anonymous.4open.science/r/aixm-gen-F5E1/configs.yaml>
