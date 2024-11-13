# Zookeeper

## 1.zookeeper简介

1. Zookeeper是Apache Hadoop项目下的一个子项目，是一个**树形目录服务**

2. 用来管理Hadoop、Hive、Pig

3. 分布式、开源的分布式应用程序的协调服务

4. 主要功能

   1. 配置管理

   2. 分布式锁

      实现：使用 Apache Curator 实现分布式锁的步骤：

      1. **添加 Maven 依赖**： 首先，确保在你的项目中添加了Apache Curator的依赖。在 Maven 项目中，可以添加如下依赖：

         ```
         <dependency>
             <groupId>org.apache.curator</groupId>
             <artifactId>curator-framework</artifactId>
             <version>5.1.0</version> <!-- 替换为最新版本 -->
         </dependency>
         <dependency>
             <groupId>org.apache.curator</groupId>
             <artifactId>curator-recipes</artifactId>
             <version>5.1.0</version> <!-- 替换为最新版本 -->
         </dependency>
         ```

         这将包含Curator的核心库和Recipes模块，后者包括用于分布式锁的实用程序类。

      2. **实现分布式锁**： 下面是一个简单的示例代码，演示如何使用Curator实现分布式锁：

         ```
         import org.apache.curator.framework.CuratorFramework;
         import org.apache.curator.framework.CuratorFrameworkFactory;
         import org.apache.curator.framework.recipes.locks.InterProcessLock;
         import org.apache.curator.framework.recipes.locks.InterProcessMutex;
         import org.apache.curator.retry.ExponentialBackoffRetry;
         
         public class DistributedLockExample {
             private static final String ZOOKEEPER_CONNECTION_STRING = "localhost:2181";
             private static final String LOCK_PATH = "/my/distributed/lock";
         
             public static void main(String[] args) throws Exception {
                 // 创建 Curator 客户端
                 CuratorFramework client = CuratorFrameworkFactory.newClient(
                     ZOOKEEPER_CONNECTION_STRING, new ExponentialBackoffRetry(1000, 3));
                 client.start();
         
                 // 创建分布式锁
                 InterProcessLock lock = new InterProcessMutex(client, LOCK_PATH);
         
                 try {
                     // 尝试获取锁，最多等待10秒
                     if (lock.acquire(10, TimeUnit.SECONDS)) {
                         // 成功获取到锁后执行业务逻辑
                         System.out.println("Successfully acquired the distributed lock");
                         // 执行业务逻辑
                         Thread.sleep(5000); // 模拟业务处理时间
                     } else {
                         System.out.println("Failed to acquire the distributed lock within 10 seconds");
                     }
                 } finally {
                     // 释放锁
                     lock.release();
                 }
         
                 // 关闭客户端
                 client.close();
             }
         }
         ```

         - 在上面的示例中，首先创建了一个Curator客户端连接到ZooKeeper服务器。
         - 然后，使用`InterProcessMutex`创建了一个分布式锁对象，指定了ZooKeeper中的锁路径。
         - 在`try`块中，使用`lock.acquire()`方法尝试获取锁，在获取到锁后执行业务逻辑（这里用`Thread.sleep`模拟了业务处理）。
         - 最后，在`finally`块中释放锁，确保锁被正确地释放，以便其他客户端可以获取锁。
         - 最后，关闭Curator客户端以释放资源。

      注意事项：

      - **异常处理**：在实际应用中，需要适当处理连接失败、锁超时等异常情况。
      - **节点路径**：确保锁的路径在ZooKeeper中是唯一的，通常使用全局唯一的路径。
      - **性能和调优**：根据实际需求调整重试策略、超时时间和锁定路径，以优化性能和可靠性。

      使用Apache Curator可以大大简化分布式锁的实现和管理，同时利用ZooKeeper提供的强一致性保证，确保分布式系统中的资源访问顺序和数据一致性。

   3. 集群管理

## 1.2.zookeeper数据模型 

> Zookeeper 的数据模型可以简单描述为一个分层的文件系统，它类似于标准的文件系统，但设计用于存储小量的非常重要的数据。以下是 Zookeeper 的核心概念和数据模型的要点：

### 2.1. **层次命名空间**

> Zookeeper 的数据模型使用类似于文件系统路径的层次结构来组织数据，称为 **znode**。每个 znode 都可以存储数据和元数据，并且可以包含子节点。

### 2.2. **Znode**

- **Znode** 是 Zookeeper 中的基本数据单元，类似于文件系统中的文件或目录。
- 每个 Znode 都有一个唯一的路径标识符，称为 **路径（path）**，例如 `/myapp/config`.
- 每个 Znode 可以存储数据（小于 1MB）和元数据（如 ACL、时间戳等）。

### 2.3. **节点类型**

Zookeeper 中的 Znode 可以根据其用途和行为分为几种类型：

- **持久节点（Persistent Znode）**：创建后即存在，直到显式删除。
- **临时节点（Ephemeral Znode）**：与客户端会话绑定，客户端断开连接时自动删除。
- **持久顺序节点（Persistent Sequential Znode）**：与持久节点类似，但具有系统生成的唯一序号。
- **临时顺序节点（Ephemeral Sequential Znode）**：与临时节点类似，但具有系统生成的唯一序号。

### 2.4. **监听机制**

Zookeeper 支持监听机制，客户端可以监听特定 Znode 的变化（例如数据更新、节点删除等），并在状态发生变化时得到通知。

### 2.5. **数据一致性**

Zookeeper 保证了分布式环境下数据的一致性和可靠性，通过原子性操作和基于 Paxos 算法的分布式协议来实现。

### 2.6. **应用场景**

Zookeeper 最常见的用途是作为分布式系统的协调服务，例如用于配置管理、服务发现、分布式锁等。它的设计使得分布式系统可以在高可用性和一致性的环境中运行。

总体而言，Zookeeper 的数据模型简单而灵活，提供了强大的分布式协调功能，使得开发人员能够在分布式环境中轻松管理和协调应用程序的状态和配置信息。

## 3.zookeeper单机安装

### 3.1. **准备工作**

- 确保已安装Java环境，ZooKeeper依赖JDK 1.8或以上版本。

  查看Java版本：

  ```
  java -version
  ```

  如果没有Java，请安装：

  ```
  sudo yum install java-1.8.0-openjdk-devel   # CentOS/RHEL
  sudo apt-get install openjdk-8-jdk          # Ubuntu/Debian
  ```

### 3.2. **下载ZooKeeper**

- 从官方Apache ZooKeeper网站下载最新的稳定版： [ZooKeeper 下载地址](https://zookeeper.apache.org/releases.html)

  使用`wget`命令下载：

  ```
  wget https://downloads.apache.org/zookeeper/zookeeper-3.8.1/apache-zookeeper-3.8.1-bin.tar.gz
  ```

### 3.3. **解压ZooKeeper**

- 解压ZooKeeper压缩包并进入解压目录：

  ```
  tar -zxvf apache-zookeeper-3.8.1-bin.tar.gz
  cd apache-zookeeper-3.8.1-bin
  ```

### 3.4. **配置ZooKeeper**

- 进入`conf`目录并复制配置文件：

  ```
  bash复制代码cd conf
  cp zoo_sample.cfg zoo.cfg
  ```

- 编辑`zoo.cfg`文件：

  ```
  vim zoo.cfg
  ```

  关键配置项包括：

  ```
  # 数据保存目录
  dataDir=/var/lib/zookeeper
  
  # 客户端连接的端口
  clientPort=2181
  
  # 日志保存目录
  dataLogDir=/var/log/zookeeper
  ```

### 3.5. **启动ZooKeeper**

- 在ZooKeeper解压目录下运行：

  ```
  bin/zkServer.sh start
  ```

- 检查ZooKeeper状态：

  ```
  bin/zkServer.sh status
  ```

### 3.6. **ZooKeeper客户端连接**

- 使用ZooKeeper自带的客户端工具连接到本地ZooKeeper实例：

  ```
  bin/zkCli.sh -server 127.0.0.1:2181
  ```

### 3.7. **设置为系统服务（可选）**

- 如果希望将ZooKeeper设置为系统服务，可以创建一个`zookeeper.service`文件：

  创建服务文件：

  ```
  sudo vim /etc/systemd/system/zookeeper.service
  ```

  添加以下内容：

  ```
  [Unit]
  Description=Apache ZooKeeper
  After=network.target
  
  [Service]
  Type=forking
  ExecStart=/path/to/zookeeper/bin/zkServer.sh start
  ExecStop=/path/to/zookeeper/bin/zkServer.sh stop
  Restart=always
  User=zookeeper
  Group=zookeeper
  
  [Install]
  WantedBy=multi-user.target
  ```

- 启动并启用服务：

  ```
  sudo systemctl daemon-reload
  sudo systemctl start zookeeper
  sudo systemctl enable zookeeper
  ```

完成后，ZooKeeper将作为服务自动启动并运行。

### 3.8. **验证ZooKeeper安装**

- 在客户端输入

  ```
  stat
  ```

  命令查看服务器状态：

  ```
  echo stat | nc localhost 2181
  ```

这将显示ZooKeeper实例的状态。

## 4.zookeeper常用shell命令

### 4.1.服务端命令

1. 启动：./zkServer.sh start
2. 状态：./zkServer.sh status
3. 停止：./zkServer.sh stop
4. 重启：./zkServer.sh restart

### 4.2.客户端命令

1. 连接：./zkCli.sh -server localhost:2181
2. 断开连接：quit
3. 显示指定目录下节点：ls /
4. ls /zookeper
5. 创建节点：create /app1 itheima
6. 获取节点值：get /app1
7. 设置节点值：set /app1 hkk
8. 删除单个节点：delete /app1
9. create /app1/app1-1
10. create /app1/app1-2
11. delete /app1/app1-1
12. delete /app1：Node not empty ：/app1
13. 删除带有子节点的节点：deleteall /app1

## 5.zookeeper的Acl权限控制

## 6.zookeeper的javaApi

# 黑马程序员

> b站：https://www.bilibili.com/video/BV1M741137qY/

## 1.安装与配置

安装和配置 Zookeeper 非常简单，以下是基本的步骤和指南：

### 1. 下载 Zookeeper

首先，你需要下载 Zookeeper 的安装包。你可以从官方网站 [Apache Zookeeper](https://zookeeper.apache.org/) 下载最新的稳定版本。选择合适的版本下载，通常会是一个 `.tar.gz` 或 `.zip` 文件。

### 2. 解压安装包

下载完成后，将压缩包解压到你选择的安装目录，例如 `/opt` 或者 `/usr/local`：

```
tar -zxf zookeeper-x.x.x.tar.gz -C /opt
```

这会将 Zookeeper 解压到 `/opt/zookeeper-x.x.x` 目录中，其中 `x.x.x` 是版本号。

### 3. 配置 Zookeeper

接下来，你需要创建和配置 Zookeeper 的配置文件。在 Zookeeper 的安装目录下，有一个 `conf` 文件夹，里面包含了 `zoo_sample.cfg` 示例配置文件。你可以复制一份并命名为 `zoo.cfg`：

```
cp /opt/zookeeper-x.x.x/conf/zoo_sample.cfg /opt/zookeeper-x.x.x/conf/zoo.cfg
```

然后编辑 `zoo.cfg` 文件，配置 Zookeeper 的相关参数，例如：

```
vi /opt/zookeeper-x.x.x/conf/zoo.cfg
```

常见的配置项包括：

- `dataDir`：指定 Zookeeper 存储数据的目录。
- `clientPort`：指定客户端连接 Zookeeper 的端口，默认是 2181。
- **其他高级配置如需要集群配置等。**

### 4. 启动 Zookeeper

配置完成后，你可以启动 Zookeeper。进入 Zookeeper 的安装目录，执行如下命令：

```
cd /opt/zookeeper-x.x.x
bin/zkServer.sh start
```

### 5. 验证和测试

启动之后，可以使用 Zookeeper 提供的客户端工具进行连接和测试，例如：

```
bin/zkCli.sh -server localhost:2181
```

这将连接到本地运行的 Zookeeper 服务，并打开一个命令行界面，可以进行一些基本的操作和测试。

### 6. 配置自动启动 (可选)

> 如果需要将 Zookeeper 设置为系统启动时自动启动，可以根据你所用的操作系统进行相应的配置，通常需要修改启动脚本或者使用系统服务管理工具。

#### 6.1.**创建服务脚本文件**：

> 首先，创建一个 Zookeeper 的服务脚本文件，用于启动、停止和管理 Zookeeper 服务。

```
sudo vi /etc/systemd/system/zookeeper.service
```

1. **编辑服务脚本**：

   在打开的编辑器中，添加以下内容：

   ```
   ini复制代码[Unit]
   Description=Zookeeper Service
   After=network.target
   
   [Service]
   Type=simple
   User=zookeeper-user  # 设置运行 Zookeeper 的用户，推荐创建一个专门的用户来运行
   ExecStart=/path/to/zookeeper/bin/zkServer.sh start
   ExecStop=/path/to/zookeeper/bin/zkServer.sh stop
   Restart=on-abnormal
   
   [Install]
   WantedBy=multi-user.target
   ```

   - 将 `User` 替换为你希望运行 Zookeeper 的用户。
   - 将 `ExecStart` 和 `ExecStop` 的路径替换为你的 Zookeeper 安装路径下 `zkServer.sh` 的实际路径。

2. **保存并关闭文件**。

#### 6.2. 设置服务和启用自启动

1. **重新加载 systemd 配置**：

   ```
   sudo systemctl daemon-reload
   ```

2. **启动 Zookeeper 服务**：

   ```
   sudo systemctl start zookeeper
   ```

3. **设置开机自启动**：

   ```
   sudo systemctl enable zookeeper
   ```

#### 6.3. 验证和测试

现在，Zookeeper 应该已经启动，并且已经设置为在系统启动时自动启动。你可以通过以下命令验证服务的状态：

- 查看服务状态：

  ```
  sudo systemctl status zookeeper
  ```

- 如果一切正常，你会看到服务处于运行状态，并且已经设置为开机自启动

#### 6.4.启动失败原因

##### 6.4.1.检查日志文件.

启动失败时，首先应查看 Zookeeper 的日志文件，以查找详细的错误信息。通常日志文件位于 Zookeeper 安装目录下的 `logs` 文件夹中，主要关注 `zookeeper.out` 或 `zookeeper.log` 文件。

```
tail -f /path/to/zookeeper/logs/zookeeper.out
```

查看日志文件，可以帮助你找到具体的错误原因。

##### 6.4.2.检查配置文件

确保你的 Zookeeper 配置文件 (`zoo.cfg`) 中的参数设置正确。特别是以下几个关键的配置项：

- `dataDir`：指定 Zookeeper 存储数据的目录。
- `clientPort`：指定客户端连接 Zookeeper 的端口。
- 其他高级配置如需要集群配置等。

##### 6.4.3.检查权限和路径设置

确保 Zookeeper 启动脚本 (`zkServer.sh`) 可执行，并且设置了正确的路径和权限。推荐使用专门的用户来运行 Zookeeper，并确保该用户对相关目录和文件有读写权限。

##### 6.4.4.查看系统日志

除了 Zookeeper 的日志文件外，还应该查看系统的日志文件（如 `/var/log/messages` 或 `/var/log/syslog`），可能会记录有关 Zookeeper 启动失败的额外信息。

##### 6.4.5.查看端口占用

确保 Zookeeper 使用的端口（通常是 `clientPort` 和可能的 `quorumPort`）没有被其他应用程序占用。

##### 6.4.6.手动启动和调试

尝试手动启动 Zookeeper，并观察输出的错误信息：

```
/path/to/zookeeper/bin/zkServer.sh start
```

这样可以实时查看启动过程中的错误信息

##### 6.4.7.确认JAVA_HOME设置正确

最后，请确保在设置 Zookeeper 的启动脚本或者服务文件时，`JAVA_HOME` 环境变量已正确设置，并且可以正确找到 Java 的安装路径。

## 2.命令操作

## 3.JavaApi操作

## 4.集群搭建

## 5.核心理论