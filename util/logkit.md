
### Logkit

#### 简介

Logkit是[七牛Pandora](https://pandora-docs.qiniu.com)开发的一个通用的数据采集工具，可以将不同数据源的数据方便的发送到[Pandora](https://pandora-docs.qiniu.com)进行数据分析，除了基本的数据发送功能，Logkit还有容错、并发、监控、删除等功能。

#### 支持的数据源

1. 文件(包括csv格式的文件，nginx日志文件等,并支持以[grok](https://www.elastic.co/blog/do-you-grok-grok)的方式解析日志)
1. MySQL
1. Microsoft SQL Server(MSSQL)
1. ElasticSearch
1. MongoDB
1. Kafka
1. Redis

#### 工作方式

Logkit本身支持多种数据源，并且可以同时发送多个数据源的数据到Pandora，每个数据源对应一个逻辑上的runner，一个runner负责一个数据源的数据推送，工作原理如下图所示

![Logkit 工作原理图](https://qiniu.github.io/pandora-docs/_media/logkit.png)

#### 下载

目前Logkit支持Linux、Windows、MacOS三种系统版本；

请移步至[Download页面](https://github.com/qiniu/Logkit/wiki/Download)

#### 使用教程

请查看[Logkit详细使用文档](https://github.com/qiniu/logkit)


### Telegraf

[Telegraf](https://github.com/influxdata/telegraf)是开源的metric收集工具,支持多种数据源的metric收集，[支持的数据源列表列表](https://github.com/influxdata/telegraf#input-plugins).

在该工具以及支持的output的基础上，我们给Telegraf增加来输出到Pandora Pipeline的output，使数据可以方便的收集到Pandora Pipeline.

#### 下载&配置

linux系统

```
wget http://orzfblcum.bkt.clouddn.com/telegraf.linux.amd64.tar.gz
```

mac系统

```
wget http://orzfblcum.bkt.clouddn.com/telegraf.darwin.amd64.tar.gz
```

生成配置文件

```
./telegraf config > telegraf.conf
```

以上命令生成一个telegraf的配置文件`telegraf.conf`

修改配置

```
# # Configuration for Pipeline server to send metrics to
# [[outputs.pipeline]]
#  # Configuration for Pandora Pipeline server to send metrics to
#   [[outputs.pipeline]]
#   url = "https://pipeline.qiniu.com" # required
#   ## The target repo for metrics (telegraf will create it if not exists).
#   repo = "monitor" # required
#
#   ## 是否自动创建series
#   auto_create_repo = false
#   ## Write timeout (for the Pandora client), formatted as a string.
#   ## If not provided, will default to 5s. 0s means no timeout (not recommended).
#   timeout = "5s"
#   ak = "ACCESS_KEY"
#   sk = "SECRET_KEY"
```

去掉行首的注释

`repo`: repo的名字

`auto_create_repo`: 是否自动创建repo以及更新schema

`ak`: 账户的ak

`sk`: 账户的sk

`timeout`: 发送数据的超时时间


#### 启动&发送数据

用上述生成的配置文件启动Telegraf

```
./telegraf -config telegraf.conf
```

配置文件中默认收集的信息有cpu,diskio,mem,kernel等基础数据，需要更多数据请编辑`telegraf.conf`文件.

#### 在Grafana中配置数据源和监控图

[参照Grafana配置部分的内容](https://qiniu.github.io/pandora-docs/#/quickstart/grafana)