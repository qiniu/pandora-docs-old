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
confs/default.conf
```

其中 `logkit.conf` 为主配置文件，用于配置监听的子配置文件夹，修改主配置文件需要重启logkit。
您需要将其中的 `confs_path` 地址设置要监听的子配置文件夹路径。

`confs` 文件夹就是一个示例的子配置文件夹，子配置文件的更新无需重启logkit，会被logkit实时监听，我们在子配置文件中设置实际要收集的各种配置文件。

下面我们将为您介绍如何配置子配置文件以收集nginx的日志。


##### 明确本机的 nginx 配置文件 log_format 位置 如图1

![图1 Nginx日志格式](http://op26gaeek.bkt.clouddn.com/logformat.png)


假设该配置文件路径为： `/opt/nginx_logs/logs/access.log`

##### 明确服务使用的 nginx 日志样式，如图2

![图2 服务nginx日志样式](http://op26gaeek.bkt.clouddn.com/realnginxconfig.png)

假设我们使用的 nginx 日志样式为 `main`

##### 根据我们明确的nginx配置文件，填写nginx日志收集的logkit配置文件，如图3，填写内容覆盖到 `confs/default.conf` 即可

![图3 nginx runner 配置文件](http://op26gaeek.bkt.clouddn.com/nginx%01config.png)

```
{
    "name":"nginx_runner",
    "reader":{
        "mode":"file",
        "meta_path":"meta",
    	"log_path":"/opt/nginx_logs/logs/access.log"
    },
    "parser":{
        "name":"nginx_parser",
        "type":"nginx",
        "nginx_log_format_path":"/opt/nginx/conf/nginx.conf",
        "nginx_log_format_name":"main",
        "nginx_schema":"time_local date,bytes_sent long,request_time float,body_bytes_sent long",
        "labels":"machine {machineNumber},team {opTeam}"
    },
    "senders":[{
        "name":"pandora_sender",
        "sender_type":"pandora",
        "pandora_ak":"your_ak",
        "pandora_sk":"your_sk",
        "pandora_host":"https://pipeline.qiniu.com",
        "pandora_repo_name":"my_nginx_log",
        "pandora_region":"nb",
        "pandora_schema_free":"true",
        "pandora_gzip": "true",
        "pandora_enable_logdb":"true",
        "fault_tolerant":"true",
        "ft_save_log_path":"./ft_log",
        "ft_strategy":"always_save",
        "ft_procs":"2"
}]
}
```

除了nginx日志，logkit还支持收集其他日志，更多logkit的高级用法，参见 [logkit wiki文档](https://github.com/qiniu/logkit/wiki)

##### 运行logkit

```
nohup ./logkit -f logkit.conf > logkit.log 2>&1 
```

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