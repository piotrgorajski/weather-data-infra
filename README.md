# weather-data-infra
Set up local infra for weather data related components

## Prerequisites

### Docker
1. Install Docker engine

### Java
1. Install Java 11
2. Set JAVA_HOME
3. Add JAVA_HOME/bin to PATH

### Spark
1. Download Spark 3.5.3 for Hadoop 3 and Scala 2.12 (https://spark.apache.org/downloads.html)
2. Set SPARK_HOME
3. Add SPARK_HOME/bin to PATH

### Hadoop
1. From https://github.com/cdarlint/winutils download:
    - winutils.exe
    - hadoop.dll
2. Move files to C:\hadoop\bin
3. Set HADOOP_HOME
4. Add HADOOP_HOME/bin to PATH

## Run the infra

### Create docker network

A dedicated network is required to enable communication between docker containers.
Specifically between Spark application and Minio.

`docker network create spark-minio-network`

### Running Minio

#### First time
```
docker run -d --name minio --network spark-minio-network \
-p 9000:9000 -p 9001:9001 \
-e "MINIO_ROOT_USER=admin" \
-e "MINIO_ROOT_PASSWORD=password" \
minio/minio server /data --console-address ":9001"
```

#### Following times

`docker start minio`

### Triggering Spark Job

#### Prerequisites
- Define config required to access Minio:
    - spark.hadoop.fs.s3a.access.key = admin (matching Minio setup above)
    - spark.hadoop.fs.s3a.secret.key = password (matching Minio setup above)
    - spark.hadoop.fs.s3a.endpoint = http://minio:9000 (due to Docker network host should be minio instead of localhost, port matching Minio setup above)
- Docker image built considering:
    - Base spark image: spark:3.5.3-scala
    - Containing copied over uber JAR (but no spark, nor hadoop libs included)
    - With hadoop-aws, hadoop-common and aws-java-sdk-bundle downloaded and put under /opt/spark/jars/
    - SPARK_HOME/bin added to PATH

#### Docker command
```
docker run --rm -it --network spark-minio-network my-spark-app \
spark-submit \
--master local[*] \
--name weather-data-processor \
--class org.goraj.weatherapp.wdp.Entrypoint \
/app/weather-data-processor-assembly-0.1.jar -d 2020-07-10
```

#### Troubleshooting
Access the Spark container: `docker run --rm -it --entrypoint bash my-spark-app`

### Running Airflow
1. Use official Airflow guide: https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html
2. Extend airflow compose file with new mount to provide access to docker engine on host:
    `/var/run/docker.sock:/var/run/docker.sock`
3. Grant Airflow the permission to Docker daemon:
    - Unsafe: `sudo chmod 666 /var/run/docker.sock`
    - By adding Airflow User to Docker Group
4. Disable samples DAGs by setting AIRFLOW__CORE__LOAD_EXAMPLES to False

### (Optional) (Needs testing) Kubernetes cluster for Spark
1. Start Minikube. Spark requires more resources than Minikube's defaults:
    `minikube start --cpus=4 --memory=4g`
2. Enable Docker in Minikube:
    `eval $(minikube docker-env)`
3. (One time only) Create service account for spark:
    `kubectl create serviceaccount spark-sa`
4. (One time only) Grant service account a ClusterRole:
    `kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark-sa --namespace=default`
5. Submit spark application:
    ```
    ./bin/spark-submit \
    --master k8s://https://localhost:<port> \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=2 \
    --conf spark.kubernetes.container.image=spark:local \
    --conf spark.kubernetes.context=minikube \
    --conf spark.kubernetes.namespace=default \
    --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark-sa \
    local:///opt/spark/examples/jars/spark-examples_2.12-3.5.3.jar
    ```
6. Monitor the app:
   - Check running pods:
    `kubectl get pods`
   - View logs of the Spark driver:
    `kubectl logs <spark-driver-pod-name>`
   - Access Spark UI:
    `kubectl port-forward <driver-pod-name> 4040:4040`