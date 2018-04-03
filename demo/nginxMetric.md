#### 运维监控 -- nginx metric 的监控搭建

nginx 是现代 web 服务栈中最重要的组件之一，本文主要介绍通过 telegraf 和 Pandora TSDB 来收集 nginx 的 metric 数据。


#### 监控内容

本文假设 nginx 的版本为开源版本的 nginx

* accepts
* active
* handled
* reading
* writing
* waiting
* requests

商业版本的 nginx 有更多的 metric 数据

#### 快速开始

1. 确认 nginx stub status module 被打开


```
nginx -V 2>&1  | grep -o with-http_stub_status_module
```

如果这条命令的输出为`with-http_stub_status_module`，那么表示该 nginx 是支持 stub status module 的。

如果这条命名没有任何输出，那么需要重新编译 nginx

```
./configure \
... \
--with-http_stub_status_module
make
sudo make install
```

2. 配置 nginx，确保 stub status 能被访问到

添加配置

```
server {
    location /nginx_status {
        stub_status on;

        access_log off;
        allow 127.0.0.1;
        deny all;
    }
}
```

!> 注意：`server`字段一般不写在 nginx 的主配置文件中（比如：`/etc/nginx/nginx.conf`),而是写在子配置文件中，通过在主配置文件中`include`的方式引入。


```
nginx -t
```
可以找到主配置文件的位置


打开主配置文件，找到类似`include /etc/nginx/conf.d/*.conf;`的字样，然后找到任一配置文件，按照上述 server 的配置进行修改

最后执行

```
nginx -s reload
```

```
$ curl http://localhost/nginx_status
Active connections: 2 
server accepts handled requests
 34 34 741 
Reading: 0 Writing: 1 Waiting: 1
```
看到以上字样说明配置成功。


3. 在七牛应用市场打开 Grafana 应用，然后按照下图所示的配置：


![配置 Grafana 数据源](https://pandora-kibana.qiniu.com/monitor1.gif)

注意事项：

![](https://pandora-kibana.qiniu.com/monitor3.png)

**第五步： 导入 Grafana dashboard 配置文件**

下载 Grafana dashboard 配置文件

```
wget http://orzfblcum.bkt.clouddn.com/NGINX.json
```

将下载的 dashboard 导入 Grafana

![将下载的 dashboard 导入 Grafana](https://pandora-kibana.qiniu.com/monitor2.gif)


完成！