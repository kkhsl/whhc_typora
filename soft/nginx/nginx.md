# 1.安装

要卸载现有的 Nginx，并重新安装并启用 `stream` 模块，您可以按照以下步骤进行操作：

### 步骤 1: 卸载现有的 Nginx

如果您是通过 `yum` 安装的 Nginx，首先需要卸载它：

```
sudo yum remove nginx
```

这将删除当前安装的 Nginx 版本。

### 步骤 2: 下载并安装支持 `stream` 模块的 Nginx 源码

接下来，您需要下载 Nginx 源代码并编译安装，启用 `stream` 模块。

#### 1. **下载 Nginx 源代码**

首先，访问 Nginx 官方下载页面，或者直接使用 `wget` 命令下载：

```
wget http://nginx.org/download/nginx-1.22.0.tar.gz
tar -zxvf nginx-1.22.0.tar.gz
cd nginx-1.22.0
```

如果您需要其他版本的 Nginx，可以修改版本号。

#### 2. **安装必需的依赖**

在编译之前，您需要安装一些依赖包：

```
sudo yum install -y gcc pcre-devel zlib-devel make openssl-devel
```

这些包是编译 Nginx 所必需的。

#### 3. **配置 Nginx 以启用 `stream` 模块**

在 Nginx 源代码目录中，运行 `./configure` 命令来启用 `stream` 模块：

```
./configure --with-stream
```

如果您需要启用其他模块（例如 `ssl` 模块），可以在此命令中添加额外的选项。例如：

```
./configure --with-stream --with-http_ssl_module
```

#### 4. **编译并安装 Nginx**

配置完成后，使用以下命令进行编译和安装：

```
make
sudo make install
```

安装完成后，Nginx 将被安装到 `/usr/local/nginx` 目录中。（**默认的安装路径是 `/usr/local/nginx`**，由./configure脚本会自动决定安装目录）

### 步骤 3: 启动 Nginx

安装完成后，您可以启动 Nginx：

```

sudo /usr/local/nginx/sbin/nginx
```

### 步骤 4: 配置 Nginx 使用 `stream` 模块

编辑 Nginx 配置文件，启用 `stream` 模块来配置 TCP 或 UDP 代理。

编辑 Nginx 配置文件（默认为 `/usr/local/nginx/conf/nginx.conf`）：

```
sudo vi /usr/local/nginx/conf/nginx.conf
```

在文件中添加 `stream` 配置，例如：

```
stream {
    server {
        listen 8549;
        proxy_pass 127.0.0.1:8549;
    }
}
```

保存文件并退出编辑器。

### 步骤 5: 验证配置

在修改完配置文件后，您可以验证 Nginx 配置是否正确：

```
sudo /usr/local/nginx/sbin/nginx -t
```

如果输出显示没有错误，可以重启 Nginx：

```
sudo /usr/local/nginx/sbin/nginx -s reload
```

### 步骤 6: 检查是否启用 `stream` 模块

您可以运行以下命令验证 `stream` 模块是否启用：

```
/usr/local/nginx/sbin/nginx -V 2>&1 | grep -- '--with-stream'
```

如果看到 `--with-stream`，则表示 `stream` 模块已经成功启用。

### 总结

- 您首先需要卸载当前通过 `yum` 安装的 Nginx。
- 然后，您可以通过编译安装 Nginx，并在配置时启用 `stream` 模块。
- 安装完成后，您可以配置 Nginx 以支持 TCP/UDP 代理，启动并验证 Nginx 配置。

如果有任何问题，随时向我提问！



# 2.配置启动服务

在 Linux 上通过 `systemd` 启动和管理服务是常见的做法。如果您想要配置和启动一个新的服务（例如，Nginx），可以按照以下步骤进行操作。

### 1. **创建 Systemd 服务文件**

首先，您需要为 Nginx 创建一个 `systemd` 服务文件，以便能够管理 Nginx 的启动、停止、重启等。

#### 步骤：

1. **创建服务文件**
   创建 Nginx 的 `systemd` 服务文件，路径通常在 `/etc/systemd/system/` 下。使用以下命令创建并编辑文件：

   ```
   sudo vi /etc/systemd/system/nginx.service
   ```

2. **编写服务配置**
   在该文件中，输入以下内容：

   ```
   [Unit]
   Description=The nginx web server
   Documentation=http://nginx.org/en/docs/
   After=network.target
   
   [Service]
   Type=forking
   PIDFile=/var/run/nginx.pid
   ExecStart=/usr/local/nginx/sbin/nginx
   ExecReload=/usr/local/nginx/sbin/nginx -s reload
   ExecStop=/usr/local/nginx/sbin/nginx -s stop
   PrivateTmp=true
   
   [Install]
   WantedBy=multi-user.target
   ```

   - `ExecStart`: 指定 Nginx 的启动命令，这里假设 Nginx 安装在 `/usr/local/nginx/sbin/nginx`，如果通过 `yum` 安装，路径会是 `/usr/sbin/nginx`。
   - `ExecReload`: 重新加载 Nginx 配置文件。
   - `ExecStop`: 停止 Nginx 服务。

3. **保存并退出**
   保存文件并退出编辑器。

### 2. **重载 `systemd` 管理器**

添加完服务文件后，需要重载 `systemd` 配置，以便它能够识别新添加的服务：

```
sudo systemctl daemon-reload
```

### 3. **启用并启动 Nginx 服务**

#### 启用服务：

如果您希望 Nginx 在系统启动时自动启动，可以启用该服务：

```
sudo systemctl enable nginx
```

#### 启动服务：

使用以下命令启动 Nginx 服务：

```
sudo systemctl start nginx
```

### 4. **检查服务状态**

您可以通过以下命令检查 Nginx 服务的状态：

```
sudo systemctl status nginx
```

### 5. **配置防火墙允许 80 和 443 端口**

确保防火墙允许访问 Nginx 服务所需的端口。对于 HTTP 和 HTTPS 服务，您需要开放 80 和 443 端口：

```
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --zone=public --add-service=https --permanent
sudo firewall-cmd --reload
```

### 6. **重新加载和停止服务**

如果需要重新加载 Nginx 配置，使用：

```
sudo systemctl reload nginx
```

停止服务使用：

```
sudo systemctl stop nginx
```

### 总结

通过上述步骤，您可以为 Nginx 创建一个 `systemd` 服务文件，使其能够像其他系统服务一样管理和启动。如果您安装了 Nginx 并且配置了 `stream` 模块，还可以通过类似的方法启动和管理其他服务。



# 3.配置

## 3.1.proxy_pass

```
location /proxy/ {
	proxy_pass http://127.0.0.1/;
}
# 代理到URL：http://127.0.0.1/test.html


location /proxy/ {
	proxy_pass http://127.0.0.1;
}
# 代理到URL：http://127.0.0.1/proxy/test.html


location /proxy/ {
	proxy_pass http://127.0.0.1/aaa/;
}
# 代理到URL：http://127.0.0.1/aaa/test.html


location /proxy/ {
	proxy_pass http://192.168.137.103/aa;
}
# 代理到 URL：http://127.0.0.1/aaatest.html，实际测试结果有点差异，后期在看原因


```

# 
