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
    `kubectl delete -f kafka.yaml -n kafka`

5- Wait for the Kafka, Zookeeper and other optional resources to be ready
6- Create a Topic:  
    `Kubectl apply –f my-topic.yaml -n kafka`

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
