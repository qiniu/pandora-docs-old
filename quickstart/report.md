### 简介
Pandora BI Studio 是七牛Pandora报表平台, 是Pandora产品体系中负责对业务数据进行展示的平台；

特性：

* 从Pandora实时工作流和离线工作流导入数据
* 支持多达30多种图表类型
* 支持在线对报表数据进行探索、查询
* 支持分享单个图表
* 以下是报表系统支持的图表类型

![](http://pandora-kibana.qiniu.com/report-figure-type2.png)
![](http://pandora-kibana.qiniu.com/report-figure-type3.png)
![](http://pandora-kibana.qiniu.com/report-figure-type4.png)
![](http://pandora-kibana.qiniu.com/report-figure-type5.png)

### 激活报表系统

1. 首先登陆七牛portal，创建实时工作流或者离线工作流。
2. 在创建工作流数据源之后，创建导出任务到报表。
3. 打开`https://bi-studio.qiniu.com`登录报表系统，即可看到创建的数据库和数据表。

![](http://pandora-kibana.qiniu.com/report-login.png)

4. 依照以下操作指南进行操作。


### 报表操作指南

Pandora Report 主要分为四部分模块：

![](http://pandora-kibana.qiniu.com/layout.png)


1. 数据源管理/数据集

这部分用来对数据库和数据表进行管理；工作流中创建的数据库和数据表会自动出现在这里，在这两个页面也可以删除数据库和数据表。


2. 报表制作

点击数据表名称会跳出报表制作页面，在报表制作页面进行不同类型报表的制作；

![](http://pandora-kibana.qiniu.com/figureconfig.png)


3. 报表、看板展现

这部分用来管理报表和看板，看板是报表的一个业务集合；这部分对报表进行管理和组织

![](http://pandora-kibana.qiniu.com/layout.png)


4. SQL编辑器

这部分用来对数据表中的数据进行分析，该页面提供了一个输入SQL语句的输入框，在输入框中输入SQL即可实现对数据的预览和分析。

![](http://pandora-kibana.qiniu.com/sqledit.png)


### 数据对应关系

报表的数据是从离线工作流或者实时工作流导出的，在导出的时候，工作流中的数据类型和报表中的数据类型有一个默认的对应关系；

他们的对应关系如下（workflow的数据类型 -> 报表系统的数据类型)

* string -> VARCHAR(1024)
* int -> INT 
* long -> BIGINT 
* float -> DOUBLE 
* date -> DATETIME

注意，在创建导出到报表系统任务的时候，这种数据类型的映射是自动的，不需要手动选择，并且目前报表系统只支持工作流中这五种基本的数据类型；

创建一个包含全部数据类型的数据表对应的MYSQL的语句应该是这样的

```
CREATE TABLE example_table(name VARCHAR(1024), age INT, height BIGINT, salary DOUBLE, birthday DATETIME)
```

### 制作报表

报表制作区分为两部分，左边为图表参数区，右边为预览区。

![](http://pandora-kibana.qiniu.com/figureconfig.png)


图表参数区主要有

1. 数据和图表类型；

用以进行数据表的选择和图表类型的选择

2. 时间

主要用于指定时间字段，以及起止时间、时间查询的粒度

3. 查询

主要用于指定sql语句中需要查询的字段，指定group by的字段，设置limit值等

4. 图表类型

用于对要生成的报表类型进行定制化的配置，比如配色，图例等

5. SQL

用于指定特定的WHERE和HAVING语句

6. 筛选

用于和其他图表进行联动的时候指定特定的字段的值

预览区主要用于报表的预览


右上角可以生成图表的永久链接、导出数据为csv等


### 图表类型介绍

#### 饼状图

在用户第一次激活报表系统的时候，系统会提示是否导入一个名为`example_database`的样例数据库，这些数据表用来帮助用户理解和学习报表的使用方法，该数据库包含若干个数据表，其中一个数据表名为`birth_names`。

```
+-------------------------+
| Tables_in_example       |
+-------------------------+
| birth_names             |
+-------------------------+

```

birth_names数据表的描述如下：


```
~ $ mysql > describe birth_names

+-----------+--------------+------+-----+---------+-------+
| Field     | Type         | Null | Key | Default | Extra |
+-----------+--------------+------+-----+---------+-------+
| ds        | datetime     | YES  |     | NULL    |       |
| gender    | varchar(16)  | YES  |     | NULL    |       |
| name      | varchar(255) | YES  |     | NULL    |       |
| num       | bigint(20)   | YES  |     | NULL    |       |
| state     | varchar(10)  | YES  |     | NULL    |       |
| sum_boys  | bigint(20)   | YES  |     | NULL    |       |
| sum_girls | bigint(20)   | YES  |     | NULL    |       |
+-----------+--------------+------+-----+---------+-------+

```


该样例数据表描述了从1965年-2008年的美国人口的变动。

* ds： 时间字段
* name： 名字字段
* gender： 性别字段
* num： 叫这个名字的人口总数
* sum_boys: 叫这个名字的男孩总数
* sum_girls: 叫这个名字的女孩总数
* state: 州名

我们使用这个表，新建一个饼状图，饼状图的限制条件如下

| 时间字段 | group by | 指标 | 
| ------- | -------- | --- | 
| 不是必须 | 需要且仅需要一个 | 需要仅需要一个 | 

不同性别人口的饼状图如下所示

![](http://pandora-kibana.qiniu.com/pie2.png)


不同州的人口的饼状图如下所示

![](http://pandora-kibana.qiniu.com/pie.png)


#### 分布-柱状图

| 时间字段 | 指标 | 项目 | 拆分 | 
| ------- | -------- | --- | --- |
| 不是必须 | 一个或者多个 | 需要仅需要一个 | 需要仅需要一个 | 

指标表示需要显示的不同颜色的柱状图的数量，项目表示X轴，拆分表示在具体的一个项目里面要进行哪些分类

下面使用两个分布-柱状图来表示项目和拆分的区别

第一种 项目是 gender，拆分是 state，出来的图为每个gender在不同state的分布图

![](http://pandora-kibana.qiniu.com/fenbu-bar.png)

第二种 项目是 state， 拆分是 gender， 出来的图为每个state中不同gender的分布图

![](http://pandora-kibana.qiniu.com/fenbu-bar2.png)


#### 词汇云

[词汇云(https://zh.wikipedia.org/wiki/%E6%A0%87%E7%AD%BE%E4%BA%91)又称标签云，是按照词语出现次数的多少来表示重要程度的一种图表类型。

所需要的字段如下

| 时间字段 | 指标 | group by | 
| ------- | -------- | --- | 
| 不是必须 | 一个或者多个 | 需要仅需要一个 | 

![](http://pandora-kibana.qiniu.com/word-cloud.png)




#### 表视图

表视图是正常的数据表，主要的配置项有时间，group by和not group by，其中时间是可选配置，group by和not group by只能二选一，其中group by是聚合查询，not group by是原始查询。

| 时间字段 | 指标 | group by | 
| ------- | -------- | --- | 
| 不是必须 | 一个或者多个 | 一个或者多个 | 

![](http://pandora-kibana.qiniu.com/table.png)



#### 时间序列图

bi studio中总共4中时间序列图，分别是 时间序列-折线图，时间序列-柱状图，时间序列-百分比变化图和时间序列-堆积图，这四种时间序列图只在图表的表现形式上有所区别，配置上的主要区别也只是不同图表类型的的X轴和Y轴的配置不同，其查询语句是相同的，此处仅以时间序列-折线图为例来说明时间序列图。

| 时间字段 | 指标 | group by | 
| ------- | -------- | --- | 
| 必须有 | 一个或者多个 | 一个或者多个 | 

![](http://pandora-kibana.qiniu.com/ts-percent.png)


![](http://pandora-kibana.qiniu.com/ts-bar.png)

![](http://pandora-kibana.qiniu.com/ts-line.png)




#### 气泡图

[气泡图](https://baike.baidu.com/item/%E6%B0%94%E6%B3%A1%E5%9B%BE)和散点图相似，比散点图不同的是散点图只标示位置，而气泡图多了一个维度用来记录散点的大小，一个样例气泡图的元数据为

| X轴 | Y轴 | 气泡大小 | 
| ------- | -------- | --- | 
| 0 | 0 | 10 | 
| 1 | 1 | 20 | 
| 2 | 2 | 30 | 

这样一张数据表绘制出来的如下的气泡图



| 时间字段 | 指标 | group by | 
| ------- | -------- | --- | 
| 不是必须 | 一个或者多个 | 一个或者多个 | 


#### 子弹图

[子弹图](https://baike.baidu.com/item/%E5%AD%90%E5%BC%B9%E5%9B%BE)是为了取代仪表盘上常见的那种里程表，时速表等基于圆形信息的表达方式，子弹图无修饰的线性表达方式使我们能够在狭小的空间中表达丰富的数据信息。

| 时间字段 | 度量 |
| ------- | -------- | 
| 不是必须 | 需要且仅需要一个 | 


#### 直方图

[直方图](https://zh.wikipedia.org/wiki/%E7%9B%B4%E6%96%B9%E5%9B%BE)是一种对数据分布情况的图形表示，是一种二维统计图表，它的两个坐标分别是统计样本和该样本对应的某个属性的度量。

bi studio中可以很方便的设置直方图

| 时间字段 | 数字列 |
| ------- | -------- | 
| 不是必须 | 需要且仅需要一个 | 


#### 数字图 & 带趋势的数字图

数字图是一种简单图表，只显示单个数字；带趋势的数字图在数据图的基础上多了一条随时间变化的趋势线。

数字图： 

| 时间字段 | 度量 |
| ------- | -------- | 
| 不是必须 | 需要且仅需要一个 | 

带趋势的数字图：

| 时间字段 | 数字列 |
| ------- | -------- | 
| 必须 | 需要且仅需要一个 | 

下面是出生人口图

![](http://pandora-kibana.qiniu.com/number.png)


![](http://pandora-kibana.qiniu.com/num-trendline.png)




#### 树状图


2. birth_france_by_region

```
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| DEPT_ID | varchar(10) | YES  |     | NULL    |       |
| 2003    | bigint(20)  | YES  |     | NULL    |       |
| 2004    | bigint(20)  | YES  |     | NULL    |       |
| 2005    | bigint(20)  | YES  |     | NULL    |       |
| 2006    | bigint(20)  | YES  |     | NULL    |       |
| 2007    | bigint(20)  | YES  |     | NULL    |       |
| 2008    | bigint(20)  | YES  |     | NULL    |       |
| 2009    | bigint(20)  | YES  |     | NULL    |       |
| 2010    | bigint(20)  | YES  |     | NULL    |       |
| 2011    | bigint(20)  | YES  |     | NULL    |       |
| 2012    | bigint(20)  | YES  |     | NULL    |       |
| 2013    | bigint(20)  | YES  |     | NULL    |       |
| 2014    | bigint(20)  | YES  |     | NULL    |       |
| date    | date        | YES  |     | NULL    |       |
+---------+-------------+------+-----+---------+-------+

```

该数据表描述了从2003-2014年法国不同地区的出生人口数量的变化。

* DEPT_ID： 地区id
* date： 时间字段

3. energy_usage

```
+--------+--------------+------+-----+---------+-------+
| Field  | Type         | Null | Key | Default | Extra |
+--------+--------------+------+-----+---------+-------+
| source | varchar(255) | YES  |     | NULL    |       |
| target | varchar(255) | YES  |     | NULL    |       |
| value  | float        | YES  |     | NULL    |       |
+--------+--------------+------+-----+---------+-------+
```

* source: 源字段
* target： 目的字段
* value: 源到目的地的权值


4. long_lat

```
+--------------+------------+------+-----+---------+-------+
| Field        | Type       | Null | Key | Default | Extra |
+--------------+------------+------+-----+---------+-------+
| LON          | double     | YES  |     | NULL    |       |
| LAT          | double     | YES  |     | NULL    |       |
| NUMBER       | text       | YES  |     | NULL    |       |
| STREET       | text       | YES  |     | NULL    |       |
| UNIT         | text       | YES  |     | NULL    |       |
| CITY         | double     | YES  |     | NULL    |       |
| DISTRICT     | double     | YES  |     | NULL    |       |
| REGION       | double     | YES  |     | NULL    |       |
| POSTCODE     | bigint(20) | YES  |     | NULL    |       |
| ID           | double     | YES  |     | NULL    |       |
| date         | date       | YES  |     | NULL    |       |
| occupancy    | float      | YES  |     | NULL    |       |
| radius_miles | float      | YES  |     | NULL    |       |
+--------------+------------+------+-----+---------+-------+

```

该表用来根据不同的经纬度在地图上描绘不同权值的分布图。


5. multiformat_time_series

```
+----------+--------------+------+-----+---------+-------+
| Field    | Type         | Null | Key | Default | Extra |
+----------+--------------+------+-----+---------+-------+
| ds       | date         | YES  |     | NULL    |       |
| ds2      | datetime     | YES  |     | NULL    |       |
| epoch_ms | bigint(20)   | YES  |     | NULL    |       |
| epoch_s  | bigint(20)   | YES  |     | NULL    |       |
| string0  | varchar(100) | YES  |     | NULL    |       |
| string1  | varchar(100) | YES  |     | NULL    |       |
| string2  | varchar(100) | YES  |     | NULL    |       |
| string3  | varchar(100) | YES  |     | NULL    |       |
+----------+--------------+------+-----+---------+-------+
```

该表显示了不同的时间的表达形式，样例数据如下

```
+------------+---------------------+---------------+------------+----------------------------+---------------------+-----------------+---------------------------+
| ds         | ds2                 | epoch_ms      | epoch_s    | string0                    | string1             | string2         | string3                   |
+------------+---------------------+---------------+------------+----------------------------+---------------------+-----------------+---------------------------+
| 2017-07-19 | 2017-07-19 13:23:33 | 1500470613000 | 1500470613 | 2017-07-19 06:23:33.000000 | 2017-07-19^06:23:33 | 20170719-062333 | 2017/07/1906:23:33.000000 |
| 2016-11-23 | 2016-11-23 00:48:29 | 1479862109000 | 1479862109 | 2016-11-22 16:48:29.000000 | 2016-11-22^16:48:29 | 20161122-164829 | 2016/11/2216:48:29.000000 |
| 2015-02-27 | 2015-02-27 10:03:31 | 1425031411000 | 1425031411 | 2015-02-27 02:03:31.000000 | 2015-02-27^02:03:31 | 20150227-020331 | 2015/02/2702:03:31.000000 |
| 2011-05-02 | 2011-05-02 01:26:50 | 1304299610000 | 1304299610 | 2011-05-01 18:26:50.000000 | 2011-05-01^18:26:50 | 20110501-182650 | 2011/05/0118:26:50.000000 |
| 2016-12-11 | 2016-12-11 11:29:46 | 1481455786000 | 1481455786 | 2016-12-11 03:29:46.000000 | 2016-12-11^03:29:46 | 20161211-032946 | 2016/12/1103:29:46.000000 |
+------------+---------------------+---------------+------------+----------------------------+---------------------+-----------------+---------------------------+
```

6. random_time_series

```
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| ds    | datetime | YES  |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
```

该表只有一个字段，用来表示随机时间序列。


#### word cloud报表制作过程

![](http://pandora-kibana.qiniu.com/birth%20name%20word%20cloud.gif)


#### 饼状图的报表制作过程

![](http://pandora-kibana.qiniu.com/birth-name-pie.gif)


#### big number报表制作过程

![](http://pandora-kibana.qiniu.com/big-number.gif)

#### 带趋势的big number报表制作过程

![](http://pandora-kibana.qiniu.com/big-number-trendline.gif)

#### 柱状图报表制作过程

![](http://pandora-kibana.qiniu.com/distribute-bar.gif)

#### 桑基图报表制作过程

> 桑基图（Sankey diagram），即桑基能量分流图，也叫桑基能量平衡图。它是一种特定类型的流程图，图中延伸的分支的宽度对应数据流量的大小，通常应用于能源、材料成分、金融等数据的可视化分析。因1898年Matthew Henry Phineas Riall Sankey绘制的“蒸汽机的能源效率图”而闻名，此后便以其名字命名为“桑基图”。
因1898年Matthew Henry Phineas Riall Sankey绘制的“蒸汽机的能源效率图”而闻名，此后便以其名字命名为“桑基图”。
桑基图最明显的特征就是，始末端的分支宽度总和相等，即所有主支宽度的总和应与所有分出去的分支宽度的总和相等，保持能量的平衡。

适用场景：
1. 网站用户细分分析
2. 数据流向图

![](http://pandora-kibana.qiniu.com/report-sanky.gif)


#### treemap报表制作过程

![](http://pandora-kibana.qiniu.com/report-treemap.gif)


#### 热力图报表制作过程

![](http://pandora-kibana.qiniu.com/report-heatmap.gif)

#### 基于日历的热力图（活跃图）报表制作过程

![](http://pandora-kibana.qiniu.com/report-calendar-heatmap.gif)

#### 导向力报表制作过程

![](http://pandora-kibana.qiniu.com/report-directed-force.gif)

#### Sunburst报表制作过程

![](http://pandora-kibana.qiniu.com/report-sunburst.gif)

#### 气泡报表制作过程

![](http://pandora-kibana.qiniu.com/report-bubble.gif)

#### 箱形图报表制作过程

![](http://pandora-kibana.qiniu.com/report-box-plot.gif)