# redis

端口号 6379

## 索引

sorted set
底层结构是跳表、有点类似哈希
它里面的每个元素都和一个浮点数映射（对应）

跳表时间复杂度：O(log2n)

## 命令

- 计数命令

- 排序命令

- 加锁命令


## 架构

- 常用数据类型

- 淘汰策略

1. lru，在设定时间的范围内LRU
2. 全部LRU
3. 随机淘汰
4. 设置超时时间的范围内随机淘汰。
5. 

- 单线程为什么快？

- RDB和AOF优缺点
  持久化方式：
  RDB数据快照、AOF是保存增量记录。

- 持久化策略选择

- redis如何开启持久化：
  vim redis.conf

## 应用

- 缓存雪崩、缓存穿透、缓存预热、缓存更新、缓存降级

- Pipeline的好处

- Redis集群 cluster原理

- Redis同步机制是什么


### Redis分布式锁的实现方式和缺点

1. **死锁（加锁以后客户端突然挂掉，锁没法释放**

- 设置锁的有效期 setnx lockkey true, expire lockkey 5 , del lockkey
  1.1 **但是setnx和expire是两次RTT操作，如果在中间突然挂掉，又会死锁**
  - 使用 set lockkey true ex 5nx 扩展命令保证原子性

2. **set和get线程不安全，可能会产生重复id**

- 使用setnx：set if not exist，redis实例存在唯一key，如果还想设置则会被拒绝。

3. **锁过期，业务没有执行完，续期**

- 使用Redisson的Api： RLock lock = redissonClient.getLock("lock");
- redisson加锁成功以后，会注册一个定时任务对锁监听，每10秒检查是否还持有锁，如果是，则续期。“看门狗机制”

4. **B的锁被A释放**

- A线程使用setnx获得了锁，此时B线程等待，随后A线程业务运行缓慢，锁被自动释放，B线程拿到了锁。
- A线程业务完成后去释放锁，把B的锁释放了
- 解决方法：在释放锁的时候判断value，每次加锁的时候设置独一无二的value，比如线程id或uuid

5.  **锁的可重入**

- 重入锁：拥有锁的客户端再次获得锁
- 给锁设置hash结构的加锁次数，每次加锁+1

6. **集群分布式锁问题**

- 主节点和从节点，**主节点**负责**写**操作，**从节点**负责**读**操作
- 会导致的问题：所有的锁都写在主节点服务器上，如果主节点宕机，且数据没有复制到从服务器上，在重新选举主节点后，其他客户端会获取到锁。
- 拥有锁的客户端会继续操作，就会产生问题。
  解决方法：
- 使用redlock，多台主机中超过半数设置成功则设置加锁成功，主机个数必须是奇数。

### redis过期机制

使用过期字典来存储过期时间（类似哈希表）

### 过期数据删除策略：

1. 惰性删除：
   1. 在取出key的时候检查是否过期，会导致大量的未删除key
2. 定期删除
   1. 每隔一段时间抽取一批key操作，并通过限制删除操作执行时长等方法来减少删除延迟

- 但是这种方法会导致产生大量无用key没法被删除，因此需要淘汰策略
- Redis提供了6种内存淘汰机制

1. volatile-lru：从已过期的key中找到最近最少使用的key进行删除操作
2. volatile-ttl：从设置过期时间的数据集中寻找
3. random：从设置过期时间的数据集中随机淘汰
4. allkeys-lru：当内存不足的时候从键空间中移除最近最少使用的key
5. allkeys-random：随机
6. volatile-lfu：最少使用的数据淘汰
7. allkeys-lfu：内存不足的时候，移除最少使用key


### 持久化：

AOF重写机制：是指将AOF生成另一个效果一样的，但是占用空间更小的AOF文件


### 缓存穿透：

是指访问缓存中不存在的数据

### 缓存雪崩：

缓存在同一时刻大量失效，导致大量的请求打到数据库上

### 缓存击穿：

热点缓存失效