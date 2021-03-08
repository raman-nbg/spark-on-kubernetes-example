1. Install [minikube](https://minikube.sigs.k8s.io/docs/start/)

2. Download and unzip [Spark with Hadoop 3.1.1](https://spark.apache.org/downloads.html)

3. Add spark to PATH
    ```
    source setup.sh
    ```

4. minkube brings its own docker daemon. We have to point our shell to minikube's docker daemon.
    ```
    eval $(minikube -p minikube docker-env)
    ```

5. Build Spark Docker Image
    ```
    $SPARK_HOME/bin/docker-image-tool.sh -r ramannbg -t v3.1.1-j11 -p kubernetes/dockerfiles/spark/bindings/python/Dockerfile -b java_image_tag=11-slim build
    ```

6. The Spark Driver needs to create (worker) pods at runtime. Therefore we need a service account 
that has these permissions. The Driver Pod must run with the privileges of the service account.
    ```
    kubectl apply -f spark.yaml
    ```

7. Submit Spark App
    ```
    spark-submit \
    --master k8s://https://127.0.0.1:55000 \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=5 \
    --conf spark.kubernetes.container.image=ramannbg/spark:v3.1.1-j11 \
    --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark-driver \
    local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar
    ```

Additional notes:
- The spark master must start with `k8s://` and should contain the the URI to the kubernetes master. You can get
the kubernetes master using `kubectl cluster-info`
- The spark app will be deployed to the default namespace. If you want to use another namespace you have to adopt the `spark.yaml`
and add the argument `--conf spark.kubernetes.namespace=<namespace>` to the `spark-submit`
- See [docs](https://spark.apache.org/docs/latest/running-on-kubernetes.html#configuration) for all spark configs on konfiguration