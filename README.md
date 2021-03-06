# SparkSQL ZTF Alerts Processing

## AWS EMR (ElasticMapReduce) Service

### Creating a cluster

Creating an EMR Cluster in AWS is a very simple process, you need to acces to https://console.aws.amazon.com/elasticmapreduce and be aware of the following parameters:

1. **Software Configuration Section**: select the spark application to be installed

``` 
Spark: Spark 2.4.5 on Hadoop 2.8.5 YARN and Zeppelin 0.8.2
```
2. **Hardware Configuration Section**: select instances of type m5a.4xlarge with 16 cores and 64 GB of RAM and try to compute the number of instances according to the size of the result. Each instance creates automatically a local HDFS of the same size of the RAM.

### Configuration (login with ec2-user)

In order to access the AWS S3 Bucket **s3://ztf-avro** where ZTF historical data is located you will need to configure in the master node:

1. **AVRO spark package**: add to **/etc/spark/conf/spark-default.conf** the following line

```
spark.jars.packages             org.apache.spark:spark-avro_2.11:2.4.5
```
If you want to use **ipython** to work interactively with **pyspark**, please follow the instructions:

1. Install ipython: ```sudo pip3 install ipython```
2. Edit .bashrc:

```
export PYSPARK_DRIVER_PYTHON=ipython3
export PYSPARK_PYTHON=python3
```
3. Execute: ```source .bashrc``` 
4. Execute **pyspark** and you will enter with **ipython** interactive mode.

## Processing ZTF Alerts

# Interactive mode with **pyspark** (login with hadoop)

You can execute **pypsark** using **ipython** as the Python driver. Inside **ipython** you can access ZTF historical data executing:

```
df = spark.read.format("avro").load("s3a://ztf-avro/*")
```
For further documentation on how to use SparkSQL and DataFrames please visit: https://spark.apache.org/docs/latest/sql-getting-started.html.

# Submitting a task with **spark-submit**

For sending a task to the cluster in terms of batch processing you could use **spark-submit**.

For example, for extracting metadata from all the ZTF historical data you can use: 

1. Bash Script: https://github.com/alercebroker/ztf_historic_data/blob/master/extract_detections.sh

2. Pyspark Script: https://github.com/alercebroker/ztf_historic_data/blob/master/extract_detections.py

It is worth noting that data is going to be store in the HDFS of the EMR Cluster in order to manage that information you can use the Hadoop command lines:

- Compute Size

```
hdfs dfs -du -h -s TARGET
```
- Remove Directory

```
hdfs dfs -rm -r TARGET
```
- Copy to local Directory

```
hdfs dfs -copyToLocal ORIGIN DESTINATION
```


# Use S3 for write output

If you want write your output in a bucket of S3, follow the next instructions:

1. Create a cluster with "Advance Options"a
2. Select Hadoop 1.10.x, Spark 2.4.x and Zeppelin 0.8.2. After that press Next button.
3. In the view of **Hardware** select your preference for the cluster. After that press Next button.
4. In the view of **General Cluster Settings** write a name for your cluster. We recommend you deselect Logging and Termination protection. In additional options select **EMRFS consistent view**. Press Next button.
5. Finally in the view of **Security** choose your pem key and enable Custom permissions. Press create cluster.

When the cluster start, follow the instruction of the [configuration](https://github.com/alercebroker/sparksql_alerts_processing#configuration-login-with-ec2-user).

For write in the bucket of S3:

```python
result.write.save("s3://<bucket_path>")
```
