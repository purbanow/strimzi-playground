## SETUP STRIMZI

1. Install minikube
1. minikube start --memory=4096
1. kubectl create namespace kafka
1. kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
1. kubectl apply -f kafka.yaml -n kafka
1. kubectl apply -f topic.yaml -n kafka
1. kubectl apply -f user_test.yaml -n kafka
1. kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n kafka

## SETUP CLIENTS
1. kubectl get secret user-test --namespace=kafka -o yaml
1. Copy  `data.password`
1. Decode password `echo "data.password" | base64 --decode`
1. Create producer.properties
```
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-512
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
  username="user-test \
  password="<PASSWORD>";
```
1. Create consumer.properties
```
group.id=group-test
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-512
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
  username="user-test" \
  password="<PASSWORD>";
```
1. kubectl get kafka -o yaml -n kafka
1. Copy `bootstrapServers` from external type to use later for clients. 

### TEST without passing user credentials 
1. kafka-console-producer --broker-list  broker:port --topic topic-test
```
>[2021-07-16 10:34:23,217] WARN [Producer clientId=console-producer] Bootstrap broker 192.168.99.100:30741 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
```

### TEST with passing user credentials but wrong topic
1. kafka-console-producer --broker-list  broker:port --topic wrong-topic-test  --producer.config producer.properties
```
>message
[2021-07-16 10:35:48,788] WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 3 : {wrong-topic-test=TOPIC_AUTHORIZATION_FAILED} (org.apache.kafka.clients.NetworkClient)
[2021-07-16 10:35:48,790] ERROR [Producer clientId=console-producer] Topic authorization failed for topics [wrong-topic-test] (org.apache.kafka.clients.Metadata)
[2021-07-16 10:35:48,791] ERROR Error when sending message to topic wrong-topic-test with key: null, value: 7 bytes with error: (org.apache.kafka.clients.producer.internals.ErrorLoggingCallback)
org.apache.kafka.common.errors.TopicAuthorizationException: Not authorized to access topics: [wrong-topic-test]
```

### TEST with passing user credentials and proper topic
1. kafka-console-producer --broker-list  broker:port --topic topic-test  --producer.config producer.properties
```
>message1
>message2
>
```
1. kafka-console-consumer --bootstrap-server  broker:port --topic topic-test --from-beginning --consumer.config consumer.properties
```
message1
message2

```
