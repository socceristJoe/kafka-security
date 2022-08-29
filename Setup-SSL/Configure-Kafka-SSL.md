
## create a server certificate !! put your public EC2-DNS here, after "CN="
```
cd /Users/joeqiao/Documents/LocalHub/kafka/kafka-security/Certs

keytool -genkeypair -keystore kafka.server.keystore.jks -validity 365 -storepass serversecret -keypass serversecret  -dname "CN=localhost" -storetype pkcs12 -keyalg RSA -keysize 4096 -alias joekafkakeypair2
#> ll

keytool -list -v -keystore kafka.server.keystore.jks -storepass serversecret
```

## create a certification request file, to be signed by the CA
```
keytool -certreq -keystore kafka.server.keystore.jks -file cert-file -storepass serversecret -keypass serversecret -alias joekafkakeypair2
#> ll
```

## sign the server certificate => output: file "cert-signed"
```
openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial -passin pass:serversecret
#> ll
```

## check certificates
### our local certificates
```
keytool -printcert -v -file cert-signed
keytool -list -v -keystore kafka.server.keystore.jks -storepass serversecret
```



# Trust the CA by creating a truststore and importing the ca-cert
```
keytool -keystore kafka.server.truststore.jks -alias CARoot -import -file ca-cert -storepass serversecret -keypass serversecret -noprompt
```
# Import CA and the signed server certificate into the keystore
```
keytool -keystore kafka.server.keystore.jks -alias CARoot -import -file ca-cert -storepass serversecret -keypass serversecret -noprompt
keytool -keystore kafka.server.keystore.jks -import -file cert-signed -storepass serversecret -keypass serversecret -noprompt

keytool -list -v -keystore kafka.server.truststore.jks -storepass serversecret
```

# Adjust Broker configuration  
Replace the server.properties in AWS, by using [this one](./server.properties).   
*Ensure that you set your public-DNS* !!

# Start Kafka
```
/Users/joeqiao/Documents/LocalHub/kafka/kafka_2.13-2.8.1/bin/zookeeper-server-start.sh /Users/joeqiao/Documents/LocalHub/kafka/kafka_2.13-2.8.1/config/zookeeper.properties

/Users/joeqiao/Documents/LocalHub/kafka/kafka_2.13-2.8.1/bin/kafka-server-start.sh /Users/joeqiao/Documents/LocalHub/kafka/kafka-security/Setup-SSL/server.properties
```
# Verify Broker startup
```
sudo grep "EndPoint" /Users/joeqiao/Documents/LocalHub/kafka/kafka_2.13-2.8.1/logs/server.log
```
# Adjust SecurityGroup to open port 9093

# Verify SSL config
from your local computer
```
openssl s_client -connect localhost:9093
```
