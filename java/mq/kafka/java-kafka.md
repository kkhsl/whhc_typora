# Kafka的Java客户端-生产者

## 1.引入依赖

```xml
<dependency>
	<groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.4.1</version><!--版本和kakfa版本保持一致-->
</dependency>
```



## 2.生产者发送消息的基本实现

```java
public class MyProducer{
    
    private final static String TOPIC_NAME = "my-replicated-topic";
    
    public static void main(String[] args)throws ExecutionException, InterruptedException {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"0.0.0.0:00,0.0.0.0:00,0.0.0.0:00");
        
        // 把发送的key从字符串序列中转化为字节数组
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        // 把发送的消息value从字符串序列转化为字节数组
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        // 发送消息的客户端
        Producer<String, String> producer = new KafkaProducer<String, String>(props);
        Order order = new Order((long)i, i);
        // 发送消息到指定分区
        ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>(TOPIC_NAME, 				order.getOrderId().toString(), JSON.toJSONString(order));
        // 等待消息发送成功的同步阻塞方法
        RecordMetadata metadata = producer.send(producerRecord).get();
        // ====== 阻塞 ======
        System.out.println("同步方式发送消息结果：" + "topic-" + metadata.topic() + "|patition-" 
                          + metadata.partition() + "|offset-" + metadata.offset());
    }
}
```



1. 发送消息到指定分区上
   
   1. 未指定分区，则会通过业务key的hash运算，算出消息往哪个分区上发
   
2. 同步发送

   1. 生产者同步发送消息
      1. 如果生产者发送消息没有收到ack，生产者会阻塞，阻塞到3s的时间，如果还没有收到消息，会进行重试，重试的次数3次

3. 异步发消息

   1. 异步发送消息，生产者发送完消息之后就可以执行之后的业务，broker在收到消息后异步调用生产者提供的callback回调方法	

4. 关于生产者的ack参数配置

   1. 在同步发消息的场景下：生产者发动broker上后，ack会有3种不同的选择
      1. acks=0；表示producer不需要等待任何broker确认收到消息的回复，就可以继续发送下一条消息。性能高，但是最容易丢消息
      2. acks=1；至少等待leader已经成功将数据写入本地log,但是不需要等待所有follerwer是否成功写入。就可以继续发送下一条消息，这种情况下，如果follower没有成功备份数据，而此时leader又挂掉了，则消息会丢失
      3. acks=-1 或 all；需要等待，min.insync.replicas=2（默认为1，推荐配置大于等于2）这个参数配置的副本个数都成功写入日志，这种策略会保证只要有一个备份存活就不会丢失数据，这是最强的数据保证。一般除非是金融级别，或跟钱打交道的场景才会使用这种配置（此时需要leader和一个follower同步完之后，才会返回ack给生产者最安全，但性能最差（此时集群中已有两个broker完成数据的接收））

5. 消息缓冲区

   1. kafka默认会创建一个消息缓冲区，用来存放要发送的消息，缓冲区是32M

      ```java
      props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
      ```

   2. Kafka本地线程会去缓冲区一次拉16k的数据，发动到broker

      ```java
      props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
      ```

      

   3. 如果线程拉不到16k，间隔10ms也会将已拉到的数据发到broker

      ```java
      props.put(ProducerConfig.LINGER_MS_CONFIG, 10);
      ```

      

# 消费者

1. 消费者消费消息的基本实现

   ```java
   public class KafkaConsumerDemo {
   
       private final static String TOPIC_NAME = "my-repllicated-topic";
       private final static String CONSUMER_GROUP_NAME = "testGroup";
   
       public static void main(String[] args) {
   
           Properties props = new Properties();
           props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "0.0.0.0:00,0.0.0.0:00,0.0.0.0:00");
           // 消费分组名
           props.put(ConsumerConfig.GROUP_ID_CONFIG, CONSUMER_GROUP_NAME);
           // 是否自动提交offset，默认就是true
           //props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
           // 自动提交offset的间隔时间
           //props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
           props.put(ConsumerConifg.ENABLE_AUTO_COMMIT_CONFIG, "false");
           /**当消费主题的是一个新的消费组，或者指定offset的消费方式，offset不存在，那么应该如何消费
           latest(默认)：只消费自己启动之后发送到主题的消息
           earliest：第一次从头开始消费，以后按照消费offset记录继续消费，这个需要区别于consumer.seekToBeginning(每次从头		  			消费)*/
           // props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG,1000);
           // consumer给broker发送心跳的间隔时间
           props.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 1000);
           // kafka如果超过10秒没有收到消费者的心跳，则会把消费者踢出消费组，进行rebalance，把分区分配给其他消费者
           props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 10 * 1000);
           
           props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
           props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
           // 创建一个消费者的客户端
           KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
           // 消费者订阅主题列表
           consumer.subscribe(Arrays.asList(TOPIC_NAME));
           while (true){
               /**
                * poll() API 是拉取消息的长轮询
                */
               ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
               for (ConsumerRecord<String, String> record : records){
                   System.out.printf("收到消息：partition = %d,offset = %d, key = %s, value = %s%n", record.partition(),record.offset(),record.key(), record.value());
               }
               if(records.count > 0){
                   // 手动同步提交offset，当前线程会阻塞直到offset提交成功
                   // 一般使用同步提交，因为提交之后一般也没有什么逻辑代码了
                   consumer.commitSync();// ====== 阻塞 ===== 提交成功
               }
           }
   
       }
   
   }
   ```

2. 自动提交offset

   1. 设置自动提交参数-默认

      ```java
      // 是否自动提交offset，默认就是true
      
      props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
      
      // 自动提交offset的间隔时间
      
      props.pu(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
      ```

      消费者poll到消息后默认情况下，会自动向broker的_consumer_offsets主题提交当权主题-分区消费的偏移量

      **自动提交会丢消息**：因为消费者还没消费完poll下来的消息就自动提交偏移量，那么此时消费者挂了，于是下一个消费者会从已提交的offset的下一个位置开始消费消息。之前未被消费的消息就会丢掉了。

3. 手动提交offset

4. 消费者poll消息的过程

   1. 消费者建立了与broker之间的长连接，开始poll消息。
   2. 默认一次poll500条消息
   3. 可以根据消费速度的快慢来设置，因为如果两次poll的时间如果超过了30s的时间间隔，kafka会认为其消费能力过弱，将其踢出消费组。将分区分配给其他消费者

   可以通过这个值来设置：

   ```java
   props.put(ConsumerConfig.MAX_POLL_INTERVAL_CONFIG, 30*1000);
   ```

   - 如果每隔1s内没有收到poll的任何消息，则继续poll消息，循环往复，知道poll到消息。如果超过1s，则此长轮询结束

   ```java
   ComsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
   ```

   - 消费者发送心跳的时间间隔

   ```java
   props.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 1000);
   ```

   - kafka如果超过10秒没有收到消费者的心跳，则会把消费者踢出消费组，进行rebalance，把分区分配给其他消费者

   ```java
   props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 10 * 1000);
   ```

   

5. 指定分区消费

   ```java
   consumer.assign(Arrays.asList(new TopicPartition(Topic_NAME, 0)));
   ```

   

6. 消息回溯消费

   ```java
   consumer.assign(Arrays.asList(new Topictition(Topic_NAME, 0)));
   consumer.seekToBeginning(Arrays.asList(new TopicPatition(TOPIC_NAME, 0)));
   ```

   

7. 指定offset消费

   ```java
   consumer.assgin(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
   consumer.seek(new TopicPartition(TOPIC_NAME, 0), 10);
   ```

   

8. 从指定时间点消费

   根据时间，去所有的partition中确定盖时间对应的offset，然后去所有的partition中找到该offset之后的消息开始消费

   ```java
   List<PartitionInfo> topicPartitions = consumer.partitionsFor(TOPIC_NAME);
   // 从1小时前开始消费
   long fatchDataTime = new Data.getTime() - 1000 * 60 * 60;
   Map<TopicPartition, Long> map = new HashMap<>();
   for(PartitionInfo par : topicPartitions){
       map.put(new TopicPattition(TOPIC_NAME, par.partition()), fatchDataTime);
   }
   Map<TopicPartition, OffsetAndTimestamp> parMap = consumer.offsetsForTimes(map);
   for(Map.Entry<TopicPartition, OffsetAndTimestamp> entry : parMap.entrySet()){
       TopicPartition key = entry.getKey();
       OffsetAndTimestamp value = entry.getValue();
       if(key == null || value = null) continue;
       Long offset = value.offset();
       System.out.println("partition-" + key.partition() + "|offset-" + offset);
       // 根据消费者的timesamp确定offset
       if(value != null){
           consumer.assgion(Arrays.asList(key));
           consumer.seek(key, offset);
       }
   }
   ```

   

9. 新消费组的消费偏移量

   新消费组中的消息在启动之后，默认会从当前分区的最后一条消息的offset + 1开始消费（消费新消息）。可以通过一下的设置，让新的消费者第一次从头开始消费。之后开始消费新消息（最后消费的位置的偏移量 + 1）

   Laster：默认的，消费新消息

   ealiest：从头开始消费。之后开始消费新消息（最后消费的位置的偏移量 + 1）

   ```java
   props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG,"earliest");
   ```

   

   

# SpringBoot中使用Kafka

1. 引入依赖

   ```java
   <dependency>
   	<groupId>org.springframwork.kafka</groupId>
       <artifactId>spring-kafka</artifactId>
   </dependency>
   ```

2. 配置文件

   ```yml
   server:
   	port: 8080
   spring:
   	kafka:
   		bootstrap-servers: 0.0.0.0:00,0.0.0.0:00,0.0.0.0:00
       	producer: #生产者
       		retries: 3	#设置大于0的值，则客户端会将发送失败的记录重新发送
       		batch-size: 16384
       		acks: 1
       		# 指定消息key和消息体的编码方式
       		key-serializer: org.apache.kafka.common.serialization.StringSerializer
       		value-serializer: org.apache.kafka.common.serialization.StringSerializer
       	consumer:
       		group-id: default-group
       		enable-auto-commit: false #手动提交
       		auto-offset-reset: earliest
       		key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
       		value-deserializer: org.apache.kafka.common.serialization.StringDeseralizer
       		max-poll-records: 500
       	listener:
       		# 当每一条记录被消费者监听（ListenerConsumer）处理之后提交
       		# RECORD
       		# 每当一批poll()的数据被消费者监听器（ListenerConsumer）处理提交之后
       		# BATCH
       		# 每当一批poll()的数据被消费者监听器（ListenerConsumer）处理提交之后，距离上次提交时间大于TIME时提交
       		# TIME
       		#　每当一批poll()的数据被消费者监听器（ListenerConsumer）处理提交之后，被处理record数量大于等于COUNT时提交
       		# COUNT
       		# TIME | COUNT 有一个条件满足时提交
       		# COUNT_TIME
       		# 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后，手动调用Acknowledgment.acknowledge()提交
       		# MANUAL
       		# 手动调用Acknowledgment.acknowledge()后立即提交，一般使用这种
       		# MANUAL_IMMEDIATE
       	redis:
       		host: 0.0.0.0
   ```

   

3. 生产者

   ```java
   @RestController
   @RequestMapping("/admin/kafka")
   public class MyKafkaController {
       private final static String TOPIC_NAME = "my-replicated-topic";
   
       @Autowired
       private KafkaTemplate kafkaTemplate;
   
       @RequestMapping("/send")
       public String sendMsg(){
           kafkaTemplate.send(TOPIC_NAME, 0,"key","this is a message");
           return "send success";
       }
   
   
   }
   ```

4. 消费者

   1. 设置消费组，消费指定topic

   ```java
   @Component
   public class MyConsumer {
   
       /**
        * 设置为手动提交时 ack才生效
        * @param record
        * @param ack
        */
       @KafkaListener(topics = "my-replicated-topic", groupId = "MyGroup1")
       public void listenerGroup(ConsumerRecord<String, String> record, Acknowledgment ack){
           String value = record.value();
           System.out.println(value);
           System.out.println(record);
           // 手动提交offset
           ack.acknowledge();// 设置了手动提交，但不执行该方法执行提交，以上方法会被重复消费
       }
   
   }
   ```

5. 消费者中配置消费主题、分区和偏移量

   1. 设置消费组、多topic、指定分区、指定偏移量及设置消费者个数

   ```java
    @KafkaListener(groupId = "testGroup", topicPartitions = {
               @TopicPartition(topic = "topic1", partitions = {"0","1"}),
               @TopicPartition(topic = "topic2", partitions = "0",
               partitionOffsets = @PartitionOffset(partition = "1", initialOffset = "100"))
       })
       public void  listenGroupPro(ConsumerRecord<String, String> record, Acknowledgment ack){
           String value = record.value();
           System.out.println(value);
           System.out.println(record);
           // 手动提交offset
           ack.acknowledge();// 设置了手动提交，但不执行该方法执行提交，以上方法会被重复消费
       }
   ```

   

