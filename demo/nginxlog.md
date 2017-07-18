#### 运维日志分析 -- nginx 日志分析搭建案例

nginx是现代web服务栈中最重要的组件之一，本文主要介绍通过logkit和Pandora LogDB来收集nginx的access.log并搜索分析。


#### 监控内容

nginx的访问日志(access.log)

#### 快速开始

1. 根据您机器的操作系统版本下载logkit

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


2. 明确本机的 nginx 配置文件 log_format 位置 如图1

![图1 Nginx日志格式](http://op26gaeek.bkt.clouddn.com/logformat.png)


假设该配置文件路径为： `/opt/nginx_logs/logs/access.log`

3. 明确服务使用的 nginx 日志样式，如图2

![图2 服务nginx日志样式](http://op26gaeek.bkt.clouddn.com/realnginxconfig.png)

假设我们使用的 nginx 日志样式为 `main`

4. 根据我们明确的nginx配置文件，填写nginx日志收集的logkit配置文件，如图3，填写内容覆盖到 `confs/default.conf` 即可

[图3 nginx runner 配置文件](http://op26gaeek.bkt.clouddn.com/nginx%01config.png)

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
        "nginx_schema":"time_local date,bytes_sent long,request_time float,body_bytes_sent long"
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

5. 运行logkit

```
nohup ./logkit -f logkit.conf > logkit.log 2>&1 
```

6. 登录七牛官方网站，在大数据工作流引擎中即可看到已经创建的数据传输通道，如图4。

![图4 大数据工作流引擎](http://op26gaeek.bkt.clouddn.com/logdbexport.png)

7. 在日志检索界面查询数据，如图5所示

![图5 日志检索搜索界面示意图](http://op26gaeek.bkt.clouddn.com/logdbsearch.png)

至此，一个基本的日志搜索功能便完成了。


#### 进阶功能：配置Grafana进行监控

1. 创建Grafana App，如图6所示

![图6 Grafana APP 创建](http://op26gaeek.bkt.clouddn.com/newbuildGrafana.png)

2. 配置Grafana LogDB 数据源，如图7所示，点击logdb使用指南，可以按照使用指南的指导在Grafana配置数据源。

![图7 Grafana数据源配置](http://op26gaeek.bkt.clouddn.com/logdbGrafana.png) 

**注意事项**

- Default Query Settings中， Group by interval 填写时间 `10s`，注意单位为`s`,`m`等，不能漏掉，必须小写。
- Time Field Name 处填写您的 logdb时间字段， 填您 nginx 配置的命名，在上述的截图示例中，是 `time_local` , 没有默认的 `$` 符号
- Index name中，模式固定为 `Daily` , 串固定为 `[reponame-]YYYY.MM.DD` , 将reponame字符串改为您的数据源名称即可。
- Version 固定为 `2.x`


3. 载入现成的Grafana配置

下载json http://op26gaeek.bkt.clouddn.com/logdbgrafana.json

在Grafana界面导入json，并选择数据源。

最终您将看到的效果，如图8所示

![图8](http://op26gaeek.bkt.clouddn.com/logdbgrafanawhole.png)


4. 自定义的 Grafana DashBoard配置示例，如图9所示

![图9 Grafana DashBoard配置示例](http://op26gaeek.bkt.clouddn.com/nginxrespcode.png)

至此，一个基于Grafana的 nginx 日志分析已经完成了。

欢迎享用并体验！


除了nginx日志，logkit还支持收集其他日志，更多logkit的高级用法，参见 [logkit wiki文档](https://github.com/qiniu/logkit/wiki)