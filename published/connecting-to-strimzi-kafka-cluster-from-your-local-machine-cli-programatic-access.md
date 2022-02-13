
# Connecting to Strimzi Kafka Cluster from your Local Machine : CLI & Programatic Access

Why you need this ?

* For local development you want to connect to a remote Kafka Cluster running on OpenShift , that is deployed using Strimzi Operator

### Prerequisite

* OpenShift Container Platform or OKD

* Strimzi Operator deployed

### **Deploy Kafka Cluster**

* Create a YAML file with these contents (only for dev/test clusters)

    apiVersion: kafka.strimzi.io/v1beta2
    kind: Kafka
    metadata:
      name: my-cluster
      namespace: nestjs-testing
    spec:
      entityOperator:
        topicOperator: {}
        userOperator: {}
      kafka:
        config:
          inter.broker.protocol.version: "2.8"
          log.message.format.version: "2.8"
          offsets.topic.replication.factor: 3
          transaction.state.log.min.isr: 2
          transaction.state.log.replication.factor: 3
        listeners:
        - name: plain
          port: 9092
          tls: false
          type: internal
        - name: tls
          port: 9093
          tls: true
          type: internal
        - name: route
          port: 9094
          tls: true
          type: route
        replicas: 3
        storage:
          type: ephemeral
        version: 2.8.0
      zookeeper:
        replicas: 3
        storage:
          type: ephemeral

### Preparing to Connect

    oc get secret my-cluster-cluster-ca-cert -o jsonpath='{.data.ca\.crt}' | base64 -d > ca.crt

    keytool -import -trustcacerts -alias root -file ca.crt -keystore truststore.jks -storepass password -noprompt

    # This should create 2 files in PWD

    ls -l *.crt *.jks

### Grab Kafka Endpoint

    KAFKA_ENDPOINT=$(oc get kafka my-cluster -o=jsonpath='{.status.listeners[?(@.type=="route")].bootstrapServers}{"\n"}')

### Connecting from CLI (Kafka Console Producer/Consumer)

* Get Kafka Console Producer & Consumer script files

    wget [https://dlcdn.apache.org/kafka/3.0.0/kafka_2.13-3.0.0.tgz](https://dlcdn.apache.org/kafka/3.0.0/kafka_2.13-3.0.0.tgz) ; tar -xvf kafka_2.13-3.0.0.tgz

* Console Producer

    kafka_2.13-3.0.0/bin/kafka-console-producer.sh --broker-list $KAFKA_ENDPOINT --producer-property security.protocol=SSL --producer-property ssl.truststore.password=password --producer-property ssl.truststore.location=truststore.jks --topic my-topic

* Console Consumer

    kafka_2.13-3.0.0/bin/kafka-console-consumer.sh --bootstrap-server $KAFKA_ENDPOINT --topic my-topic --from-beginning  --consumer-property security.protocol=SSL --consumer-property ssl.truststore.password=password --consumer-property ssl.truststore.location=truststore.jks

### Connecting from Python Client (running locally)

```
from kafka import KafkaProducer, KafkaConsumer
import json
from bson import json_util

bootstrap_server = 'my-cluster-kafka-route-bootstrap-nestjs-testing.apps.ocp.ceph-s3.com:443'

print("Producing messages to Kafka topic ...")
producer = KafkaProducer(bootstrap_servers=bootstrap_server, ssl_cafile='ca.crt', security_protocol="SSL")

for i in range(10):
    message = {'value': i}
    producer.send('my-topic', json.dumps(message, default=json_util.default).encode('utf-8'))

print("Consuming messages from Kafka topic ...")

consumer = KafkaConsumer('my-topic',  group_id='my-group', bootstrap_servers=bootstrap_server, ssl_cafile='ca.crt', security_protocol="SSL", consumer_timeout_ms=10000, enable_auto_commit=True)
for message in consumer:
    # message value and key are raw bytes -- decode if necessary!
    # e.g., for unicode: `message.value.decode('utf-8')`
    print ("%s:%d:%d: value=%s" % (message.topic, message.partition,message.offset,message.value))
```

![Output of Kafka Python Producer & Consumer example](https://cdn-images-1.medium.com/max/2000/1*Is-Hc882G5EdnS4nb4vZFA.png)

> This is how you can connect to a remote Kafka cluster from your local machine. This is handy when you are developing locally and eventually deploy that to your OpenShift environment.
