### 创建仓库与序列

**操作流程：**

进入时序数据库服务，点击**创建仓库**按钮，开始创建第一个数据仓库。创建仓库完成后，在`序列`页面点击**创建序列**按钮，开始创建第一个序列。

**数据源/消息队列节点填写参数说明：**

|参数|必填|说明|
|:---|:---|:---|
|仓库名称|是|用来区分其他仓库，唯一标识，用户自行命名|
|所属区域|是|所属区域,计算与存储所使用的物理资源所在区域,目前支持华东区域；</br>此参数是为了降低用户传输数据的成本；应当尽量选择离自己数据源较近的区域|


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
|area|string|tags|设备所在区域|
|reach|string|tags|设备所在河段|
|waterLevel|float|filed|目前水位，单位为米|

##### 1.查询所有数据

?>select * from waterLevelInfo

##### 2.查询某个字段的数据

只有属性为`filed`的列才能作为查询主体，`tags`无法单独作为查询主体。

正确的查询：

?>select waterLevel from waterLevelInfo

?>select waterLevel,area from waterLevelInfo

错误的查询：

!>select area from waterLevelInfo

##### 3.对查询主体做函数运算

函数运算同样只能在`filed`字段上，`tags`和`timestamp`只能作为分组和查询条件。

目前时序数据库支持的聚合函数：

```
count
sum:总和
mean:平均值
distinct
bottom(field,N):最小的N个值
top(field,N):最大的N个值
max
min
first:时间戳最新的值
last:时间戳最老的值
difference【不保证精确】
```

正确的查询：

?>select mean(waterLevel) from waterLevelInfo

错误的查询：

!>select count(area) from waterLevelInfo

##### 4.分组

Group By在时序数据库中可以对`tags`和`timestamp`使用。

Group By`timestamp`时，必须拥有where timestamp条件。

正确的查询：

?>select mean(waterLevel) from waterLevelInfo group by area

?>select mean(waterLevel) from waterLevelInfo where time > now() - 3d group by time(5m)

错误的查询：

!>select mean(waterLevel),area from waterLevelInfo group by id

##### 5.排序

Order By在时序数据库中可以对`timestamp`使用。

正确的查询：

?>select mean(waterLevel) from waterLevelInfo order by time desc

错误的查询：

!>select mean(waterLevel),area from waterLevelInfo order by id desc

##### 6.时间作为查询条件

?> select mean(waterLevel),area from waterLevelInfo where createTime > now() - 5m

?> select mean(waterLevel),area from waterLevelInfo where createTime > now() - 3d and createTime > now() - 1d

?> select mean(waterLevel),area from waterLevelInfo where createTime > `'2017-01-01T13:00:00Z'`

!>注意：`now()`表示当前时间， '-'符号的左右两边都必须至少包含一个空格。

> `s`=秒，`m`=分，`h`=小时，`d`=天

### SQL语法

## 基本的SELECT语句

### 语法

`select`用来对特定的序列进行查询，语法如下:

```
SELECT <field_key>[,<field_key>,<tag_key>] FROM <measurement_name>
```
`select`语句需要一个`select`和一个`from`

### 语法描述

#### select clause

`select`支持若干种查询数据的格式
* `select *` 返回所有的tag和fields字段
* `select <field_key>` 返回特定的field
* `select <field_key>,<field_key>` 返回特定的多个field
* `select <field_key>,<tag_key>` 返回特定的tag和field，注意`select`必须包含一个`field_key`

#### from clause

`from`表示从特定的seires获取数据

### 引号的用法

如果标识符号包含除了[A-z,0-9,_]之外的符号，或者包含TSDB关键字，又或者以数字开头，那么必须用双引号扩起来.

### 举例


### 常见问题
1. select语句必须至少包含一个field key，不能只有tag key

## WHERE语句
`WHERE`语句用来根据tags、fields、timestamp来对数据进行条件查询。

### 语法

```
SELECT_clause FROM TABLE WHERE <condition_expression> [(AND|OR) <condition_expression> [...]]
```

#### `WHERE`语句支持以fields为条件进行查询
```
field_key <operator> ['string` | boolean | float | integer]
```
支持的operator
`=` 等于
`<>` 不等于
`!=` 不等于
`>` 大于
`>=` 大于等于
`<` 小于
`<=` 小于等于


#### `WHERE`语句支持以tags为条件进行查询
```
tag_key <operator> ['tag_value']
```
支持的operator
`=` 等于
`<>` 不等于
`!=` 不等于

#### `WHERE`语句支持以timestamp为条件进行查询
对于没有写时间条件的查询语句，默认的查询时间范围为`1677-09-21T00:12:43.145224194`到`2262-04-11T23:47:16.854775806Z`.
如果查询语句中包含group by语句，那么默认的查询时间范围是`1677-09-21T00:12:43.145224194`到`now()`.

### 常见问题
1. `WHERE`查询没有数据返回
大部分情况下是因为tag value或者是string类型的field value没有加单引号或者错误的加上了双引号。

## GROUP BY语句

将查询的结果按照指定的tag进行分组。

### 语法
```
SELECT_clause From TABLE [WHERE_clause] GROUP BY[* | <tag_key>[,tag_key]]
```

`GROUP BY *` 根据所有的tag进行分组
`GROUP BY <tag_key>` 根据单个tag进行分组
`GROUP BY <tag_key>,<tag_key>` 根据多个tag进行分组

如果查询语句包含`WHERE`语句，`GROUP BY`语句必须在`WHERE`语句之后。


## GROUP BY time语句
`GROUP BY time`语句根据时间条件对查询结果进行过滤

### 基本语法

```
SELECT <function>(<field_key>) FROM TABLE WHERE <time_range> GROUP BY time(<time_interval>),[tag_key]
```
基本的`GROUP BY time`语法需要一个聚合函数在`SELECT`语句内，并且需要一个时间范围在`WHERE`语句内。

#### `time(<time_interval)` 
`time_interval`是duration类型的，该值用来确定按照什么时间段对结果进行分组，比如当`time_interval`为`5m`的时候，表示对结果按照5分钟的粒度分组


## ORDER BY time DESC

默认情况下TSDB按照时间升序返回结果，第一个点的时间是最老的，最后一个点的时间是最新的
`ORDER BY time DESC`按照时间降序返回结果。
`ORDER BY time DESC`必须在`GROUP BY`语句之后，必须在`WHERE`语句之后

## LIMIT
语法
```
SELECT_clause FROM TABLE [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] LIMIT <N>
```
`N`用来指定返回的数据点的数量，如果N大于数据点总量，那么返回所有的数据点

## time语法

对于没有写时间条件的查询语句，默认的查询时间范围为`1677-09-21T00:12:43.145224194`到`2262-04-11T23:47:16.854775806Z`.
如果查询语句中包含group by语句，那么默认的查询时间范围是`1677-09-21T00:12:43.145224194`到`now()`.

### 绝对时间
```
SELECT_clause FROM TABLE WHERE time <operator> ['<rfc3339_date_time_string>' | '<rfc3339_like_date_time_string>' | <'epoch_time'>] [AND ['<rfc3339_date_time_string>' | '<rfc3339_like_date_time_string>' | <'epoch_time'>] [...]]
```

支持的operator
`=` 等于
`<>` 不等于
`!=` 不等于
`>` 大于
`>=` 大于等于
`<` 小于
`<=` 小于等于

目前TSDB不支持对绝对时间进行`OR`操作

#### rfc3339_date_time_string

```
YYYY-MM-DDTHH:MM:SS.nnnnnnnnnZ
```
其中，`.nnnnnnnnn`是可选的，如果不写的话，默认值为`.000000000`。 
`rfc3339_date_time_string`必须用单引号括起来。

#### rfc3339_like_date_time_string

```
YYYY-MM-DD HH:MM:SS.nnnnnnnnn
```
其中，`.nnnnnnnnn`是可选的，如果不写的话，默认值为`.000000000`。 
`rfc3339_like_date_time_string`必须用单引号括起来。

#### epoch_time
`epoch_time`是自1970-01-01 00：00：00以来的纳秒数，如果需要以秒表示那么需要在字符串后面加`s`.


### 相对时间

TSDB中采用`now()`来表示相对时间，该时间为服务器上的当前时间.

```
SELECT_clause FROM TABLE WHERE time <operator> now() [[- | +] <duration>] [(AND|OR) now() [...]]
```

`now()`是sql执行当时的服务器时间， 需要注意的是`duration`和`[- | +]`之间需要一个`空格`。 

支持的operator
`=` 等于
`<>` 不等于
`!=` 不等于
`>` 大于
`>=` 大于等于
`<` 小于
`<=` 小于等于

支持的duration
`u` 微秒
`ms` 毫秒
`s` 秒 
`m` 分钟 
`h` 小时
`d` 天
`w` 星期

### 常见问题
1. 对绝对时间条件进行`OR`操作
目前TSDB不支持对绝对时间进行`OR`操作

2. 查询`将来`数据和`过期`数据 
如果查询大于`now()`的数据，必须在查询条件中增加一个时间上界，比如`where time > now() + 10d`会返回将来十天的数据（如果有的话)
TSDB对于过期的数据会自动删除

## 数据类型
field value可以是`float`,`integer`,`string`或者`boolean`.
同一个字段不能拥有两个以上的数据类型，否则数据将被丢弃

### 使用Grafana