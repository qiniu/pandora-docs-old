#### 运维日志分析 -- nginx 日志分析搭建案例

nginx是现代web服务栈中最重要的组件之一，本文主要介绍通过logkit和Pandora LogDB来收集nginx的access.log并搜索分析。

#### 功能描述

* 海量数据支撑
* 快速接入，无侵入式配置，快速部署使用
* 多种类型分析、分词、加工、变换
* 实时监控，数据可视化（Grafana用户无障碍迁移）
* 离线分析，发现数据更大价值
* 计算结果导出到用户自身，快速回流


#### 监控内容

nginx的访问日志(access.log)

#### 快速开始

#### 数据接入

##### 根据您机器的操作系统版本下载logkit

https://github.com/qiniu/logkit/wiki/Download

解压后您可以看到

```
logkit
logkit.conf
```

其中 `logkit.conf` 为主配置文件，用于配置启动监听的端口，启动后可以通过浏览器访问该端口进行logkit的配置。默认情况下为: "http://127.0.0.1:3000"。

##### 运行logkit

```
nohup ./logkit -f logkit.conf > logkit.log 2>&1 &
```

下面我们将为您介绍如何配置logkit以收集nginx的日志。

##### 访问logkit配置页面

通过浏览器打开logkit的页面，在首页可以看到总体的运行情况。

![图1 logkit首页](http://ou3jgt6kj.bkt.clouddn.com/logkitnginx1.png)

点击 【增加Runner】就可以开始配置nginx的数据收集。

##### 明确要收集的 nginx 日志路径

假设该配置文件路径为： `/home/users/nginx/log.log`

![图2 服务nginx日志样式](http://ou3jgt6kj.bkt.clouddn.com/logitnginx2.png)

选择数据源类型为 "从文件读取(file 模式)"

然后在 “日志文件路径” 处填写您的文件路径。

点击【下一步】，进入到【配置解析方式】部分。

##### 配置解析方式

选择【grok 方式解析】，如图所示

![此处输入图片的描述][1]

我们的grok与Logstash的grok方式完全兼容，如果您熟悉，可以跳过这一节的介绍；如果您不熟悉【grok】，没关系，通过下面的简单介绍您就可以完全掌握。

**使用grok parser解析nginx/apache日志的过程，实际上就是利用grok pattern(正则表达式)去匹配您的nginx日志**，对于像nginx/apache日志这样的成熟日志内容，日志的所有组成部分均已经有非常成熟的grok pattern可以使用，下面我们先介绍下用于解析nginx/apache日志时常用的几个内置在logkit的grok pattern。

##### 常用grok pattern介绍

1. `NOTSPACE` 匹配所有非空格的内容，这个是性能较高且最为常用的一个pattern，比如你的日志内容是`abc efg`，那么你只要写两个`NOTSPACE`的组合Pattern即可，如 `%{NOTSPACE:field1} %{NOTSPACE:field2}`。
1. `QUOTEDSTRING` ， 匹配所有被双引号括起来的字符串内容，跟NOTSAPCE类似，会包含双引号一起解析出来，如`"abc" - "efx sx"` 这样一串日志，写一个组合Pattern`%{QUOTEDSTRING:field1} - %{QUOTEDSTRING:field2}`，field2就包含数据`"efx sx"`，这个同样性能较高，好处是不怕有空格等其他特殊字符，缺点是解析的内容包含了双引号本身，如果要转换成long等类型需要去掉引号。
1. `DATA` 匹配所有字符，这个pattern需要结合一些特殊的语境使用，如双引号等特殊字符。举例来说 `"abc" - "efx sx"`，这样一串日志，可以写一个组合Pattern `"%{DATA:field1}" - "%{DATA:field2}"`，这个就起到了`QUOTEDSTRING`的效果，另外数据中不会包含双引号。
1. `HTTPDATE` 匹配常见的HTTP日期类型，类似nginx和Apache生产的timestamp都可以用这个Pattern解析。如`[30/Sep/2017:10:50:53 +0800]`，就可以写一个组合Pattern `[%{HTTPDATE:ts:date}]`，中括号里面包含 `HTTPDATE` 这个Pattern，就把时间字符串匹配出来了。
1. `NUMBER` 匹配数字类型，包括整数和浮点数，利用这个Pattern就可以把nginx里面的如响应时间这样的数据解析出来。如`"10.10.111.117:8888" [200] "0.002"`，就可以写`"%{NOTSPACE:ip}" [%{NUMBER:status:long}] "%{NUMBER:resptime:float}"` 来解析出status状态码以及resptime响应时间。

基本上，上述这些基础的grok Pattern 组合起来，就可以解决几乎所有nginx的日志解析，但有时候会遇到一些特殊情况，如某个字段可能存在也可能不存在，比如如下两行日志，我们都希望解析。

1. POST中包含HTTP协议信息
```
"POST /resouce/abc HTTP/1.1"
```

2. POST中不包含协议信息
```
"GET /resouce/abc"
```

此时就需要编写一种组合场景，表达`或`的逻辑，此时可以在Pattern组合中融入正则表达式的组概念，如下串即可解析：
```
"(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion}))"
```
其中括号就是正则表达式的组，组里面还可以包含组，每个组通过"?"问号开头表示可以存在0次或1次，":"冒号后表达匹配的内容。

在nginx日志中常常还会出现内容为空的情况，为空时nginx字段填充`-`横杠，此时也可以用类似的方法写`或`。
如这两种数据 `0.123` 以及 `-`，如果把"-"当成正常的数字去解析，就会出错，所以需要去掉没有数字的情况，如：
```
(?:%{NUMBER:bytes}|-)
```

最后，假设我们遇到一种不太规则的nginx日志写法，如：

```
110.220.330.550 - - [12/Oct/2017:14:16:50 +0800] "POST /v2/repos/xsxsxs/data HTTP/1.1" 200 729 2 "-" "Faraday v0.13.1" "-" 127.9.2.1:80 www.qiniu.com xsxsxsxsx122dxsxs 0.019
```

我们就可以用上面描述的方法拼接出如下的串：
```
NGINX_LOG %{NOTSPACE:client_ip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:ts:date}\] "(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:http_version:float})?|%{DATA})" %{NUMBER:resp_code} (?:%{NUMBER:resp_bytes:long}|-) (?:%{NUMBER:resp_body_bytes:long}|-) "(?:%{NOTSPACE:referrer}|-)" %{QUOTEDSTRING:agent} %{QUOTEDSTRING:forward_for} %{NOTSPACE:upstream_addr} (%{HOSTNAME:host}|-) (%{NOTSPACE:reqid}) %{NUMBER:resp_time:float}
```

最后，你可以在logkit的web页面上调试一下，将您的实际日志数据放到【输入样例日志】中查看，点击【解析样例数据】按钮，就可以尝试解析。若无法解析出字段和数据内容，说明您的grok Pattern还需要修正。

![](http://ou3jgt6kj.bkt.clouddn.com/grok_nginx.png)

##### 更方便的调试技巧

1. 访问网址： http://grokdebug.herokuapp.com
2. 如下图所示，填写各类信息：

![](http://op26gaeek.bkt.clouddn.com/grok_debuger.png)

###### 一条示例日志：

```
[04/Jun/2016:12:41:45 +0100] 1.25 200 192.168.1.1 5.432µs 101
```

###### 最终使用的grok pattern

```
%{TEST_LOG_A}
```

###### 自动生成grok pattern

对于一些常见的日志，甚至可以先使用【Discover】模块生成一个基本的grok patter再进行调试修改。

![此处输入图片的描述][2]

最终调试完成您的样例数据后，可以点击【下一步】，进入到【配置发送方式】

##### 配置发送方式

![此处输入图片的描述][3]

在发送方式这边，最基本的只需要填写您的”数据源名称“，”七牛的公钥“，以及”七牛的私钥“ 三项即可。

点击【下一步】，【确认并提交】，至此就完成了您的所有配置。

更多logkit的高级用法，参见 [logkit wiki文档](https://github.com/qiniu/logkit/wiki)


#### 数据加工

##### 登录七牛官方网站，在大数据工作流引擎中即可看到已经创建的数据传输通道，如图4。

![图4 大数据工作流引擎](http://op26gaeek.bkt.clouddn.com/logdbexport.png)

##### 在日志检索界面查询数据，如图5-1所示

![图5-1 日志检索搜索界面示意图](http://op26gaeek.bkt.clouddn.com/logdbsearch.png)

至此，您已经可以通过搜索玩转您本地的数据啦。

除了默认导出一份到日志检索之外，您也可以回到大数据工作流引擎，根据您的需要任意创建针对实时数据流的自定义计算并导出，如图5-2所示。

![图5-2 数据实时计算](http://ou3jgt6kj.bkt.clouddn.com/transform.png)

经过多种实时计算变换的数据，除了导出到Pandora已有的日志检索、时序数据库以及对象存储以外，还可以根据需要，导出到您本地假设的http服务器上，即在Pandora进行数据计算后将结果回流到您的平台落地，如图5-3所示。

![图5-3 数据实时计算](http://ou3jgt6kj.bkt.clouddn.com/transformhttp.png)

当然，导出到对象存储的数据，还可以在工作流引擎中创建**离线计算工作流**，再次进行数据加工聚合计算并导出，如图5-4所示。

![图5-4 离线计算](http://ou3jgt6kj.bkt.clouddn.com/lixian.png)

在离线计算的工作流引擎，你可以根据需要周期性的运行您的计算任务，如定时分析一天的数据、一周的数据，出一份日报、周报等。

#### 实时数据展示与监控

我们提供创建并配置Grafana进行监控。

##### 创建Grafana App，如图6所示

![图6 Grafana APP 创建](http://op26gaeek.bkt.clouddn.com/newbuildGrafana.png)

##### 配置Grafana LogDB 数据源，如图7所示，点击logdb使用指南，可以按照使用指南的指导在Grafana配置数据源。

![图7 Grafana数据源配置](http://op26gaeek.bkt.clouddn.com/logdbGrafana.png)

**注意事项**

- Default Query Settings中， Group by interval 填写时间 `10s`，注意单位为`s`,`m`等，不能漏掉，必须小写。
- Time Field Name 处填写您的 logdb时间字段， 填您 nginx 配置的命名，在上述的截图示例中，是 `time_local` , 没有默认的 `$` 符号
- Index name中，模式固定为 `Daily` , 串固定为 `[reponame-]YYYY.MM.DD` , 将reponame字符串改为您的数据源名称即可。
- Version 固定为 `2.x`

##### 载入现成的Grafana配置

下载json http://op26gaeek.bkt.clouddn.com/logdbgrafana.json

在Grafana界面导入json，并选择数据源。

最终您将看到的效果，如图8-1所示

![图8-1](http://op26gaeek.bkt.clouddn.com/logdbgrafanawhole.png)

仅仅以Nginx日志为例，您可以看到哪些十分有价值的数据呢？

* 实时总用户访问量(请求数统计)，如图8-2所示

![图8-2](http://ou3jgt6kj.bkt.clouddn.com/totalrequest.png)

* 机器请求数随时间变化趋势，如图8-3所示

![图8-3](http://ou3jgt6kj.bkt.clouddn.com/requestnum.png)

* 实时请求状态码占比，如图8-4所示

![图8-4](http://ou3jgt6kj.bkt.clouddn.com/statscode.png)

* 实时请求TOP排名，如图8-5所示

![图8-5](http://ou3jgt6kj.bkt.clouddn.com/toprequest.png)

* 实时请求来源IP TOP排名，如图8-6所示

![图8-6](http://ou3jgt6kj.bkt.clouddn.com/topip.png)

* 响应时间随时间变化趋势图，如图8-7所示

![图8-7](http://ou3jgt6kj.bkt.clouddn.com/responsetime.png)

* 实时用户请求的客户端TOP排名，如图8-8所示

![图8-8](http://ou3jgt6kj.bkt.clouddn.com/topagent.png)

* 实时根据不同情况进行具体数据的查询，包括状态码、响应时间范围进行筛选等，如图8-9所示

![图8-9](http://ou3jgt6kj.bkt.clouddn.com/detail.png)

* 其他更多自定义配置...

自定义的 Grafana DashBoard 配置示例，如图9所示

![图9 Grafana DashBoard配置示例](http://op26gaeek.bkt.clouddn.com/nginxrespcode.png)

在此，您可以通过Grafana，通过您的 Nginx日志 完整而详尽的 了解您业务的流量入口的各类情况。

##### 报警

除此之外，我们还为您创建的Grafana提供了完善多样的报警功能。

首先设置下Grafana报警的Channel，如图10所示。

![图10 Grafana Alert Channel](http://op26gaeek.bkt.clouddn.com/alertchannel.png)

点击 **New Channel** 按钮，您可以在 **Type** 那边选择包括 Slack， Email邮箱，Webhook等十来种报警方式。

设置好报警的Channel以后，回到Dashboard界面，您就可以愉快的设置报警啦。比如说如图11，我们设置了一个响应时间大于1000ms的报警

![图11 响应时间大于1000ms报警](http://op26gaeek.bkt.clouddn.com/alerting.png)

**LogDB采用的是基于Elasticsearch协议的报警，这个Grafana的功能是七牛独家哦！**

那么您可以看到报警形式是怎么样的呢？

![图12 Slack报警示意图](http://op26gaeek.bkt.clouddn.com/slackalerting.png)

看到图12 Slack上的报警了吗？除了基本的文字，还会带上酷炫的报警图片！图片都会被存储到您七牛云存储的bucket(`grafana-alert-images`)里面！

![图13 邮件报警示意图](http://op26gaeek.bkt.clouddn.com/nginxalert.png)

邮件报警内容也一样酷炫！

##### 离线分析

除了实时的分析外，您还可以创建离线的XSpark，分析更多更久的海量数据，详见 [Xspark使用入门](https://qiniu.github.io/pandora-docs/#/quickstart/xspark)。


##### 更多功能，欢迎试用体验！！！


  [1]: http://ou3jgt6kj.bkt.clouddn.com/logkitnginx3.png
  [2]: http://ou3jgt6kj.bkt.clouddn.com/logkitnginx4.png
  [3]: http://ou3jgt6kj.bkt.clouddn.com/logkitnginx5.png