FROM apache/spark:3.5.3

ARG HADOOP_VERSION=3.3.6
ARG AWS_SDK_VERSION=1.12.367

RUN wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/${HADOOP_VERSION}/hadoop-aws-${HADOOP_VERSION}.jar -P /opt/spark/jars/ && \
    wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-common/${HADOOP_VERSION}/hadoop-common-${HADOOP_VERSION}.jar -P /opt/spark/jars/ && \
    wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/${AWS_SDK_VERSION}/aws-java-sdk-bundle-${AWS_SDK_VERSION}.jar -P /opt/spark/jars/

ENV PATH="$SPARK_HOME/bin:$PATH"

# Minio credentials. Should match Minio config.
ENV AWS_ACCESS_KEY_ID="admin"
ENV AWS_SECRET_ACCESS_KEY="password"

ENTRYPOINT [ "spark-submit" , "--conf", "spark.hadoop.fs.s3a.endpoint=http://minio:9000", "--conf", "spark.hadoop.fs.s3a.path.style.access=true"]
