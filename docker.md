# 1.安装

## 1.1.CentOS 7

### 1. 安装 Docker 依赖包

首先，确保你的系统已经更新到最新版本，并安装一些依赖包：

```
sudo yum update -y
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 2. 添加 Docker 官方仓库

CentOS 7 默认的软件库中包含了 Docker，但版本可能不如 Docker 官方提供的最新。为了获取最新版本，首先添加 Docker 的官方仓库：

```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### 3. 安装 Docker

使用 `yum` 安装 Docker：

```
sudo yum install -y docker-ce docker-ce-cli containerd.io
```

### 4. 启动 Docker 服务

安装完成后，启动 Docker 服务并设置为开机自动启动：

```
sudo systemctl start docker
sudo systemctl enable docker
```

### 5. 检查 Docker 是否安装成功

运行以下命令查看 Docker 版本，以确认安装成功：

```
docker --version
```

如果输出类似以下内容，说明 Docker 安装成功：

```
Docker version 20.10.24, build 3a2c30b
```

### 6. 运行 Docker 测试容器

为了确认 Docker 能正常工作，可以运行一个测试容器，比如 `hello-world`：

```
sudo docker run hello-world
```

如果 Docker 正常安装和配置，这个命令会输出一条“Hello from Docker!”的消息，表示 Docker 已成功安装。

### 7. 配置非 root 用户（可选）

默认情况下，Docker 命令需要 `root` 权限。如果你希望以非 root 用户运行 Docker 命令，可以将你的用户添加到 Docker 组中：

```
sudo usermod -aG docker $USER
```

添加完成后，注销并重新登录，或者运行以下命令使更改生效：

```
newgrp docker
```

然后你就可以无需 `sudo` 直接运行 Docker 命令了。

### 8. 防火墙设置（可选）

如果你在使用 Docker 时遇到防火墙问题，可以打开 Docker 所需的端口。以下是打开默认 Docker 端口（如 2375、2376 等）的命令：

```
sudo firewall-cmd --permanent --add-port=2375/tcp
sudo firewall-cmd --reload
```

### 9. 验证 Docker 是否正常运行

最后，使用以下命令确认 Docker 正在运行：

```
sudo systemctl status docker
```

你应该会看到 Docker 服务的状态为 "active (running)"。

## 1.2.Ubuntu

在 Ubuntu 上安装 Docker 非常简单，下面是详细的步骤：

### 1. 更新系统

首先，确保你的系统软件包列表是最新的：

```
sudo apt-get update
```

### 2. 安装必要的依赖包

Docker 安装需要一些依赖包，所以首先需要安装它们：

```
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```

### 3. 添加 Docker 官方的 GPG 密钥

Docker 的安装包需要通过官方的 GPG 密钥来验证其完整性，首先添加 Docker 的 GPG 密钥：

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### 4. 添加 Docker 官方的 APT 源

接下来，我们需要添加 Docker 官方的 APT 仓库：

```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 5. 更新软件包列表

添加完 Docker 的仓库后，更新本地的软件包列表：

```
sudo apt-get update
```

### 6. 安装 Docker CE (Community Edition)

现在，安装 Docker 社区版：

```
sudo apt-get install -y docker-ce
```

### 7. 启动 Docker 服务

安装完成后，启动 Docker 服务并设置为开机自启：

```
sudo systemctl start docker
sudo systemctl enable docker
```

### 8. 检查 Docker 是否安装成功

检查 Docker 版本，确认 Docker 已经安装成功：

```
docker --version
```

如果 Docker 安装成功，你将看到类似以下的输出：

```
Docker version 20.10.24, build 3a2c30b
```

### 9. 运行 Docker 测试容器

为了确认 Docker 是否正常运行，可以通过运行 `hello-world` 镜像来做一个简单的测试：

```
sudo docker run hello-world
```

如果 Docker 正常运行，它会输出类似以下内容：

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### 10. 配置非 root 用户运行 Docker（可选）

默认情况下，Docker 需要 root 权限来运行命令。如果你希望以非 root 用户运行 Docker，可以将用户添加到 `docker` 组：

```
sudo usermod -aG docker $USER
```

然后注销并重新登录，或者运行以下命令使更改生效：

```
newgrp docker
```

### 11. 安装 Docker Compose（可选）

如果你需要使用 Docker Compose 来管理多个容器，可以按照以下步骤安装 Docker Compose：

1. 下载 Docker Compose 二进制文件：

   ```
   sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

2. 给予执行权限：

   ```
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. 检查 Docker Compose 是否安装成功：

   ```
   docker-compose --version
   ```

这样，你就成功安装了 Docker 和 Docker Compose，开始管理容器和应用了。

# 2.常用命令

> **容器、镜像、网络、卷、compose**

### **1. 容器操作**

- **查看运行中的容器**

  ```
  docker ps
  ```

- **查看所有容器（包括停止的容器）**

  ```
  docker ps -a
  ```

- **启动容器**

  ```
  docker start <container_id or container_name>
  ```

- **停止容器**

  ```
  docker stop <container_id or container_name>
  ```

- **重启容器**

  ```
  docker restart <container_id or container_name>
  ```

- **删除容器**

  ```
  docker rm <container_id or container_name>
  ```

- **查看容器的详细信息**

  ```
  docker inspect <container_id or container_name>
  ```

- **进入正在运行的容器**

  ```
  docker exec -it <container_id or container_name> /bin/bash
  ```

- **查看容器的日志**

  ```
  docker logs <container_id or container_name>
  ```

- **查看容器的资源使用情况**

  ```
  docker stats <container_id or container_name>
  ```

### **2. 镜像操作**

- **列出所有本地镜像**

  ```
  docker images
  ```

- **拉取镜像**

  ```
  docker pull <image_name>
  ```

- **删除镜像**

  ```
  docker rmi <image_id or image_name>
  ```

- **构建镜像（从 Dockerfile 构建）**

  ```
  docker build -t <image_name> <path_to_dockerfile>
  ```

- **查看镜像的详细信息**

  ```
  docker inspect <image_id or image_name>
  ```

### **3. 容器和镜像管理**

- **创建并运行一个新容器**

  ```
  docker run -it <image_name> /bin/bash
  ```

- **创建一个容器并在后台运行**

  ```
  docker run -d <image_name>
  ```

- **运行一个临时容器并执行命令**

  ```
  docker run --rm <image_name> <command>
  ```

- **运行容器时指定端口映射**

  ```
  docker run -d -p 8080:80 <image_name>
  ```

- **运行容器并挂载卷（用于数据持久化）**

  ```
  docker run -d -v <host_path>:<container_path> <image_name>
  ```

- **运行容器并设置环境变量**

  ```
  docker run -d -e <env_var_name>=<value> <image_name>
  ```

### **4. Docker 网络**

- **查看 Docker 网络**

  ```
  docker network ls
  ```

- **创建自定义网络**

  ```
  docker network create <network_name>
  ```

- **连接容器到网络**

  ```
  docker network connect <network_name> <container_id or container_name>
  ```

- **断开容器与网络的连接**

  ```
  docker network disconnect <network_name> <container_id or container_name>
  ```

### **5. Docker 卷（Volume）**

- **查看所有卷**

  ```
  docker volume ls
  ```

- **创建卷**

  ```
  docker volume create <volume_name>
  ```

- **挂载卷到容器**

  ```
  docker run -d -v <volume_name>:<container_path> <image_name>
  ```

- **删除卷**

  ```
  docker volume rm <volume_name>
  ```

### **6. Docker Compose（用于管理多容器应用）**

- **启动多容器应用**

  ```
  docker-compose up
  ```

- **在后台启动多容器应用**

  ```
  docker-compose up -d
  ```

- **停止多容器应用**

  ```
  docker-compose down
  ```

- **查看容器的日志**

  ```
  docker-compose logs
  ```

- **查看 Docker Compose 服务的状态**

  ```
  docker-compose ps
  ```

### **7. 查看和调试容器**

- **查看容器的资源使用情况**

  ```
  docker stats
  ```

- **查看所有容器的进程**

  ```
  docker top <container_id or container_name>
  ```

- **查看容器运行时的文件系统（挂载容器）**

  ```
  docker cp <container_id>:/path/to/file /host/path
  ```

- **获取容器内的文件**

  ```
  docker cp <container_id>:<path_to_file_in_container> <host_path>
  ```

### **8. 清理未使用的资源**

- **删除未使用的容器、镜像、卷和网络**

  ```
  docker system prune
  ```

- **删除未使用的镜像**

  ```
  docker image prune
  ```

- **删除未使用的容器**

  ```
  docker container prune
  ```

- **删除未使用的卷**

  ```
  docker volume prune
  ```

这些是 Docker 中最常用的一些命令。通过这些命令，你可以管理容器、镜像、网络和数据卷等。



# 3.安装第三方软件

## 3.1.ELK

使用 Docker 安装 ELK（Elasticsearch, Logstash, Kibana）并创建网络的步骤如下：

### 3.1.1. 创建 Docker 网络

首先，我们需要创建一个 Docker 网络，确保 ELK 各个组件能够互相通信。

```
docker network create elk-network
```

### 3.1.2. 拉取 ELK 镜像

接下来，拉取 Elasticsearch、Logstash 和 Kibana 镜像：

```
# 拉取 Elasticsearch 镜像
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.6.0

# 拉取 Logstash 镜像
docker pull docker.elastic.co/logstash/logstash:8.6.0

# 拉取 Kibana 镜像
docker pull docker.elastic.co/kibana/kibana:8.6.0
```

### 3.1.3. 启动 Elasticsearch 容器

启动 Elasticsearch 容器并将其连接到 `elk-network` 网络：

```
docker run --name elasticsearch --net elk-network -d \
  -e "discovery.type=single-node" \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  -p 9200:9200 \
  docker.elastic.co/elasticsearch/elasticsearch:8.6.0
```

```
  docker run --name elasticsearch --net elk-network -d \
  -e "discovery.type=single-node" \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  -e "network.host=0.0.0.0" \
  -p 9200:9200 \
  -p 9300:9300 \
  --health-cmd="curl --silent --fail http://localhost:9200/_cat/health || exit 1" \
  --health-interval=60s \
  --health-retries=5 \
  --health-timeout=10s \
  --health-start-period=1m \
  docker.elastic.co/elasticsearch/elasticsearch:8.6.0
```



- `discovery.type=single-node` 表示 Elasticsearch 在单节点模式下运行。
- `ES_JAVA_OPTS=-Xms512m -Xmx512m` 设置 Java 堆内存。
- `-p 9200:9200` 映射本地端口 9200 到容器的 9200 端口，供 Elasticsearch API 使用。
- -p 9300:9300 节点互连端口（黑马）
- -v es-data:/usr/share/elasticsearch/data \（黑马）
- -v es-plugins:/usr/share/elasticsearch/plugins  \（黑马）

### 3.1.4. 启动 Logstash 容器

启动 Logstash 容器并将其连接到 `elk-network` 网络：

```
docker run --name logstash --net elk-network -d \
  -p 5044:5044 \
  -v /path/to/logstash/config:/usr/share/logstash/config \
  docker.elastic.co/logstash/logstash:8.6.0
```

- `-p 5044:5044` 映射 Logstash 的输入端口。
- `-v /path/to/logstash/config:/usr/share/logstash/config` 将本地的 Logstash 配置文件挂载到容器中。

### 3.1.5. 启动 Kibana 容器

启动 Kibana 容器并将其连接到 `elk-network` 网络：

```
docker run --name kibana --net elk-network -d \
  -p 5601:5601 \
  docker.elastic.co/kibana/kibana:8.6.0
```

```
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://elasticsearch \
--network=elk-network \
-p 5601:5601 \
docker.elastic.co/kibana/kibana:8.6.0
```



- `-p 5601:5601` 映射 Kibana 的 Web 界面端口。

### 3.1.6. 验证 ELK 是否启动成功

- 访问 Elasticsearch: http://localhost:9200
- 访问 Kibana: http://localhost:5601

### 3.1.7. 停止和删除容器

如果需要停止或删除容器，可以使用以下命令：

```
# 停止容器
docker stop elasticsearch logstash kibana

# 删除容器
docker rm elasticsearch logstash kibana
```

通过这些步骤，您可以在 Docker 中成功安装并运行 ELK 堆栈，并确保它们在自定义网络中相互连接。
