### 运维监控 -- 最佳实践

运维监控是大数据应用最为基础的功能之一，Pandora提供的transform和TSDB组合配合开源的Telegraf可以满足运维监控的需求！

#### 典型场景

* 机器的基础信息监控，比如cpu，内存，网络，磁盘，内核等
* 对特定组件的监控，比如redis，MongoDB,mesos,kubernetes,elasticsearch,apache,aws cloudwatch,docker等
* 对自研组建的监控
* 对运维数据进行实时监控，按照特定时间段、特定关键字聚合
* 对异常情况进行报警通知

#### 最佳实践

下面就Pandora提供的组件来搭建一个运维监控应用，搭建这个应用只需要5步（在5分钟之内完成）

> 注意，为了顺利使用Pandora的各种服务，第一，需要一个已经实名认证的七牛账户；第二，申请开通了Pandora的使用权限；

1. 第一步：下载&配置

linux系统

```
wget http://orzfblcum.bkt.clouddn.com/telegraf.linux.amd64.tar.gz
```

mac系统

```
wget http://orzfblcum.bkt.clouddn.com/telegraf.darwin.amd64.tar.gz
```

以linux系统为例，解压

```
tar xvf telegraf.linux.amd64.tar.gz
```

生成配置文件,将其重定向到文件`telegraf.conf`中

```
./telegraf config > telegraf.conf
```

2. 第二步：修改配置

```
 # Configuration for Pipeline server to send metrics to
 [[outputs.pipeline]]
  # Configuration for Pandora Pipeline server to send metrics to
   [[outputs.pipeline]]
   url = "https://pipeline.qiniu.com" # required
   ## The target repo for metrics (telegraf will create it if not exists).
   repo = "monitor" # required

   ## 是否自动创建series
   auto_create_repo = true
   ## Write timeout (for the Pandora client), formatted as a string.
   ## If not provided, will default to 5s. 0s means no timeout (not recommended).
   timeout = "5s"
   ak = "ACCESS_KEY"
   sk = "SECRET_KEY"
```


`repo`: repo的名字,默认为monitor

`ak`: 账户的ak

`sk`: 账户的sk

将内容填写完整的上述配置拷贝到`telegraf.conf`文件，保存文件

3. 第三步：启动&发送数据

用上述生成的配置文件启动Telegraf

```
./telegraf -config telegraf.conf
```

4. 第四步： 配置Grafana数据源

按照下图所示的配置

![配置Grafana数据源](http://oo6e9ks0k.bkt.clouddn.com/QQ20170629-0.png)


5. 第五步： 导入Grafana dashboard配置文件

下载Grafana dashboard配置文件

```
wget http://orzfblcum.bkt.clouddn.com/Main%20Dashboard.json
```

将下载的dashboard导入Grafana

![将下载的dashboard导入Grafana](http://orzfblcum.bkt.clouddn.com/WechatIMG3135.jpeg)


![将下载的dashboard导入Grafana](http://orzfblcum.bkt.clouddn.com/WechatIMG3136.jpeg)


![将下载的dashboard导入Grafana](http://orzfblcum.bkt.clouddn.com/WechatIMG3137.jpeg)

完成！

![最终能看到的效果](http://orzfblcum.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-30%20%E4%B8%8B%E5%8D%8812.07.25.png)