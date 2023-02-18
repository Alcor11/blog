## kafka

默认端口：9092

- 是一个分布式、支持分区、多副本的、基于zk的的分布式消息系统。
- 能够实时处理大量数据。
- 低延迟实时系统、访问日志，消息服务

主要功能：消息队列和数据处理。

**kafka的优势：**
性能：给予Scala和Java开发，大量的批量处理和异步思想，最高可以每秒处理千万级别消息。
生态系统兼容性好：Spack


#### Broker

- 一个kafka节点就是一个brocker

#### Topic

- 根据topic归类消息，发到kafka中的每条消息都需要指定topic

#### producer

- 生产者

#### Consumer

- 消费者

#### ConsumerGroup

- 每个consumer属于一个ConsumertGroup。
- 一条消息可以被多个CG消费，但是一个CG中只有一个Consumer能消费消息。

#### Partition （分区

- 一个topic分为多个Partition


#### Leader

- 负责数据读写的Partition
- 一个Partition有多个副本，其中只有一个作为leader，负责数据读写（和rocket MQ类似

#### Follower

- 跟随leader，所有写请求都经过Leader路由，然后数据广播给Follower，与Leader保持数据同步。如果Leader失效，则会从Follower中选举出一个新的Leader。
- Follower和leader都挂掉，则Leader会把这个Follower从ISR中删除，重新创建Follower。

#### offset

- 偏移量，kafka的存储文件时offset.kafka。

### 持久化

kafka会把消息持久化到磁盘，防止消息丢失。


### 集群高可用


### 数据高可靠的机制

1. **副本同步保证高可靠：**

- AR（Assigned Replicas）
  表示一组leader副本加follower副本，是已经分配数据的分区副本。

- ISR（In Sync Replicas）
  与leader保持同步的follower的集合。保持同步延时：默认10s内，一旦leader挂掉，新的leader从ISR中选出

- OSR(out Sync Replicas)
  follower同步的进度跟不上leader，则放进OSR中

- 通过**replica.lag.time.max.ms**参数设置数据同步时间差

- **副本数据同步**
- HW（High watermark）高水位，用来体现副本间数据同步的相对位置，consumer最多只能消费到HW位置，可以通过HW位置来判断数据对副本是否可见。
- LEO（Log End Offset）下一条待写入消息的位置。
  ![](https://alcor-1306883605.cos.ap-shanghai.myqcloud.com/my/p4vTmq.png)
- **HW截断机制**
- 当leader宕机，然后恢复以后，该节点的LEO会退回到旧HW的位置，然后重新同步，解决了一致性的问题。
- ![](https://alcor-1306883605.cos.ap-shanghai.myqcloud.com/my/gAPRPy.png)


2. **应答机制的可靠性**
   kafka的消息确认是通过ack来应答的
   ack=0：生产者发送消息以后不管是否成功，都会默认为成功，不再发送。性能很高，但是可靠性很低
   ack=1：leader接收到消息并持久化后则返回ack。这是默认的配置，缺点是可能还没同步给follower，如果leader所在的broker宕机了，则数据也会丢失。
   ack=1-：所有分区的副本全部响应ack才表示写成功，吞吐量很低，效率比较差，并且一旦在同步过程中某个副本挂掉，producer会重发消息，可能会导致消息重复。

3. **消息语义**

- **at most once**表示消息可能被消费者最多消费1次
  - 消息从partition发送给消费者集群
  - 消费者把自己收到的消息告诉集群，集群offset后移
  - 消费者将数据入库持久化。
- 可能会出现消息丢失。

- **at least once**
  - 消费者先将消息入库持久化
  - 再告诉集群收到消息
- 可能会重复消费，若消费者A数据入库后，还没回复就宕机，消费者b会去重新拉取消息。

- **exactly once**
  - 消息正好被消费一次
  - 再at least once的基础上，若消费者A入库后宕机了，消费者B会先去库中查询最新的消息，然后根据offset去集群中查询。

### 日志清理机制

- 基于时间戳：过期时间默认6天

- 基于文件大小：日志文件分段，总和超过某个大小时，删除某段日志。

- 日志压缩

### 多副本机制

kafka为分区：Partition引入了多副本Replica机制。分区(Partition)中的多个副本之间会有leader，其他的事follower，



