# Big KG-OLAP Demonstrator

## Dataset

In our experiments we used generated datasets based on AIXM data model[^1], using the Donlon dataset and support tools[^3] as the basis. We developed an AIXM data generator[^2], which allows to generate a realistic dataset. The provided link to the AIXM data generator explains how to build and use the tool to generate the desired dataset. In our case, we used the configuration file provided[^4], we specify the start and end time stamp, storage location, and storage type.

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

### Input file

The data is generated based on a schema that can be found [here](dnotam-hierarchies.yaml), this file defines the dataset dimensions -- in this case, location, topic and time and their respective hierarchy.  

The file [20210101_EDDF_AeronauticalGroundLight_1](20210101_EDDF_AeronauticalGroundLight_1.xml) is an example of the message data. The file name is not used in the processing within KG-OLAP, however, for convenance it does indicate the time, location and the message type. The content of the file conforms to AIXM standard and envelops the contextual information as well as the actual information which is used for analytic purpose. More information about the structure and content of AIXM messafe can found [here](https://aixm.aero/page/aixm-51-specification).

### How to generate the dataset?

* Clone the repository data generator repository.
* Build the data generator (requirements to build are specified in the [README](https://anonymous.4open.science/r/aixm-gen-F5E1/README.md)):

```shell
$ ./build.sh
```

* Create a directory to store the dataset (default directory is files if you want to change it please change the configuration in config.yaml)
* Modify the configuration if necessary, default configuration to replicate our experiments are stored in [config.yaml](https://anonymous.4open.science/r/aixm-gen-F5E1/configs.yaml). We provided other examples in the repo. 
* Run the generator program:

```shell
$ java -jar app.jar
```

## KG Lakehouse

The source code for the KG Lakehouse is provided in this [repository](https://anonymous.4open.science/r/kgolap-4C60).

### How to build the KG Lakehouse? 

Please refer to [README](https://anonymous.4open.science/r/kgolap-4C60/README.md) to make sure you satisfy all the requirements. You also need make sure you have Docker installed and you have access to a Kubernetes cluster. The document also explains how to build the code and docker images, assuming you have all the regiments the build process should be straight forward.  Once you have build the docker images, you need to push them to [DockerHub](https://hub.docker.com/)

To deploy the KG Lakehouse service please refer to [Kubernetes yaml file](https://anonymous.4open.science/r/kgolap-4C60/k3s/kg-olap.yaml). You need to modify this file and use the correct image names as used when uploading the images to DockerHub. You also need to have access to a Kubernetes server and have [kubectl tool](https://kubernetes.io/docs/reference/kubectl/) the command to deploy the service is simply: 

```shell
$ kubectl apply file k3s/kg-olap.yaml
```

## Queries

To evaluate our implementation, we design a set of queries that vary in result size and roll-up operations to verify the system response against different scenarios. In our implementation, we used a SPARQL-like query syntax for ease of use. In the following, we explain each query and the expected result KG-OLAP cube.  

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

### Query result

A sample of the query result can be found in [SmallResult.nq](SmallResult.nq). The result is RDF quad format which we can use to run graph operation for analytics.  

[^1]: <https://www.aixm.aero>
[^2]: <https://anonymous.4open.science/r/aixm-gen-F5E1>
[^3]: <https://github.com/aixm/donlon>
[^4]: <https://anonymous.4open.science/r/aixm-gen-F5E1/configs.yaml>
