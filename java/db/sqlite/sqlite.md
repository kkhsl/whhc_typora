# 1.sql基本命令

## 1.1.创建和连接数据库

SQLite数据库是存储在文件中的。可以通过以下命令创建并连接到数据库（如果文件不存在，会自动创建）：

```
sqlite3 mydatabase.db
```

这将打开SQLite命令行界面并连接到`mydatabase.db`数据库。

## 1.2.创建表

在命令行界面中，可以使用以下命令创建一个表：

```
CREATE TABLE students (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL
);
```

## 1.3.插入数据

使用以下命令插入数据：

```
INSERT INTO students (name) VALUES ('John Doe');
INSERT INTO students (name) VALUES ('Jane Smith');
```

## 1.4.查询数据

使用以下命令查询数据：

```
SELECT * FROM students;
```

## 1.5.更新数据

使用以下命令更新数据：

```
UPDATE students
SET name = 'John Smith'
WHERE id = 1;
```

## 1.6.删除数据

使用以下命令删除数据：

```
DELETE FROM students
WHERE id = 1;
```

## 1.7.退出SQLite命令行

使用以下命令退出SQLite命令行界面：

```
.quit
```



# 2.配置linux，允许远程访问sqlite

## 2.1.centos nginx 增加 php-fpm

在 CentOS 上配置 Nginx 以支持 PHP-FPM，主要涉及安装 PHP 和 PHP-FPM 模块，然后配置 Nginx 以将 PHP 请求转发给 PHP-FPM 进程池。以下是详细步骤：

### 步骤概述：

1. **安装 PHP 和 PHP-FPM**
2. **配置 PHP-FPM**
3. **配置 Nginx 支持 PHP**

### 详细步骤：

#### 1. 安装 PHP 和 PHP-FPM

首先，确保系统已经安装了 Nginx。然后，通过以下命令安装 PHP 和 PHP-FPM：

```
sudo yum install epel-release
sudo yum install php php-fpm
```

这将安装 PHP 和 PHP-FPM，同时安装 PHP 扩展和依赖项。

#### 2. 配置 PHP-FPM

编辑 PHP-FPM 配置文件 `/etc/php-fpm.d/www.conf`，调整配置以适应您的需求。主要配置包括监听地址、用户和组、进程数等。以下是一些常用配置示例：

```
sudo vi /etc/php-fpm.d/www.conf
```

示例配置：

```
listen = /var/run/php-fpm/php-fpm.sock
listen.owner = nginx
listen.group = nginx
user = nginx
group = nginx
```

确保 PHP-FPM 的监听地址与 Nginx 配置中的 `fastcgi_pass` 地址一致。

#### 3. 配置 Nginx 支持 PHP

编辑 Nginx 的配置文件，通常位于 `/etc/nginx/nginx.conf` 或 `/etc/nginx/conf.d/default.conf`，添加 PHP 支持配置段。

```
sudo vi /etc/nginx/nginx.conf
```

示例配置段：

```
server {
    listen       80;
    server_name  localhost;

    root   /usr/share/nginx/html;
    index  index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include       fastcgi_params;
        fastcgi_pass  unix:/var/run/php-fpm/php-fpm.sock; # PHP-FPM 监听地址
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

### 重启服务

完成配置后，分别重启 PHP-FPM 和 Nginx 服务，使更改生效。

```
sudo systemctl restart php-fpm
sudo systemctl restart nginx
```

### 测试

创建一个简单的 `info.php` 文件来测试 PHP 是否正常工作：

```
sudo vi /usr/share/nginx/html/info.php
```

在 `info.php` 文件中添加以下内容：

```
<?php
phpinfo();
?>
```

保存并退出文件。然后在浏览器中访问 `http://your-server-ip/info.php`，应该能看到 PHP 的信息页面，表明 PHP 成功与 Nginx 集成。

这些步骤应该能帮助您在 CentOS 上配置 Nginx 以支持 PHP-FPM。根据您的具体需求和环境，可能需要进一步调整 PHP-FPM 和 Nginx 的配置以达到最佳性能和安全性。



## 2.2.配置nginx将info.php文件放到其他目录

要配置 Nginx 将 `info.php` 文件放到其他目录，您需要调整 Nginx 的配置文件，特别是 `location` 块中的 `root` 指令。以下是具体的步骤和示例：

### 步骤：

1. **编辑 Nginx 配置文件**
2. **配置 Nginx 的 location 块**

### 详细步骤：

#### 1. 编辑 Nginx 配置文件

通常，Nginx 的主配置文件位于 `/etc/nginx/nginx.conf` 或者 `/etc/nginx/conf.d/default.conf`。找到您的服务器块（`server` 块），并编辑以适应新的路径。

```
sudo vi /etc/nginx/nginx.conf
```

#### 2. 配置 Nginx 的 location 块

在 `server` 块内，找到处理 PHP 的 `location` 块。通常，它看起来像这样：

```
server {
    listen       80;
    server_name  localhost;

    root   /usr/share/nginx/html;
    index  index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include       fastcgi_params;
        fastcgi_pass  unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

要将 `info.php` 文件放到其他目录，例如 `/var/www/html`，可以添加一个新的 `location` 块来处理这个目录的请求：

```
server {
    listen       80;
    server_name  localhost;

    root   /usr/share/nginx/html;
    index  index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location /other-directory {
        alias /var/www/html;
        index  index.php;

        location ~ \.php$ {
            include       fastcgi_params;
            fastcgi_pass  unix:/var/run/php-fpm/php-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $request_filename;
        }
    }

    location ~ \.php$ {
        include       fastcgi_params;
        fastcgi_pass  unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

#### 解释配置说明：

- `location /other-directory`：这是新添加的 `location` 块，用于处理 `/other-directory` 目录下的请求。
- `alias /var/www/html;`：指定 `location` 中的路径别名，将请求映射到实际的文件系统路径。
- `index index.php;`：如果请求的是目录，默认显示 `index.php` 文件。
- `location ~ \.php$`：在 `/other-directory` 的 `location` 块内，用于处理 PHP 文件的 `location` 块。
- `SCRIPT_FILENAME $request_filename;`：用于设置 FastCGI 参数，指定 PHP 文件的路径。

### 注意事项：

- 在修改 Nginx 配置后，使用 `sudo nginx -t` 命令检查配置文件的语法是否正确。
- 使用 `sudo systemctl reload nginx` 命令重新加载 Nginx 配置文件，使更改生效。

通过以上配置，您可以将 `info.php` 文件或其他 PHP 文件放置到指定的目录，并通过 Nginx 进行访问和执行。



## 2.3.centos php 增加sqlite3扩展

**检查 PHP 扩展**：确保 PHP 中包含了 SQLite3 扩展。您可以通过检查 PHP 的 `phpinfo()` 输出或查看 PHP 配置文件来确定。

在 CentOS 上增加 PHP 的 SQLite3 扩展，您可以通过以下步骤来实现。SQLite3 是 PHP 中与 SQLite 数据库交互的扩展，需要确保它已经正确安装和配置。

### 步骤概述：

1. **安装 PHP SQLite3 扩展**
2. **重启 PHP-FPM 或 Web 服务器**

### 详细步骤：

#### 1. 安装 PHP SQLite3 扩展

首先，使用 `yum` 包管理器安装 SQLite3 扩展。在 CentOS 上，SQLite3 扩展通常随 PHP 一起分发，但是您可能需要手动安装它。

```
sudo yum install php-sqlite3
```

这将安装 PHP 的 SQLite3 扩展以及它的依赖项。如果您使用的是不同的 PHP 版本，可能需要替换 `php-sqlite3` 为适合您 PHP 版本的包名。

#### 2. 重启 PHP-FPM 或 Web 服务器

安装完成后，需要重新启动 PHP-FPM（如果您使用 PHP-FPM）或 Web 服务器（如 Nginx、Apache）以使更改生效。

```
sudo systemctl restart php-fpm
```

或者，

```
sudo systemctl restart nginx
```

具体操作根据您的环境和 Web 服务器配置略有不同。

#### 3. 验证安装

为了验证安装是否成功，您可以创建一个简单的 PHP 文件来输出 PHP 的 `phpinfo()` 函数的信息，并查看是否有 SQLite3 扩展列出。

创建一个名为 `info.php` 的文件：

```
sudo vi /usr/share/nginx/html/info.php
```

在文件中添加以下内容：

```
<?php
phpinfo();
?>
```

保存并关闭文件。然后在浏览器中访问 `http://your-server-ip/info.php`，找到 SQLite3 扩展部分，确认它已经正确安装和启用。

### 注意事项：

- 如果您在安装时遇到问题，可以查看 `php-sqlite3` 包的详细信息和依赖关系，以确定问题所在。
- 确保您具有适当的权限来安装软件包和重新启动服务。

通过这些步骤，您应该能够成功在 CentOS 上增加 PHP 的 SQLite3 扩展，并使其能够与 SQLite 数据库进行交互。



# 3.java集成Sqlite、mybatis