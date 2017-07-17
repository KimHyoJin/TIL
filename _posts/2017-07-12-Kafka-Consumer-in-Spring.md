---
layout: post
title:  "Kafka Consumer in Spring"
date:   2017-07-17 22:00
categories: Kafka
---
# Spring에서 Kafka Consumer 구현하기

#### Kafka Consumer implementation using org.apache.kafka dependency

Spring에 맞춰지지 않은 apache에서 제공하는 API를 통해 Kafka Consumer를 구현해보았습니다.
github에 있는 데 간단히 말하자면
1. property 설정
2. KafkaConsumer에 property설정 및 topic 구독
3. poll을 통해 data get

#### Kafka Consumer implementation using org.springframework.kafka dependency

그 다음으로는 좀 더 spring스럽게 구현하기 위해 Kafka Consumer를 spring framework를 위한 라이브러리를 이용하여 구현해보았습니다.

##### Dependency 추가
```xml
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka</artifactId>
  <version>1.1.6.RELEASE</version>
</dependency>
```
다음의 dependency를 추가해줍니다. Kafka Client별 맞는 spring-kafka 버전은 [여기](https://projects.spring.io/spring-kafka/)에서 확인할 수 있습니다.

##### Configuration
Kafka-Consumer를 위한 configuration을 설정해주어야합니다.
```java
@Bean
public Map<String, Object> consumerConfigs() {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "10.101.17.192:9092");
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "trackingService-consumer-view-mumo");
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

    return props;
}
```
Map형태의 props를 만들어줍니다.

###### return type을 Map으로 선언해준 이유
Map으로 선언해준 이유는 뒤에서 DefaultKafkaConsumerFactory를 이용하여 instance를 생성해줄 건데, 이때 DefaultKafkaConsumerFactory가 configuration으로 받는 타입이 Map이기 때문입니다.

@Bean으로 설정하여 어플리케이션 실행 시 Bean이 만들어짐과 함께 동작할 수 있도록 만들어줍니다.

```java
@Bean
public ConsumerFactory<String, String> consumerFactory(){
    return new DefaultKafkaConsumerFactory<String, String>(consumerConfigs());
}
```
consumerConfigs()를 configuration으로 주어 ConsumerFactory를 return합니다.
###### ConsumerFactory
ConsumerFactory는 interface로 정의되어 있다. Interface로도 return이 가능하다!(EX. Map를 return한다던가 등..) [여기참고](https://stackoverflow.com/questions/5699427/what-does-it-mean-for-a-function-to-return-an-interface)
DefaultKafkaConsumerFactory말고 다른 Class that implement ConsumerFactory를 return하고 싶을 수도 있기 때문이다.


```java
@Bean
KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());
    factory.setConcurrency(3);
    factory.getContainerProperties().setPollTimeout(3000);
    return factory;
}
```
우선 코드의 내용?부분은 이해가기 쉬우니까 넘어가고, return type은 KafkaListenerContainerFactory이다. 이건 ContainerFactory인데,
Factory of MessageListenerContainer based on a KafkaListenerEndpoint definition. 이렇게 적혀있다. MessageListenerContainer를 생성해주는 것 같다.
MessageListenerContainer로 주어진 것이 ConcurrentMessageListenerContainer이다.
그리고 실제 return type은 KafkaListenerContainerFactory를 상속한 ConcurrentKafkaListenerContainerFactory이다.
ConcurrentKafkaListenerContainerFactory는 concurrency에 특화되어있다고 소스에 적혀있는데, 어떻게 특화되어있는지는 모르겠다.
```java
This should be the default for most users and a good transition paths
for those that are used to build such container definitions manually.
 ```
보통 이걸 쓰나보다

```java

private CountDownLatch latch = new CountDownLatch(1);


@KafkaListener(topics="line-timeline-view")
public void receive(String data) {
    logger.info(data);
    latch.countDown();
}
```
그 다음에 Listener를 설정해준다. @KafkaListener는 다음의 것들을 @과 함께 정의할 수 있다.
- topics
- containerFactory
- topicPattern
- topicPartitions
- group

흠 나는 containerFactory를 설정하지 않았는데, 여기에 앞서 설정한 containerFactory를 넣어줘야할까? 앞서서 @Bean으로 설정했으므로 해당 빈은 생성이 되었고, 그 bean사이의 관계?도 설정이 되었을 거같은데 설정해줘야하는 지는 모르겠다.

CountDownLatch는 thread많을 때 synchronization을 위한 것이다. [여기참고](http://suribada.com/wp/?p=13)

근데 이렇게 했는데
1. Bootstrap broker 10.101.17.192:9092 disconnected 가 뜬다 (= 제대로 작동하지 않음)
2. Zookeeper는 어디에 넣는 것인가.. 관련 property를 찾지 못하였다.
