# Developing Kafka Applications - Producer API

In this part of the lab, you will create a Kafka Producer using the Java API. The next part will be the creation of the Kafka Consumer so that you can see an end to end example using the API.

## Objectives

1. Create topics on the Kafka server for the producer to send messages
2. Understand what the producer Java code is doing
3. Compile and run the example producer

## Prerequisites

Like the previous lab, [Docker](https://www.docker.com) will be used to start a Kafka and Zookeeper server. We will also 
have the option to use a [Maven](https://maven.apache.org/) Docker image to compile the Java code, or simply go through the IDE.
The Kafka Producer example will be explained and then you will compile and execute it against the Kafka server.

## Instructions

All the directory references in this lab are relative to where you extracted the lab files `lab02-kafka-producer-consumer`

1. Open a terminal in the directory: `docker/`.

   1. Open the `docker-compose.yml` file which contains the following:

       ```yaml
       version: '3'
       services:
         zookeeper:
           image: bitnami/zookeeper:latest
           environment:
             ALLOW_ANONYMOUS_LOGIN: yes
           ports:
             - 2181:2181
         kafka:
           image: bitnami/kafka:latest
           ports:
             - 9092:9092
             - 7203:7203
           environment:
         #    KAFKA_ADVERTISED_HOST_NAME: [YOUR_IP_ADDRESS]
              KAFKA_ADVERTISED_HOST_NAME: localhost
              KAFKA_ADVERTISED_PORT: 9092
              ALLOW_PLAINTEXT_LISTENER: yes
              KAFKA_CFG_LISTENERS: PLAINTEXT://:9092
              KAFKA_CFG_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
              KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
           depends_on:
             - zookeeper
       ```

   To allow connectivity between the local Java application and Kafka running in Docker, you need to insert the locally assigned IP address for your computer for the value of `KAFKA_ADVERTISED_HOST_NAME`.

   On OSX, you can use the following command:

       $ ifconfig | grep inet
           inet 127.0.0.1 netmask 0xff000000
           inet6 ::1 prefixlen 128
           inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1
           inet6 fe80::1cf1:f9f4:6104:f2a3%en0 prefixlen 64 secured scopeid 0x4
           inet 10.0.1.4 netmask 0xffffff00 broadcast 10.0.1.255
           inet6 2605:6000:1025:407f:101b:9cd8:e973:b9dd prefixlen 64 autoconf secured
           inet6 2605:6000:1025:407f:9077:8802:8c88:e63d prefixlen 64 autoconf temporary
           inet6 fe80::cc8a:c5ff:fe43:b670%awdl0 prefixlen 64 scopeid 0x8
           inet6 fe80::7df6:ec93:ffea:367a%utun0 prefixlen 64 scopeid 0xa

       In this case, the IP address to use is `10.0.1.4`. Make sure you *do not* use `127.0.0.1` because that will not work correctly.
   
   On Windows, you can use the following command:
   
   ```
    $ ipconfig
    Ethernet adapter Local Area Connection:
    Connection-specific DNS Suffix . : hsd1.ut.comcast.net.
    IP Address. . . . . . . . . . . . : 192.168.201.245
    Subnet Mask . . . . . . . . . . . : 255.255.255.0
    Default Gateway . . . . . . . . . : 192.168.201.1
    ```

    Here the IP address to use is `192.168.201.245`.

    After you replace the line in `docker-compose.yml`, it should look like this:

    ```
    KAFKA_ADVERTISED_HOST_NAME: 192.168.201.245
    ```

    Save the `docker-compose.yml` file after making this modification.

      > We have noticed on some configurations of Windows and Linux that the use of `KAFKA_ADVERTISED_HOST_NAME` does not work properly (the Kafka clients can't connect).
      > We've not found the source of this problem, but in many of the cases we've seen, the use of `localhost` instead of the host IP may work.
      > Note though, that the use of `localhost` prevents you from running multiple Kafka brokers on the same machine.

2. Start the Kafka and Zookeeper processes using Docker Compose:

    ```
    $ docker compose up
    ```

3. Open an additional terminal window in the lab directory, `docker/`. We are going to create two topics that will be used in the Producer program. Run the following commands:

    ```
    $ docker compose exec kafka /opt/bitnami/kafka/bin/kafka-topics.sh --create --bootstrap-server kafka:9092 --replication-factor 1 --partitions 1 --topic user-events
    $ docker compose exec kafka /opt/bitnami/kafka/bin/kafka-topics.sh --create --bootstrap-server kafka:9092 --replication-factor 1 --partitions 1 --topic global-events
    ```

4. List the topics to double check they were created without any issues.

    ```
    $ docker compose exec kafka /opt/bitnami/kafka/bin/kafka-topics.sh --list --bootstrap-server kafka:9092
    global-events
    user-events
    ```

5. Open `producer/pom.xml` in your IntelliJ. The current stable version of Kafka is 3.1.0. Maven is being used for dependency management in this lab. It includes the following in the `pom.xml` for Kafka:

    ```xml
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>3.1.0</version>
    </dependency>
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>31.0.1-jre</version>
    </dependency>
    ```

     The only other dependency being used is `guava` from Google which contains some helpful utility classes that we are going to use to load a resource file.

6. Next open `producer/src/main/java/com/example/Producer.java` in your IDE. This class is fairly simple Java application, but it contains all the functionality necessary to operate as a Kafka Producer. The application has two main responsibilities:

    * Initialize and configure a [KafkaProducer](https://javadoc.io/doc/org.apache.kafka/kafka-clients/latest/org/apache/kafka/clients/producer/KafkaProducer.html)
    * Send messages to topics with the Producer object

    To create our Producer object, we must create an instance of `org.apache.kafka.clients.producer.KafkaProducer` which requires a set of properties for initialization. While it is possible to add the properties directly in Java code, a more likely scenario is that the configuration would be externalized in a properties file. The following code instantiates a `KafkaProducer` object using `resources/producer.properties`.

    ```java
    KafkaProducer<String, String> producer;
    try (InputStream props = Resources.getResource("producer.properties").openStream()) {
        Properties properties = new Properties();
        properties.load(props);
        producer = new KafkaProducer<>(properties);
    }

    ```

    Open `resources/producer.properties` and you can see that the configuration is minimal:

    ```
    acks=all
    retries=0
    bootstrap.servers=localhost:9092
    key.serializer=org.apache.kafka.common.serialization.StringSerializer
    value.serializer=org.apache.kafka.common.serialization.StringSerializer
    ```

    * `acks` is the number of acknowledgments the producer requires the leader to have received before considering a request complete. This controls the durability of records that are sent. A setting of `all` is the strongest guarantee available.
    * Setting `retries` to a value greater than 0 will cause the client to resend any record whose send fails with a potentially transient error.
    * `bootstrap.servers` is our required list of host/port pairs to connect to Kafka. In this case, we only have one server. The Docker Compose file exposes the Kafka port so that it can be accessed through `localhost`.
    * `key.serializer` is the serializer class for key that implements the `Serializer` interface.
    * `value.serializer` is the serializer class for value that implements the `Serializer` interface.

    There are [many configuration options available for Kafka producers](http://kafka.apache.org/documentation.html#producerconfigs) that should be explored for a production environment.

7. Once you have a Producer instance, you can post messages to a topic using the [ProducerRecord](https://kafka.apache.org/31/javadoc/org/apache/kafka/clients/producer/ProducerRecord.html). A `ProducerRecord` is a key/value pair that consists of a topic name to which the record is being sent, an optional partition number, an optional key, and required value.

    In our lab, we are going to send 100000 messages in a loop. Each iteration of the loop will consist of sending a message to the `user-events` topic with a key/value pair. Every 100 iterations, we also send a randomized message to the `global-events` topic. The message sent to `global-events` does not have a key specified which normally means that Kafka will assign a partition in a round-robin fashion but our topic only contains 1 partition.

    After the loop is complete, it is important to call `producer.close()` to end the producer lifecycle. This method blocks the process until all the messages are sent to the server. This is called in the `finally` block to guarantee that it is called. It is also possible to use a Kafka producer in a [try-with-resources statement](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html).

    ```java
    try {
        for (int i = 0; i < 100000; i++) {
            producer.send(new ProducerRecord<String, String>(
                  "user-events", // topic
                  "user_id_" + i, // key
                  "some_value_" + System.nanoTime())); // value

            if (i % 100 == 0) {
                String event = global_events[(int) (Math.random() * global_events.length)] + "_" + System.nanoTime();

                producer.send(new ProducerRecord<String, String>(
                    "global-events", // topic
                    event)); // value

                producer.flush();
                System.out.println("Sent message number " + i);
            }
        }
    } catch (Throwable throwable) {
        System.out.println(throwable.getStackTrace());
    } finally {
        producer.close();
    }
    ```

8. Now we are ready to compile and run the lab in the IDE by simply starting the Producer.  Alternatively, in a terminal, change to the `lab` directory and run the following  [Maven](https://maven.apache.org/) command:

    ```
    $ mvn clean package
    ```

9. For convenience, the project is set up so that the `package` target produces a single executable: `target/producer`. Run the producer to send messages to our two topics -- `user-events` and `global-events`.

    ```
    $ target/producer
    INFO org.apache.kafka.clients.producer.ProducerConfig - ProducerConfig values:
   ...
    Sent message number 0
    Sent message number 100
    ...
    Sent message number 99800
    Sent message number 99900
    ```

10. *Don't* stop the Kafka and Zookeeper servers because they will be used in the next part of the lab focusing on the Consumer API.

### Conclusion
We have now successfully sent a series of messages to Kafka using the Producer API. In the next part, we will write a consumer program to process the messages from Kafka.

Congratulations, this part of the lab is complete!
