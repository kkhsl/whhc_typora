# 1.ElasticSearch

> Elasticsearch（日志管理）
>
> elasticsearch采用倒排索引：
>
> - 文档（document）：每条数据就是一个文档
> - 词条（term）：文档按照语义分成的词语

## 1.1.应用场景

> 近乎实时、检索数据；扩展性好，处理PB级别的数据；海量数据的搜索、分析、计算

1. 
   全文搜索
2. 日志分析
   1. **服务器日志分析**：开发人员和运维人员可以使用 Elasticsearch 来收集服务器日志（如 Apache、Nginx、Tomcat 等），对这些日志进行索引，方便查询和分析。
   2. **应用日志分析**：应用程序（如后台服务、微服务等）会生成大量日志，Elasticsearch 可用来集中管理这些日志数据，帮助开发者快速定位问题和分析性能瓶颈。
   3. **异常监测与报警**：日志数据通常包含系统或应用的错误信息，通过实时分析，能够快速检测到异常（如错误日志、异常栈信息等），并且可与其他工具集成，自动触发报警通知。

3. **运维监控**
4. 数据库（mysql等）负责事务类型操作

## 1.2.安装

> 见docker文档

## 1.3.分词器

> IK分词器（analysis-ik）https://www.alibabacloud.com/help/zh/es/user-guide/use-the-analysis-ik-plug-in



### 1.规则

> IK**分词器的分词规则**
>
> IK分词插件的分词器的分词规则包括ik_smart和ik_max_word两种：

- ik_max_word：将文本按照最细粒度进行拆分，适合术语查询。
- ik_smart：将文本按照粗粒度进行拆分，适合短语查询。

### 2.作用

- 创建倒排索引时对文档分词
- 用户搜索时，对输入内容分词

## 1.4.索引库

#### 1.mapping

> mapping是对索引库中文档的约束，常见的mapping属性包括：
>
> 字段名、数据类型、是否参与搜索、是否分词、分词器

1. mapping常见属性

   - type：数据类型

     - 字符串：text（可分词文本）、keyword（精确值，例如：品牌、国家、ip地址）

     - 数字：long、integer、short、byte、double、float
     - 布尔：boolean
     - 日期：date
     - 对象：object

   - index：是否索引

   - analyzer：分词器

   - properties：子字段

#### 2.创建索引库

ES中通过Restful请求操作索引库、文档。请求内容用DSL语句来表示。创建索引库和mapping的DSL语法如下：

```json
PUT /索引名称
{
    "mappings":{
        "properties":{
            "字段名":{
                "type":"text",
                "analyzer":"ik_smart"
            },
            "字段名2":{
                "type":"keyword",
                "index":"false"
            },
            "字段名3":{
                "properties":{
                    "子字段":{
                        "type":"keyword"
                    }
                }
            },
            /// 略
        }
    }
}
```

```json
PUT /heima
{
	"mappings":{
		"properties":{
			"info":{
				"type":"text",
				"analyzer":"ik_smart"
			},
			"email":{
				"type":"keyword",
				"index":false
			},
			"name":{
			  "type":"object",
				"properties":{
					"firstName":{
						"type":"keyword"
					},
					"lastName":{
						"type":"keyword"
					}
				}
			}
		}
	}
}
```

#### 3.查看、删除

```
GET /索引库名

示例：
GET /heima
```

```
DELETE /删除索引库

示例：
DELETE /heima
```

#### 4.修改索引库

> 索引库和mapping一旦创建无法修改，但是可以添加新的字段

```json
PUT /索引库名/_mapping
{
	"properties":{
		"新字段名"：{
			"type":"integer"
		}
	}
}

```

## 1.5.文档

#### 1.新增文档

```json
POST /索引库名/_doc/文档id
{
	"字段1":"值1",
    "字段2":"值2",
    "字段3":{
    	"子属性1":"值1",
    	"子属性2":"值2",
    }
}

POST /heima/_doc/1
{
	"info":"黑马程序员Java讲师",
	"email":"123456@itcast.cn",
	"name":{
		"firstName":"云",
		"lastName":"赵"
	}
}
```

#### 2.查看删除文档

```
# 查看文档
GET /索引库名/_doc/1

GET /heima/_doc/1

# 删除文档
DELETE /索引库名/_doc/1

DELETE /heima/_doc/1
```

#### 3.修改文档

> 全量修改，会删除旧文档，添加新文档

```
PUT /索引库名/_doc/文档id
{
	"字段1":"值1"，
	"字段2":"值2"
}

PUT /heima/_doc/1
{
	"info":"黑马程序员高级java工程师",
	"email":"123456@163.com",
	"name":{
		"firstName":"云",
		"lastName":"赵"
	}
}

```

> 局部修改

```
POST /索引库名/_update/1
{
	"doc":{
		"字段名":"新值"
	}
}

POST /heima/_update/1
{
	"doc":{
		"email":"123456@189.cn"
	}
}
```

## 1.6.RestClient操索引库



### .注

1. ES官方提供了各种不通语言的客户端，用来操作ES。这些客户端的本质就是组装DSL语句，通过http请求发送给ES。官方文档地址：https://www.elastic.co/guide/en/elasticsearch/client/index.html

# 2.Logstash

> Logstash（数据搜索）
>
> **Logstash 需要一个配置文件来定义输入、过滤和输出。通常，这个配置文件的后缀名为 `.conf`，例如 `logstash.conf`。确保你已经有一个配置文件**
>
> Logstash事件处理有三个阶段：inputs → filters → outputs。是一个接收，处理，转发日志的工具。支持系统日志，webserver日志，错误日志，应用日志，总之包括所有可以抛出来的日志类型。

1. 常见的**输入**行为
   1. file：从文件系统中读取，类似tail -f命令
   2. syslog：在514端口上监听系统日志消息，并根据RFC3164标准进行解析（VPN远程读取日志）
   3. redis：从redis service中读取
   4. beats：从filebeat中读取
   5. Filters：数据中间处理，对数据进行操作
2. 常见的过**滤器**行为
   1. grok：解析任意文本数据，Grok 是 Logstash 最重要的插件。它的主要作用就是将文本格式的字符串，转换成为具体的结构化的数据，配合正则表达式使用。内置120多个解析语法。
   2. mutate：对字段进行转换。例如对字段进行删除、替换、修改、重命名等。
   3. drop：丢弃一部分events不进行处理。
   4. clone：拷贝 event，这个过程中也可以添加或移除字段。
   5. geoip：添加地理信息(为前台kibana图形化展示使用)
   6. Outputs：outputs是logstash处理管道的最末端组件。一个event可以在处理过程中经过多重输出，但是一旦所有的outputs都执行结束，这个event也就完成生命周期。
3. 一些常见的**outputs**为：
   1. elasticsearch：可以高效的保存数据，并且能够方便和简单的进行查询。
   2. file：将event数据保存到文件中。
   3. graphite：将event数据发送到图形化组件中，一个很流行的开源存储图形化展示的组件。
   4. Codecs：codecs 是基于数据流的过滤器，它可以作为input，output的一部分配置。Codecs可以帮助你轻松的分割发送过来已经被序列化的数据。
4. 一些常见的codecs：
   1. json：使用json格式对数据进行编码/解码。
   2. multiline：将汇多个事件中数据汇总为一个单一的行。比如：java异常信息和堆栈信息。

## 2.1.应用场景

**Logstash** 是 Elastic Stack（原 ELK Stack）中的一个关键组件，用于数据收集、处理和转发，特别适用于日志和事件数据的处理。它能够从多种数据源收集数据，进行过滤、解析和转换，然后将数据发送到 Elasticsearch、Kafka 或其他目标系统进行存储和分析。

以下是 **Logstash** 的一些常见应用场景：

### 1. **日志收集与集中管理**

Logstash 最常见的应用场景是 **日志收集与集中管理**。它可以从各种日志源（例如 Web 服务器、应用程序服务器、操作系统日志、数据库日志等）收集日志，并将这些日志数据统一传送到集中式存储系统（如 Elasticsearch）进行进一步的分析和可视化。

- **应用场景**：
  - 从各类服务器（Web、数据库、应用）中收集日志文件。
  - 使用 Logstash 进行日志解析和标准化，然后将其发送到 Elasticsearch。
  - 用 Kibana 对集中存储的日志数据进行可视化和查询。
- **例如**：
  - 将 Apache 或 Nginx 的访问日志、错误日志收集到一个集中的位置。
  - 将 Java 应用程序或其他应用程序的日志流式处理到 Elasticsearch。

### 2. **实时数据处理与分析**

Logstash 可以实时处理来自不同来源的流数据，并将其转发到目标系统。它能够对数据进行各种处理，如过滤、转换、聚合等，以便更好地进行实时分析。

- **应用场景**：
  - 处理 Web 服务器的实时访问日志，分析用户行为。
  - 处理 IoT 设备的数据流，并实时监控设备健康状况。
  - 收集并分析系统监控数据（如 CPU 使用率、内存使用量、磁盘 I/O 等）进行性能分析。
- **例如**：
  - 从 Kafka 或 RabbitMQ 中消费消息并进行处理，进行实时告警。
  - 从 SNMP 设备收集网络流量数据，实时监控网络性能。

### 3. **数据管道与 ETL（Extract, Transform, Load）**

Logstash 作为一个 ETL 工具，常用于从不同数据源提取数据，进行转换和清洗后，加载到目标存储系统（如数据库、数据湖、搜索引擎等）。它支持对数据进行复杂的转换操作，如格式化、字段提取、字段计算等。

- **应用场景**：
  - 从多种数据源（如数据库、API、CSV 文件、日志文件等）抽取数据。
  - 转换数据格式，例如将 CSV 数据转换为 JSON 格式，或者根据自定义规则提取字段。
  - 数据清洗，例如去除重复记录、过滤无用数据、对数据进行格式标准化。
  - 数据加载到数据仓库或 Elasticsearch 进行存储和分析。
- **例如**：
  - 将 MySQL 数据库中的数据从关系型表转换为 Elasticsearch 中的文档格式，进行搜索和分析。
  - 从 REST API 中获取数据并清洗，然后将其存储到数据湖中。

### 4. **安全信息与事件管理（SIEM）**

Logstash 在 **安全信息与事件管理（SIEM）** 系统中扮演重要角色，收集和分析来自各种网络设备、操作系统、应用程序的安全日志。它可以用于检测潜在的安全威胁，并提供实时告警。

- **应用场景**：
  - 收集和分析防火墙日志、入侵检测系统（IDS）日志、操作系统日志等。
  - 通过对数据的实时处理和规则过滤，发现潜在的安全事件，如 SQL 注入攻击、DDoS 攻击等。
  - 将日志数据发送到 Elasticsearch，并使用 Kibana 进行实时分析和可视化，帮助安全分析人员发现问题。
- **例如**：
  - 从 Web 防火墙、应用程序防火墙（WAF）等收集日志并分析异常行为。
  - 从服务器日志中筛选出特定的安全事件，例如用户登录失败次数过多。

### 5. **数据格式转换**

Logstash 可以用于不同数据格式之间的转换。例如，它可以将非结构化的日志数据转换为结构化的 JSON 格式，便于存储和分析。它还支持对数据进行格式化、字段拆分、合并等操作。

- **应用场景**：
  - 将传统的文本日志（如 Apache 日志、系统日志）转换为 JSON 格式，以便于存储在 Elasticsearch 中。
  - 从 CSV 文件中提取数据，将其转换为 JSON 格式并加载到 Elasticsearch 或数据库。
- **例如**：
  - 将 CSV 数据源转换为 JSON 格式并将其存储到 Elasticsearch 进行快速搜索。
  - 从 SQL 数据库中抽取数据，转换为结构化的 JSON 格式进行进一步分析。

### 6. **多数据源整合**

Logstash 可以集成多个数据源，并将它们统一处理后发送到一个目标系统（如 Elasticsearch、Kafka、Redis 等）。它可以连接各种日志来源，包括文件、数据库、消息队列、REST API 等。

- **应用场景**：
  - 从多台 Web 服务器和数据库收集日志并统一处理。
  - 集成多种监控工具和数据源，将其数据整合到一个统一的数据平台（如 Elasticsearch）中进行分析。
- **例如**：
  - 从 MySQL、PostgreSQL 和 MongoDB 等不同数据库收集日志数据，统一发送到 Elasticsearch 进行分析。
  - 从多个消息队列（如 Kafka 或 RabbitMQ）收集数据，进行处理后存储到数据仓库。

### 7. **告警与事件触发**

Logstash 可以与其他系统集成，触发告警或事件。例如，当处理到某些特定的数据时（如日志中的错误或警告信息），可以触发一个事件或告警，通知相关人员进行处理。

- **应用场景**：
  - 根据某些日志中的错误信息（如应用崩溃、数据库连接失败）触发告警。
  - 基于某些指标（如 CPU 使用率过高）触发自动化运维操作。
- **例如**：
  - 从日志中识别出包含 `ERROR` 或 `WARN` 关键字的日志行，并通过邮件或 Slack 通知管理员。
  - 基于 Web 服务器日志中的访问频率异常，触发警报并启动自动化扩展。

### 总结

**Logstash** 是一个强大的数据收集和处理工具，广泛应用于以下场景：

1. **日志收集与集中管理**：收集来自各类日志源的数据，进行统一存储和分析。
2. **实时数据处理与分析**：处理实时数据流，进行分析和监控。
3. **ETL 数据管道**：用于从不同数据源提取数据，进行清洗和转换后加载到目标存储系统。
4. **SIEM 系统**：安全信息与事件管理，通过日志分析实时检测安全事件。
5. **数据格式转换**：将各种格式的数据（如日志、CSV、数据库）转换为结构化数据。
6. **多数据源整合**：将来自不同来源的数据进行整合和统一处理。
7. **告警与事件触发**：基于日志和数据内容触发告警，进行自动化处理。

这些功能使得 **Logstash** 成为处理和转发大量异构数据的理想工具，特别是在日志收集、数据分析和监控场景中。

# 3.Kibana

> Kibana（可视化分析）

## 1.2安装

### 1.2.1.windows

在 Windows 系统上安装 Kibana 的步骤比较简单，以下是详细的步骤：

### 1. **前提条件**

- **Elasticsearch 已安装**：Kibana 需要与 Elasticsearch 配合工作，因此在安装 Kibana 之前，确保已经安装并启动了 Elasticsearch。
- **Java 环境**：Kibana 不需要单独的 Java 环境，Kibana 会自带其所需的 Java 运行时环境。

### 2. **下载 Kibana**

1. 访问 Kibana 官方下载页面：https://www.elastic.co/downloads/kibana

2. 选择适合你操作系统版本的 Kibana 版本。对于 Windows，下载 `.zip` 格式的文件。

   例如：`kibana-8.6.0-windows-x86_64.zip`

### 3. **解压 Kibana**

1. 下载完毕后，解压下载的 `.zip` 文件到你想安装 Kibana 的目录。例如，解压到 `C:\Program Files\Kibana`。

   ```
   bash
   
   
   复制代码
   C:\Program Files\Kibana
   ```

2. 解压后，你会看到 Kibana 的所有文件，包括 `bin`、`config`、`data` 等目录。

### 4. **配置 Kibana**

1. 找到解压后的 Kibana 目录，进入 `config` 文件夹。

2. 打开 `kibana.yml` 配置文件，使用文本编辑器（如 Notepad++ 或 Visual Studio Code）。

   在文件中，可以配置以下几个重要项：

   - **Elasticsearch 的地址**：默认情况下，Kibana 会连接到 `http://localhost:9200`，即 Elasticsearch 的默认地址。如果你的 Elasticsearch 不在本地，或者使用了其他端口，可以修改 `elasticsearch.hosts`。

     ```yaml
     elasticsearch.hosts: ["http://localhost:9200"]
     ```

   - **Kibana 地址**：默认情况下，Kibana 会绑定在 `localhost` 上的 5601 端口。如果需要修改 Kibana 的地址或端口，可以更改 `server.host` 和 `server.port` 配置项。

     ```yaml
     server.host: "0.0.0.0"  # 使 Kibana 对外可访问
     server.port: 5601       # 默认端口是 5601
     ```

   - **启用 X-Pack（可选）**：如果你使用了 Elastic Stack 的 X-Pack 插件进行身份验证或其他功能，可能需要配置相关的安全设置。

### 5. **启动 Kibana**

1. 打开命令提示符（CMD），以管理员身份运行。

2. 进入到解压的 Kibana 目录中的 `bin` 文件夹。例如：

   ```bash
   cd C:\Program Files\Kibana\bin
   ```

3. 在命令行中输入以下命令来启动 Kibana：

   ```bash
   kibana.bat
   ```

   启动后，Kibana 将会绑定在你在 `kibana.yml` 文件中配置的地址和端口上（默认是 `localhost:5601`）。

### 6. **访问 Kibana Web 界面**

- 启动 Kibana 后，打开浏览器，访问 `http://localhost:5601`，你将看到 Kibana 的 Web 界面。
- 在 Kibana 中，你可以使用图形界面来管理 Elasticsearch 中的数据，创建和管理仪表板等。

### 7. **设置 Kibana 为 Windows 服务（可选）**

如果你希望 Kibana 在 Windows 启动时自动启动并以服务的方式运行，可以通过以下步骤来配置它：

1. 使用 `nssm`（Non-Sucking Service Manager）工具将 Kibana 注册为 Windows 服务。你可以从 [nssm官网](https://nssm.cc/) 下载该工具。

2. 使用 `nssm` 注册 Kibana 服务：

   - 运行 `nssm`，在 **Application** 选项卡中设置程序路径为 `kibana.bat` 文件的路径（例如：`C:\Program Files\Kibana\bin\kibana.bat`）。
   - 设置启动目录为 Kibana 的根目录（例如：`C:\Program Files\Kibana`）。
   - 点击 **Install Service** 按钮完成安装。

3. 启动 Kibana 服务：

   ```
   bash
   
   
   复制代码
   net start kibana
   ```

   这将自动启动 Kibana 服务，并让它在后台运行。

### 8. **验证 Kibana 是否正常工作**

访问 `http://localhost:5601`，确保 Kibana 界面加载正常。如果无法访问，检查以下内容：

- **Elasticsearch 是否运行**：Kibana 依赖于 Elasticsearch，确保 Elasticsearch 服务已启动且可访问。
- **防火墙设置**：确保没有防火墙阻止访问 Kibana 的端口（默认是 5601）。

### 9. **日志查看**

如果 Kibana 启动时遇到问题，可以查看 Kibana 的日志文件。日志文件位于 Kibana 安装目录的 `logs` 文件夹中。查看 `kibana.log` 文件中的错误信息，帮助诊断问题。

# 4.分词器

> lk分词器



- 





