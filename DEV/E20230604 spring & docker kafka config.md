## spring & docker kafka config

### Docker kafka 

docker-compose-kafka.yml
```
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

docker compose up
```
docker-compose -f docker-compose-kafka.yml up -d
```

sample usage
```
docker exec -it kafka bash

// list topics
kafka-topics.sh --list --bootstrap-server localhost:9092

// describe topic
kafka-topics.sh --describe --topic ${TOPIC_NAME} --bootstrap-server localhost:9092

// consume
kafka-console-consumer.sh --topic ${TOPIC_NAME} --from-beginning --bootstrap-server localhost:9092
```

### Add gradle dependency (build.gradle)
```
dependencies {
    implementation 'org.springframework.kafka:spring-kafka'
}
```

### Configure Spring.kafka producer
```
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
```

### Sample usage (producer)
``` java
@Component
public class KafkaSampleProducer {

    @Value("${mymarket.order.kafka.topic.name}")
    private String topic;

    private final KafkaTemplate<String, User> kafkaTemplate;

    public KafkaOrderProducer(KafkaTemplate<String, User> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void send(User user) {
        kafkaTemplate.send(topic, user);
    }
}
```
