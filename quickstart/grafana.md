#### Grafana是什么
[Grafana](https://oiw6da4op.qnssl.com/181919_0KHT_865233.jpg)
是开源的用来实时展示、分析和报警的软件，在七牛的应用中心提供Grafana应用，Pandora 时序数据库、日志检索服务都适配了Grafana，可以用Grafana实时展示、分析时序数据库和日志检索服务中的数据，并报警。

#### Grafana有什么特点

> 可视化

> 报警

> 动态面板展示

> 混合数据源展示

> 多种报警通知方式

更多特定可以通过[Grafana提供的demo](http://play.grafana.org/)发现更多用法。

#### TSDB+Grafana快速上手指南
七牛应用中心的Grafana原生支持Pandora TSDB。

##### 创建Grafana应用

1. 首先登陆七牛portal的应用平台 找到Grafana
![grafana](https://oiw6da4op.qnssl.com/grafana/img1.png)

2. 点击立即部署按钮开始创建Grafana应用
![grafana](https://oiw6da4op.qnssl.com/grafana/img2.png)

输入您的app名和应用别名，选择部署区域(注意！目前TSDB的数据在`华东`，所以Grafana只可部署在华东区域)，点击确定创建

应用名称：账号内唯一应用名称，且只能满足以下条件：（1. 只能包含字母、数字和减号，首尾字符只能为字母或数字。 2. 字符长度不能超过 30）
应用别名：供显示使用的标题名。

3. 等待app启动后，输入密码（密码长度必须>=6），点击 确认配置

> 注意，因为Grafana App具有公网域名，所以建议设置一个高强度的密码（此密码在进入Grafana App后可以修改）。

![Grafana](https://oiw6da4op.qnssl.com/grafana/img3.png)

4. 访问Grafana进入Grafana页面

> 注意，该Grafana App是暴露在公网上的，可收藏地址用于后续访问。

![Grafana](https://oiw6da4op.qnssl.com/grafana/img5.png)

##### Pandora TSDB + Grafana使用方法

在Grafana中使用Pandora TSDB之前，我们需要先添加数据源。


1. 登录grafana，点击菜单中的 Data Sources
![点击菜单中的 Data Sources](https://oiw6da4op.qnssl.com/grafana/QQ20170308-1@2x.png)

2. 点击 Add data source 按钮
![点击 Add data source 按钮](https://oiw6da4op.qnssl.com/grafana/QQ20170308-0@2x.png)

3. 在Name填入数据源的名字, Type 选择 Pandora TSDB
![在Name填入数据源的名字, Type 选择 Pandora TSDB](https://oiw6da4op.qnssl.com/grafana/QQ20170308-2@2x.png)

4. 填入相应参数，点击添加按钮即可
> 注意： url 必须填入 http://localhost:8999
![填入相应参数，点击添加按钮即可](https://oiw6da4op.qnssl.com/grafana/QQ20170308-4@2x.png)

其中，repo名可以在七牛portal的时序数据库页面中找到

![repo名可以在七牛portal的时序数据库页面中找到](https://oiw6da4op.qnssl.com/grafana/DD847726796988BEDEDEDB26809B2D1C.jpg)

5. 添加完数据源后，在菜单中选择 dashbords -> new 就可以新建自己的dashbords来展示数据了

![新建dashboard](https://oiw6da4op.qnssl.com/grafana/QQ20170308-5@2x.png)

6. 选择graph可以创建新的图表

![新建图表](https://oiw6da4op.qnssl.com/grafana/QQ20170308-6@2x.png)

7. 点击图标标题，然后点击弹出菜单的Edit即可编辑图标

![编辑图标](https://oiw6da4op.qnssl.com/grafana/QQ20170308-7@2x.png)

8. 编辑界面中的 datasource 选择刚才添加的datasource，即可实时预览和智能提示

![编辑界面中的 datasource 选择刚才添加的datasource，即可实时预览和智能提示](https://oiw6da4op.qnssl.com/grafana/QQ20170308-8@2x.png)

9. 编辑完成后点击 保存 按钮保存新添加的模板
![保存模板](https://oiw6da4op.qnssl.com/grafana/QQ20170308-9@2x.png)

##### 报警使用方法

1. 定义通知方式

![定义通知方式](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%885.58.37.png)

Alert list用来显示最近出现的报警；

![Alert list用来显示最近出现的报警](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%885.58.48.png)

Notification用来定义通知方式，比方说`邮件`,`slack`等；

![Notification用来定义通知方式](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%885.59.23.png)

2. 设定阈值

打开报警设置页面

![打开报警设置页面](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%886.04.54.png)

选择起止时间，序列，阈值

![选择起止时间，序列，阈值](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%886.05.39.png)

测试设定的阈值

![测试设定的阈值](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%886.05.53.png)

查看报警的历史记录

![查看报警的历史记录](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%886.06.30.png)

3. 删除报警规则

删除报警规则

![删除报警规则](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%886.06.41.png)

#### Grafana常见问题

##### 点击访问后，看不到Grafana

弹出窗口可能会被浏览器拦截，点击允许弹出即可

##### 看不到图
看不到图的问题多种多样，建议按照下面顺序排查
1. 在图中所选的时间段内，是否有数据打进TSDB

2. 数据量是否很少导致点无法连成线

![数据量是否很少导致点无法连成线](http://oo6e9ks0k.bkt.clouddn.com/1327FAE695E586CE266566B445C0391B.jpg)
尝试将上图中的选项都选上，以排除是由于点太少造成的图无法显示。

3. 图表类型太少，没有需要的图表类型

七牛应用市场的图表类型目前有，折线图，柱状图，堆叠图，饼状图，如果有需要其他模板的，请联系我们。

##### 在Grafana中有没有中国地图

Grafana官方的市场里只有一个[世界地图](https://grafana.com/plugins/grafana-worldmap-panel),无法展示具体省份

七牛容器应用市场提供了的Grafana内置了全国地图，名字叫[pili map panel](https://github.com/pre-dem/dem-grafana/tree/master/src/github.com/grafana/grafana/data/plugins/pili-map-panel),是七牛PiLi团队开发的一个开源的中国地图。

注意该中国地图的精度只到省份，无法显示市区县的数据，并且只支持Pandora TSDB这种数据源(Data Source)

下面是使用该中国地图的方法:

1. 新建一个pili map panel的图表

![新建一个pili map panel的图表](http://oo6e9ks0k.bkt.clouddn.com/QQ20170619-0.png)

2. 点击panel title，开始编辑图表

![点击panel title，开始编辑图表](http://oo6e9ks0k.bkt.clouddn.com/QQ20170619-1.png)

3. 配置图表

![配置图表](http://oo6e9ks0k.bkt.clouddn.com/QQ20170619-2.png)

选择数据源(目前只支持Pandora TSDB类型的数据源)

选择series

选择字段和聚合函数

去掉group by time(此处必须去掉，否则图出不来)

增加对province的group by(此处要求数据字段中必须有一个tag key为province)

![增加对province的group by](http://oo6e9ks0k.bkt.clouddn.com/QQ20170619-3.png)


### 使用Telegraf

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