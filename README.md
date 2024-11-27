# weather-data-infra
Set up local infra for weather data related components.

## Prerequisites

### Docker
1. Install Docker engine

### Minio

`docker compose -f minio/compose.yaml up -d`

### Base Spark image

#### Build

`make -f spark/Makefile build`

#### Push

`make -f spark/Makefile push`

#### Build & Push

`make -f spark/Makefile all`

### Airflow

#### Context
1. Use official Airflow guide: https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html
2. Extend airflow compose file with new mount to provide access to docker engine on host:
    `/var/run/docker.sock:/var/run/docker.sock`
3. Grant Airflow the permission to Docker daemon:
    - Unsafe: `sudo chmod 666 /var/run/docker.sock`
    - By adding Airflow User to Docker Group
4. Disable samples DAGs by setting AIRFLOW__CORE__LOAD_EXAMPLES to False
5. Update compose.yaml to point AIRFLOW_PROJ_DIR to Airflow repo that contains DAGs.

#### Run

`docker compose -f airflow/compose.yaml up -d`

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
