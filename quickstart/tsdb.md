### 创建仓库与序列

**操作流程：**

进入时序数据库服务，点击**创建仓库**按钮，开始创建第一个数据仓库。创建仓库完成后，在`序列`页面点击**创建序列**按钮，开始创建第一个序列。

**数据仓库填写参数说明：**

|参数|必填|说明|
|:---|:---|:---|
|仓库名称|是|用来区分其他仓库，唯一标识，用户自行命名|
|所属区域|是|所属区域,计算与存储所使用的物理资源所在区域,目前支持华东区域；</br>此参数是为了降低用户传输数据的成本；应当尽量选择离自己数据源较近的区域|

**序列填写参数说明：**

|参数|必填|说明|
|:---|:---|:---|
|序列名称|是|用来区分其他序列，唯一标识，用户自行命名|
|所属仓库|是|该序列属于哪个数据仓库|
|存储时限|是|数据存储的时限，最小为1天，最大为30天，存储时间超过这个时限的数据，将会被系统自动删除|

**操作演示：**

![](_media/tsdb-createRepo.gif)

### 数据查询

**操作流程：**

点击页面左侧的**数据查询**，进入查询页面。

首先我们选择一个数据仓库，然后在SQL编辑框里编写SQL，最后点击数据查询按钮/图表查询按钮即可。

> 提示：
> 
> 1.点击**查询字段信息**下拉列表，选择一个序列，然后点击右侧的Tags/Fileds可以查看这个序列的索引字段和普通字段。
> 
> 2.编写完一条SQL之后，可以点击**保存SQL语句**按钮，将这个SQL语句作为常用SQL，方便操作。

**操作演示：**

![](_media/search_tsdb.gif)

### SQL编写规范

我们用一个示例来讲解时序数据库的SQL编写规范以及一些核心的概念。

假设我们现在有一个水位监测的序列（可以理解为一张表），它的名称为`waterLevelInfo`，它所包含的信息如下：

|字段名称|字段类型|属性|备注|
|:----|:----|:----|:----|
|createTime|date| timestamp |数据产生时间，注意，timestamp的内容为RFC 3339 时间戳|
|id|long|filed|设备唯一编号|
|area|string|tag|设备所在区域|
|reach|string|tag|设备所在河段|
|waterLevel|float|filed|目前水位，单位为米|

##### 1.查询所有数据

?>select * from waterLevelInfo

!> 注意： 我们非常不建议用户使用 `*` 来做查询，它可能会产生大量的查询费用。

##### 2.查询某个字段的数据

只有属性为`filed`的列才能作为查询主体，`tag`无法单独作为查询主体。

**正确的查询：**

?>select waterLevel from waterLevelInfo

?>select waterLevel,area from waterLevelInfo

**错误的查询：**

!>select area from waterLevelInfo

查询语句支持若干种查询数据的格式：

* `select *` 返回所有的tag和field字段
* `select <field_key>` 返回特定的field
* `select <field_key>,<field_key>` 返回特定的多个field
* `select <field_key>,<tag_key>` 返回特定的tag和field，注意`select`必须至少包含一个`field_key`

>如果标识符号包含除了[A-z,0-9,_]之外的符号，或者包含TSDB关键字，又或者以数字开头，那么必须用双引号扩起来。

##### 3.对查询主体做函数运算

函数运算同样只能在`filed`字段上，`tag`和`timestamp`只能作为分组和查询条件。

目前时序数据库支持的聚合函数：

```
count:总数
sum:总和
mean:平均值
distinct:去重
max:最大
min:最小
bottom(field,N):最小的N个值
top(field,N):最大的N个值
first:时间戳最新的值
last:时间戳最老的值
difference【不保证精确】
```

正确的查询：

?>select mean(waterLevel) from waterLevelInfo

错误的查询：

!>select count(area) from waterLevelInfo

##### 4.条件语句

条件语句根据tag、fields、timestamp来对数据进行筛选。

**正确的查询：**

?>select * from waterLevelInfo where area = 'shanghai' or area = 'beijing'

?>select * from waterLevelInfo where waterLevel > 5 and area = 'shanghai'

**错误的查询：**

!>select * from waterLevelInfo where area = beijing 

当作为查询条件的schema属性为`filed`时，支持以下运算符：

* `=` 等于
* `<>` 不等于
* `!=` 不等于
* `>` 大于
* `>=` 大于等于
* `<` 小于
* `<=` 小于等于

当作为查询条件的schema属性为`tag`时，支持以下运算符：

* `=` 等于
* `<>` 不等于
* `!=` 不等于

当作为查询条件的schema属性为`timestamp`时：

##### 5.分组

Group By在时序数据库中可以对`tag`和`timestamp`使用。

Group By`timestamp`时，必须拥有where timestamp条件。

语法规则：

* `GROUP BY *` 根据所有的tag进行分组
* `GROUP BY <tag_key>` 根据单个tag进行分组
* `GROUP BY <tag_key>,<tag_key>` 根据多个tag进行分组

!> 注意： 如果查询语句中包含条件语句，那么分组必须在条件语句之后!

**正确的查询：**

?>select mean(waterLevel) from waterLevelInfo group by area

?>select mean(waterLevel) from waterLevelInfo where time > now() - 3d group by time(5m)

**错误的查询：**

!>select mean(waterLevel),area from waterLevelInfo group by id

##### 6.排序

Order By在时序数据库中可以对`timestamp`使用。

在Order By时，`timestamp`可以指定一个`time_interval`，用来表示对时间的聚合粒度。

正确的查询：

?>select mean(waterLevel) from waterLevelInfo order by time(5m) desc

错误的查询：

!>select mean(waterLevel),area from waterLevelInfo order by id desc

##### 7.时间作为查询条件

?> select mean(waterLevel),area from waterLevelInfo where createTime > now() - 5m

?> select mean(waterLevel),area from waterLevelInfo where createTime > now() - 3d and createTime > now() - 1d

?> select mean(waterLevel),area from waterLevelInfo where createTime > `'2017-01-01T13:00:00Z'`

!>注意：`now()`表示当前时间， '-'符号的左右两边都必须至少包含一个空格。

> `s`=秒，`m`=分，`h`=小时，`d`=天





### SQL高级用法

##### LIMIT

```
SELECT_clause FROM TABLE [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] LIMIT <N>
```

`N`用来指定返回的数据点的数量，如果N大于数据点总量，那么返回所有的数据点

##### 时间查询语法

对于没有写时间条件的查询语句，默认的查询时间范围为`1677-09-21T00:12:43.145224194`到`2262-04-11T23:47:16.854775806Z`.
如果查询语句中包含group by语句，那么默认的查询时间范围是`1677-09-21T00:12:43.145224194`到`now()`.

##### 绝对时间

```
SELECT_clause FROM TABLE WHERE time <operator> ['<rfc3339_date_time_string>' | '<rfc3339_like_date_time_string>' | <'epoch_time'>] [AND ['<rfc3339_date_time_string>' | '<rfc3339_like_date_time_string>' | <'epoch_time'>] [...]]
```

支持的运算符：

* `=` 等于
* `<>` 不等于
* `!=` 不等于
* `>` 大于
* `>=` 大于等于
* `<` 小于
* `<=` 小于等于

目前TSDB不支持对绝对时间进行`OR`操作


**rfc3339_date_time_string解释：**

```
YYYY-MM-DDTHH:MM:SS.nnnnnnnnnZ
```
其中，`.nnnnnnnnn`是可选的，如果不写的话，默认值为`.000000000`。 
`rfc3339_date_time_string`必须用单引号括起来。

**rfc3339_like_date_time_string解释：**

```
YYYY-MM-DD HH:MM:SS.nnnnnnnnn
```
其中，`.nnnnnnnnn`是可选的，如果不写的话，默认值为`.000000000`。 
`rfc3339_like_date_time_string`必须用单引号括起来。

**epoch_time解释：**

`epoch_time`是自1970-01-01 00：00：00以来的纳秒数，如果需要以秒表示那么需要在字符串后面加`s`.


##### 相对时间

TSDB中采用`now()`来表示相对时间，该时间为服务器上的当前时间.

```
SELECT_clause FROM TABLE WHERE time <operator> now() [[- | +] <duration>] [(AND|OR) now() [...]]
```

`now()`是sql执行当时的服务器时间， 需要注意的是`duration`和`[- | +]`之间需要一个`空格`。 

支持的运算符

* `=` 等于
* `<>` 不等于
* `!=` 不等于
* `>` 大于
* `>=` 大于等于
* `<` 小于
* `<=` 小于等于

支持的单位

* `u` 微秒
* `ms` 毫秒
* `s` 秒 
* `m` 分钟 
* `h` 小时
* `d` 天
* `w` 星期



### 使用Grafana

#### Grafana是什么
[Grafana](https://oiw6da4op.qnssl.com/181919_0KHT_865233.jpg)
Grafana是开源的用来实时展示、分析和报警的软件，在七牛的应用中心提供Grafana应用，Pandora TSDB适配了Grafana，可以用Grafana实时展示、分析TSDB中的数据，并报警。

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

输入您的app名和应用别名，选择部署区域(注意！目前TSDB的数据在`华东`，所以选择华东区域会得到更好的体验)，点击确定创建

应用名称：账号内唯一应用名称，且只能满足以下条件：（1. 只能包含字母、数字和减号，首尾字符只能为字母或数字。 2. 字符长度不能超过 30）
应用别名：供显示使用的标题名。

3. 等待app启动后，输入密码（密码长度必须大于6），点击 确认配置

![Grafana](https://oiw6da4op.qnssl.com/grafana/img3.png)

4. 访问Grafana进入Grafana页面

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

