# kafka-on-kubernetes
This repository describes the steps and resources to deploy Kafka on Kubernetes using Strimzi.

**Demo-1: We will deploy the Strimzi operator and then the Kafka cluster resources.**(Use resources from _kafka-demo/demo1-strimzi-setup_ path)

1- Starting Minikube:  
    `minikube start --memory=4096`

2- Create Kafka namespace:   
    `kubectl create namespace kafka`

3- Deploy Strimzi Operator (including ClusterRole, ClusterRoleBinding and CRDs): 
    
`kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
    `

4- Now we can add a Kafka custom resource to deploy a Kafka Cluster:   
    `kubectl apply -f kafka.yaml -n kafka`

5- Wait for the Kafka, Zookeeper and other optional resources to be ready

6- Create a Topic:  
    `kubectl apply -f kafka-topic.yaml -n kafka`

7- Run Producer  
    `kubectl -n kafka run kafka-producer -ti --image=quay.io/strimzi/kafka:0.28.0-kafka-3.1.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --bootstrap-server kafka-kafka-bootstrap:9092 --topic my-topic`

8- Run Consumer    
    `kubectl -n kafka run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.28.0-kafka-3.1.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server kafka-kafka-bootstrap:9092 --topic my-topic --from-beginning`


**Demo-2: We will deploy the keda operator and use it to auto-scale the kafka consumers in case of load**Use resources from _kafka-demo/demo2-auto-scaling-using-keda_ path)

1- Create keda namespace: 
    `kubectl create namespace keda`

2- Deploy keda operator:

    a. helm repo add kedacore https://kedacore.github.io/charts
    b. helm repo update
    c. helm install keda kedacore/keda --namespace keda
3- Deploy Consumer application:

    a. kubectl apply -f consumer-service.yml -n keda
    b. kubectl apply -f consumer-deployment.yml -n keda
4- Deploy Keda scaled object:  
    `kubectl apply -f keda.yaml -n keda`

5- Run Kafka Producer:  
    `kubectl -n kafka run kafka-producer -ti --image=quay.io/strimzi/kafka:0.28.0-kafka-3.1.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --bootstrap-server kafka-kafka-bootstrap:9092 --topic my-topic`

6- Produce some messages and see if the consumer pods are getting scaled


**Demo-3: This demo is to show the Kafka cluster monitoring using Prometheus and Grafana**Use resources from _kafka-demo/demo-3 monitoring_ path)

1- Create monitoring namespace: 

    `kubectl create namespace monitoring`

2- Deploy Prometheus operator:

    `kubectl apply -f prometheus-operator-deployment.yaml -n monitoring --force-conflicts=true --server-side`

3- Create configmap for jmx metrics:

    kubectl apply -f kafka-metrics-config.yaml -n monitoring
    kubectl apply -f zookeeper-metrics.yaml -n monitoring

4- Update the Kafka resource with jmxPrometheusExporter to scrape the jmx metrics and kafkaExporter for exporting the topic and consumer lag metrics

    kubectl apply -f kafka.yaml -n kafka

5- Deploy Prometheus - 

    kubectl apply -f prometheus.yaml -n monitoring

6- Deploy pod monitor-

    kubectl apply -f strimzi-pod-monitor.yaml -n monitoring

7- Deploy Grafana - 

    kubectl apply -f grafana.yaml -n monitoring

8- Port forward for Prometheus and Grafana-

    kubectl port-forward svc/grafana 3000:3000 -n monitoring
    kubectl port-forward svc/prometheus-operated 9090:9090 -n monitoring

9- Add Prometheus datasource in Grafana
10 Upload grafana dashboards from- 
    https://github.com/strimzi/strimzi-kafka-operator/tree/0.28.0/examples/metrics/grafana-dashboards
