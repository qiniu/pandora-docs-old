> 应用市场上的Grafana、Kibana应用如何开通

应用市场上的应用都需要单独申请开通，请联系我们。

> 应用市场的Kibana很多功能不能用？

应用市场的Kibana是一个精简版的Kibana，主要用于用惯了Kibana的用户可以平滑的使用我们的日志检索产品，后续会开放更多Kibana的功能。如紧急需要某些功能，请联系我们。

> 如何将本地部署的Grafana对接到Pandora

七牛提供应用市场的Grafana来对接Pandora系统，应用市场中的Grafana原生支持Pandora TSDB和Pandora LogDB数据源，使用方便；

除此之外，Pandora提供一个工具用来将本地部署的Grafana对接到Pandora系统，这个工具称为`grafanaProxy`.

其工作原理如下图所示：

![Grafana工作原理图](https://pandora-kibana.qiniu.com/grafanaproxy.jpg)

!> 注意：Grafana连接到GrafanaProxy的数据通路必须是可信的，必须不能走公网，因为Grafana会将鉴权所用的AK,SK通过basic auth的方式发送到GrafanaProxy.

1. 下载grafana proxy

```
$ wget https://pandora-kibana.qiniu.com/GrafanaProxy.2017-09-26-12-26-03.tar.gz

$ tar xvf grafanaProxy.tar.gz 

#可以看到如下文件
x _package/
x _package/pandora-grafanaProxy
x _package/README
```


2. 配置grafana proxy

grafana proxy的配置文件是JSON文件，下面是一个样例配置`default.conf`

```
{
	"tsdbHost"  : "https://tsdb.qiniu.com", # TSDB的host地址
	"port" : 8086,                          # grafana proxy的端口，在grafana上要填写的端口
	"response_timeout": 60,                 # 请求的超时时间
	"cross_domain" : true,                  # 必须为true
	"debug_level" : 1,                      # debug的level
	"logdbHost" : "https://logdb.qiniu.com" # LogDB的host地址
}
```

3. 启动

```
$ cd _package
$ ./pandora-grafanaProxy -f default.conf
```

4. 配置Grafana

![Grafana配置](https://oiw6da4op.qnssl.com/grafana/QQ20170308-4@2x.png)

对于图中的`1`，如果Grafana和proxy是在同一个机器上面，那么就填写`localhost`；
如果不在同一个机器上面，那么就将`localhost`改成其他机器的地址.
