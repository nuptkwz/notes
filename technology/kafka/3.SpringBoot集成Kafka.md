# 0. 前言
> Kafka是由Apache软件基金会开发的一个开源流处理平台，由Scala和Java编写。 Kafka 是一种高吞吐量的
  分布式发布订阅消息系统，它可以处理消费者在网站中的所有动作流数据。本章介绍SpringBoot集成Kafka
  收发消息。本文介绍了SpringBoot集成Kafka。

# 1. 环境
- spring-boot: 2.6.6
- spring-kafka: 2.8.4

# 2. 配置文件
## 2.1 yml配置文件
```yaml
spring:
  application:
    name: springboot-kafka
  kafka:
    bootstrap-servers: 192.168.30.130:9092
    producer:
      # 序列化key的类
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      # 序列化value的类
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      properties:
        partitioner:
          # 指定分区规则
          class: com.engrz.lab.springboot.kafka.CustomPartitioner
        spring:
          json:
            value:
              default:
                type:
            trusted:
              # 允许json反序列化的包
              packages: com.engrz.lab.*
            add:
              type:
                headers: false
    consumer:
      #bootstrap-servers: 192.168.30.130:9092 # 会覆盖 spring.kafka.bootstrap-servers 配置
      # 消费者所属消息组
      group-id: group_1
      # 反序列化key的类
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      # 反序列化value的类
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
    template:
      default-topic: kwz_1
```
## 2.2 KafkaProducerConfig
```java
@Configuration
public class KafkaProducerConfig {

    @Value("${spring.kafka.bootstrap-servers}")
    private String producerBootstrapServers;

    @Bean
    public ProducerFactory<String, String> producerFactory() {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, producerBootstrapServers);
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return new DefaultKafkaProducerFactory<>(configs);
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

## 2.3 KafkaConsumerConfig
```java
@EnableKafka
@Configuration
public class KafkaConsumerConfig {

    @Value("${spring.kafka.bootstrap-servers}")
    private String consumerBootstrapServers;

    @Value("${spring.kafka.consumer.group-id}")
    private String consumerGroupId;

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, consumerBootstrapServers);
        props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 50);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConcurrency(3);
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

# 3. 生产消费者配置
## 3.1 producer
```java
@Component
public class Producer {

    @Value("${spring.kafka.template.default-topic}")
    private String defaultTopic;

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    public void sendMessage(String message) {
        kafkaTemplate.send(defaultTopic, message);
    }

    public void sendMessage(String key, String message) {
        kafkaTemplate.send(defaultTopic, key, message);
    }
}
```
## 3.2 consumer
```java
@Component
@Slf4j
public class Consumer {

    @KafkaListener(topics = "kwz_1")
    public void listener(ConsumerRecord<String, String> record) {
        String value = record.value();
        log.info("【receive】:{}", value);
    }
}
```

## 4 参考
1. [Spring for Apache Kafka](https://docs.spring.io/spring-kafka/docs/current/reference/html/)
2. [Spring Boot 集成 Kafka](https://engr-z.com/150.html)
3. [SpringBoot集成Kafka](https://segmentfault.com/a/1190000038838207)
4. [Spring Boot集成Kafka](https://juejin.cn/post/6844903969265975309)

