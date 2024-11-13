# 1.ElasticSearch

> Elasticsearch（日志管理）

## 1.1.应用场景

> 近乎实时、检索数据；扩展性好，处理PB级别的数据

1. 
   全文搜索
2. 日志分析
3. 运维监控

# 2.Logstash

> Logstash（数据搜索）
>
> Logstash事件处理有三个阶段：inputs → filters → outputs。是一个接收，处理，转发日志的工具。支持系统日志，webserver日志，错误日志，应用日志，总之包括所有可以抛出来的日志类型。

1. 常见的输入行为
   1. file：从文件系统中读取，类似tail -f命令
   2. syslog：在514端口上监听系统日志消息，并根据RFC3164标准进行解析（VPN远程读取日志）
   3. redis：从redis service中读取
   4. beats：从filebeat中读取
   5. Filters：数据中间处理，对数据进行操作
2. 常见的过滤器行为
   1. grok：解析任意文本数据，Grok 是 Logstash 最重要的插件。它的主要作用就是将文本格式的字符串，转换成为具体的结构化的数据，配合正则表达式使用。内置120多个解析语法。
   2. mutate：对字段进行转换。例如对字段进行删除、替换、修改、重命名等。
   3. drop：丢弃一部分events不进行处理。
   4. clone：拷贝 event，这个过程中也可以添加或移除字段。
   5. geoip：添加地理信息(为前台kibana图形化展示使用)
   6. Outputs：outputs是logstash处理管道的最末端组件。一个event可以在处理过程中经过多重输出，但是一旦所有的outputs都执行结束，这个event也就完成生命周期。
3. 一些常见的outputs为：
   1. elasticsearch：可以高效的保存数据，并且能够方便和简单的进行查询。
   2. file：将event数据保存到文件中。
   3. graphite：将event数据发送到图形化组件中，一个很流行的开源存储图形化展示的组件。
   4. Codecs：codecs 是基于数据流的过滤器，它可以作为input，output的一部分配置。Codecs可以帮助你轻松的分割发送过来已经被序列化的数据。
4. 一些常见的codecs：
   1. json：使用json格式对数据进行编码/解码。
   2. multiline：将汇多个事件中数据汇总为一个单一的行。比如：java异常信息和堆栈信息。



# 3.Kibana

> Kibana（可视化分析）



