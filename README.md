## Features:

**Supports multiple spark contexts.**

The project launches new processes in order to run and manage multiple spark contexts. Inter-process comunication is achieved with akka actors.

## Building Spark-job-rest (SJR)

The project is build with maven.
```
mvn clean install
```

## Configure Spark-job-rest

In order to configure SJR the following file need to be edited: resources/application.conf

* Configure the default spark properties for context creation
``` 
#spark default configuration
spark.executor.memory=2g
spark.mesos.coarse=false
#Default Spark Driver JVM memory
driver.xmxMemory = 1g
```
* Configure settings like web server port and akka system ports
```
#application configuration
appConf{
#the port on which to deploy the apis
web.services.port=8097
........
}
```

## Run Spark-job-rest

After editing all the configuration files SJR can be run by executing the script : start-server.sh

## API

**Contexts**

- POST /context/{contextName}  -  Create Context

 * Body:  Raw entity with key-value pairs. 
 * jars key is required and it should be in the form of a ':' separated list of jar paths. These jars will be added at Spark context creation time to the class path of the newly created context's JVM process.
  ``` 
 jars="/home/ubuntu/example.jar:/home/ubuntu/spark-job-project.jar”
 spark.executor.memory=2g
 driver.xmxMemory = 1g
  ```

- GET /context/{contextName}  -  returns Context exists. | No such context.

- DELETE /context/{contextName}  -  Delete Context

**Jobs**

- POST /job?runningClass={runningClass}&context={contextName}  - Job Submission 

  * Body:  Raw entity with key-value pairs. Here you can set any configuration properties that will be passed to the config parameter of the validate and run methods of the provided jar (see the SparkJob definition below)

- GET /job?jobId={uuid}&contextName={contextName} - Gets the result or status of a specific job

## Create Spark Job Project

Add maven Spark-Job-Rest dependency:
```
<dependency>
    <groupId>com.xpatterns</groupId>
    <artifactId>spark-job-rest</artifactId>
    <version>0.2.0</version>
</dependency>
```

To create a job that can be submitted through the server, the class must implement the SparkJob trait.

```
class Example extends SparkJob {
    override def runJob(sc:SparkContext, jobConfig: Config): Any = { ... }
    override def validate(sc:SparkContext, config: Config): SparkJobValidation = { ... }
}
```

- runJob method contains the implementation of the Job. SparkContext and Config objects are provided through parameters.
- validate method allows for an initial validation. In order to run the job return SparkJobValid(), otherwise return SparkJobInvalid(message).

