### 简介
Qiniu BI Studio 是七牛通用的自助报表平台。支持

* 从Pandora工作流实时和离线导入数据
* 支持多达30种图表类型
* 支持在线对报表数据进行探索、查询
* 支持分享单个图表

* 以下是报表系统支持的图表类型

![](http://pandora-kibana.qiniu.com/report-figure-type2.png)
![](http://pandora-kibana.qiniu.com/report-figure-type3.png)
![](http://pandora-kibana.qiniu.com/report-figure-type4.png)
![](http://pandora-kibana.qiniu.com/report-figure-type5.png)

### 激活报表系统

1. 首先登陆七牛portal，打开工作流。
2. 创建数据源，创建导出任务到报表。
3. 打开`https://bi-studio.qiniu.com`登录报表系统。

![](http://pandora-kibana.qiniu.com/report-login.png)

4. 依照以下操作指南进行操作。


### 报表操作指南

Pandora Report 主要分为四部分模块：

1. 数据源管理/数据集
2. 报表分析器
3. 报表、看板展现
4. SQL编辑器

![](http://pandora-kibana.qiniu.com/report-main.png)


### 分析制作图表

在用户第一次激活报表系统的时候，系统会导入一个名为`example_database`的样例数据库，包含若干个数据表；
这些数据表用来帮助用户理解和学习报表的使用方法。

```
+-------------------------+
| Tables_in_example       |
+-------------------------+
| birth_france_by_region  |
| birth_names             |
| energy_usage            |
| long_lat                |
| multiformat_time_series |
| random_time_series      |
| wb_health_population    |
+-------------------------+

```

不同数据表的描述如下：

1. birth_names

```
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