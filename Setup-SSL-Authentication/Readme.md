# on local computer
## create a CLIENT certificate !! put your LOCAL hostname after "CN=" and specify an alias
```
export CLIPASS=clientpass
cd /Users/joeqiao/Documents/LocalHub/kafka/kafka-security/Certs

keytool -genkeypair -keystore kafka.clientcert.keystore.jks -validity 365 -storepass joeclientpass2 -keypass joeclientpass2  -dname "CN=joemacbook" -alias joe-macbook -storetype pkcs12 -keyalg RSA -keysize 4096 -alias joe-macbook-key

keytool -list -v -keystore kafka.clientcert.keystore.jks -storepass joeclientpass2
```

## create a certification request file, to be signed by the CA
```
keytool -keystore kafka.clientcert.keystore.jks -certreq -file client-cert-sign-request -alias joe-macbook-key -storepass joeclientpass2 -keypass joeclientpass2
```
# switch to client instance, hence to the CA
## sign the server certificate => output: file "cert-signed"
```
openssl x509 -req -CA ca-cert -CAkey ca-key -in client-cert-sign-request -out client-cert-signed -days 365 -CAcreateserial -passin pass:joeserversecret
```

# switch back to local computer
## copy the signed certificate from EC2 instance to local computer and import to keystore
```
keytool -keystore kafka.clientcert.keystore.jks -alias CARoot -import -file ca-cert -storepass joeclientpass2 -keypass joeclientpass2 -noprompt
keytool -keystore kafka.clientcert.keystore.jks -import -file client-cert-signed -alias joe-macbook-cert -storepass joeclientpass2 -keypass joeclientpass2 -noprompt

keytool -list -v -keystore kafka.clientcert.keystore.jks -storepass joeclientpass2
```

## Add client cert to broker truststore
```
keytool -keystore kafka.server.truststore.jks -alias clientcert -import -file client-cert-signed -storepass serversecret -keypass serversecret -noprompt
keytool -list -v -keystore kafka.server.truststore.jks -storepass serversecret
```

# configure Kafka Broker
  * use [server.properties](./server.properties), the new property is *ssl.client.auth*
  * restart Kafka  
```
/Users/joeqiao/Documents/LocalHub/kafka/kafka_2.13-2.8.1/bin/kafka-server-start.sh /Users/joeqiao/Documents/LocalHub/kafka/kafka-security/Setup-SSL-Authentication/server.properties
```

# TEST
use [client-ssl-auth.properties](./client-ssl-auth.properties) and execute console producer/-consumer
```
/Users/joeqiao/Documents/LocalHub/kafka/kafka_2.13-2.8.1/bin/kafka-console-producer.sh --broker-list localhost:9093 --topic kafka-security-topic --producer.config /Users/joeqiao/Documents/LocalHub/kafka/kafka-security/Setup-SSL-Authentication/client-ssl-auth.properties
/Users/joeqiao/Documents/LocalHub/kafka/kafka_2.13-2.8.1/bin/kafka-console-consumer.sh --bootstrap-server localhost:9093 --topic kafka-security-topic --consumer.config /Users/joeqiao/Documents/LocalHub/kafka/kafka-security/Setup-SSL-Authentication/client-ssl-auth.properties
```
