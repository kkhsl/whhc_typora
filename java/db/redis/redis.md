# 1.应用场景

1. 缓存：Redis可以作为应用程序的缓存，减少数据库的读取压力，提高数据访问速度

2. 会话存储：在Web应用中，Redis可以用来存储用户的会话信息，如登录状态、购物车内容等

3. 排行榜和计数器：Redis支持原子操作，非常适合实现实时排行榜、点赞数、访问计数等共功能

4. 消息队列：Redis可以作为消息队列系统，用于处理异步任务，例如邮件发送、后台任务处理等

5. 实时分析：Redis可以用于实时分析，如用户行为分析、实时统计信息等

6. 分布式锁：在分布式系统中，Redis可以用于实现分布式锁，确保在多个节点之间共享资源的一致性

   1. **Redis**：使用Redis的 `SETNX` 和 `EXPIRE` 命令来实现分布式锁。通过设置一个特定的键值对来表示锁的占用状态，并设置锁的过期时间来避免死锁。

      ```java
      // 使用Redisson库实现分布式锁示例
      RedissonClient redisson = Redisson.create();
      RLock lock = redisson.getLock("myLock");
      try {
          lock.lock();
          // 执行需要保护的代码块
      } finally {
          lock.unlock();
      }
      ```

      

   2. **ZooKeeper**：

      使用ZooKeeper的临时顺序节点和 Watcher 机制来实现分布式锁。每个客户端尝试创建一个唯一的临时顺序节点，根据节点顺序来判断锁的竞争情况，并通过 Watcher 监听节点的变化来实现锁的释放（chatgpt）

      ```java
      // 使用Curator库实现分布式锁示例
      CuratorFramework client = CuratorFrameworkFactory.newClient("localhost:2181", new RetryForever(1000));
      client.start();
      
      InterProcessMutex lock = new InterProcessMutex(client, "/myLockPath");
      try {
          if (lock.acquire(10, TimeUnit.SECONDS)) {
              // 执行需要保护的代码块
          }
      } finally {
          lock.release();
      }
      
      ```

      1. 黑马
         1. 核心思想：当客户端要获取锁，则创建节点，使用完锁，则删除该节点。（黑马）
            - 客户端获取锁时，在lock节点下创建临时顺序节点
            - 然后获取lock下面的所有子节点，客户端获取到所有子节点之后，如果发现自己创建的子节点序号最小，那么就认为该客户端获取到了锁。使用完锁后，将该节点删除
            - 如果发现自己创建的节点并非lock所有子节点中最小的，说明自己还没有获取到锁，此时客户端需要找到比自己小的那个节点，同时对其注册事件监听器，监听删除事件
         2. Curator实现分布式锁API
            1. InterProcessSemaphoreMytex：分布式排它锁（非可重入）
            2. InterProcessMutex：分布式可重入排它锁
            3. InterProcessReadWriteLock：分布式读写锁
            4. InterProcessMultiLock：将多个锁作为单个实体管理的容器
            5. InterProcessSemaphoreV2：共享信号量

   3. **分布式数据库**：对于支持分布式事务和行锁的数据库，可以使用数据库的行锁机制来实现分布式锁。例如，TiDB、CockroachDB等分布式数据库提供了分布式事务和行锁功能，可以用来实现分布式锁（**可能存在性能瓶颈**）

      1. 基于数据库的锁，如何避免死锁

   4. **分布式锁框架**：有些第三方库或框架提供了封装好的分布式锁实现，如Curator（基于 ZooKeeper）、Redisson（基于 Redis）、Spring Integration等。这些框架可以简化分布式锁的使用和管理。

   5. **使用分布式数据库的行锁**：对于支持分布式事务的分布式数据库（如TiDB、CockroachDB等），可以使用数据库提供的行锁或者分布式事务特性来实现分布式锁

   6. 基于缓存

      1. redis、memcache
      2. zookeeper：curator
      3. 数据库：悲观锁、乐观锁

7. 发布/订阅：Redis提供了发布/订阅模式，可以用于实现消息广播，例如实时通知系统

8. 限流：Redis可以用于实现限流功能，防止系统过载，如API调用频率限制

9. 数据过期：Redis支持设置数据过期时间，自动清理过期数据，适用于临时数据存储

10. 全页缓存：Redis可以缓存整个页面的输出，减少数据库查询和页面渲染的时间

11. 社交功能：在社交网络应用中，Redis可以用于存储好友关系、用户状态更新等

12. 实时推荐系统：Redis可以用于存储用户的行为数据和偏好，实现实时推荐

13. 地理位置信息：Redis支持Geospatial索引，可以用户实现地理位置相关的查询

14. 时间序列数据：Redis可以存储时间序列数据，用于监控和分析

15. 任务调度：Redis可以用于任务调度，例如定时任务执行

16. 数据共享：在微服务架构中，Redis可以作为服务间共享数据的媒介

17. 持久化：虽然Redis是内存数据库，但它也支持数据库持久化，可以在系统故障后恢复数据

> 注：
>
> 1. 函数式参数
> 2. 主从、哨兵、集群