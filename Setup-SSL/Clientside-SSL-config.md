# Client configuration for using SSL

## grab CA & self signed certificate from remote server and add it to local CLIENT truststore

```
export CLIPASS=clientpass
cd /Users/joeqiao/Documents/LocalHub/kafka/kafka-security/Certs
keytool -keystore kafka.client.truststore.jks -alias CARoot -import -file ca-cert -storepass joeclientpass -keypass joeclientpass -noprompt
keytool -keystore kafka.client.truststore.jks -alias brokercert -import -file cert-signed -storepass joeclientpass -keypass joeclientpass -noprompt

keytool -list -v -keystore kafka.client.truststore.jks -storepass joeclientpass
```

## create client.properties and configure SSL parameters
security.protocol
ssl.truststore.location
ssl.truststore.password
==> use template [client.properties](./client.properties)

## TEST
test using the console-consumer/-producer and the [client.properties](./client.properties)
### Producer
```
/Users/joeqiao/Documents/LocalHub/kafka/kafka_2.13-2.8.1/bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic kafka-security-topic --create --partitions 1 --replication-factor 1

/Users/joeqiao/Documents/LocalHub/kafka/kafka_2.13-2.8.1/bin/kafka-console-producer.sh --broker-list localhost:9093 --topic kafka-security-topic --producer.config /Users/joeqiao/Documents/LocalHub/kafka/kafka-security/Setup-SSL/client.properties

/Users/joeqiao/Documents/LocalHub/kafka/kafka_2.13-2.8.1/bin/kafka-console-producer.sh --broker-list localhost:9093 --topic kafka-security-topic

export KAFKA_OPTS="-Djavax.net.debug=ssl -Djavax.net.ssl.trustStore=/Users/joeqiao/Documents/LocalHub/kafka/kafka-security/Certs/kafka.client.truststore.jks"
```
### Consumer
```
/Users/joeqiao/Documents/LocalHub/kafka/kafka_2.13-2.8.1/bin/kafka-console-consumer.sh --bootstrap-server localhost:9093 --topic kafka-security-topic --consumer.config /Users/joeqiao/Documents/LocalHub/kafka/kafka-security/Setup-SSL/client.properties
```
