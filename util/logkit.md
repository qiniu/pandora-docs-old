
# Logkit [![Build Status](https://api.travis-ci.org/qiniu/logkit.svg)](http://travis-ci.org/qiniu/logkit)

#### 简介

Logkit是[七牛Pandora](https://pandora-docs.qiniu.com)开发的一个通用的数据采集工具，可以将不同数据源的数据方便的发送到[Pandora](https://pandora-docs.qiniu.com)进行数据分析，除了基本的数据发送功能，Logkit还有容错、并发、监控、删除等功能。

#### 支持的数据源

1. 文件(包括csv格式的文件，kafka-rest日志文件，nginx日志文件等,并支持以[grok](https://www.elastic.co/blog/do-you-grok-grok)的方式解析日志)
1. MySQL
1. Microsoft SQL Server(MSSQL)
1. ElasticSearch
1. MongoDB

#### 工作方式

Logkit本身支持多种数据源，并且可以同时发送多个数据源的数据到Pandora，每个数据源对应一个逻辑上的runner，一个runner负责一个数据源的数据推送，工作原理如下图所示

![Logkit 工作原理图](https://qiniu.github.io/pandora-docs/_media/logkit.png)

#### 下载

目前Logkit支持Linux、Windows、MacOS三种系统版本；

请移步至[Download页面](https://github.com/qiniu/Logkit/wiki/Download)

#### 使用教程

请查看[Logkit详细使用文档](https://github.com/qiniu/logkit)