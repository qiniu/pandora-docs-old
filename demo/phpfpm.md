#### 运维监控 -- 快速搭建一个服务器性能监控系统

本文主要介绍通过telegraf和Pandora TSDB来收集phpfpm的metric数据.

#### 监控内容

服务器基础性能信息：

* accepted connections
* active processes
* idle processes
* total processes
* max active processes
* listen queue
* listen queue length
* max listen queue
* slow requests
* max children reached

#### 效果图

![最终能看到的效果](http://orzfblcum.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-14%20%E4%B8%8B%E5%8D%886.38.18.png)

#### 快速开始

**第一步：确保编译的php-fpm已经包含**

找到php-fpm的配置文件

```
;pm.status_path = /status
```

去掉注释，改为

```
pm.status_path = /status
```

**第二步：修改nginx配置**

新增加一个nginx `location`

```
location ~ ^/(status|ping)$ {
    access_log off;
    allow 127.0.0.1;
    deny all;
    include fastcgi_params;
    fastcgi_pass 127.0.0.1:9000;
}

```

`allow 127.0.0.1;`: 改为你的ip地址(如果是和telegraf同一台机器，则不用修改)
`fastcgi_pass 127.0.0.1:9000;`: 此处为php-fpm暴露的端口


**第三步：修改telegraf配置**

将下列的配置信息复制到`telegraf.conf`文件的最顶端，然后保存`telegraf.conf`文件；

```
# # Read metrics of phpfpm, via HTTP status page or socket
 [[inputs.phpfpm]]

   urls = ["http://localhost/status"]

```


**第四步：启动&发送数据**

用上述生成的配置文件启动Telegraf，输入以下命令：

```
./telegraf -config telegraf.conf
```

**第五步： 配置Grafana数据源**

在七牛应用市场打开Grafana应用，然后按照下图所示的配置：

![配置Grafana数据源](_media/monitor1.gif)

注意事项：

![](_media/monitor3.png)

**第五步： 导入Grafana dashboard配置文件**

下载Grafana dashboard配置文件

```
wget http://orzfblcum.bkt.clouddn.com/phpfpm.json
```

完成！


#### 高级用法

* [告警配置方法](https://qiniu.github.io/pandora-docs/#/quickstart/grafana?id=报警使用方法)
* [自研组件监控](https://qiniu.github.io/pandora-docs/#/demo/customMonitor)
* [配置nginxMetric监控](https://qiniu.github.io/pandora-docs/#/demo/nginxMetric)