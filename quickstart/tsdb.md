### 概念
**数据仓库：**

`数据仓库`的概念和传统数据库的`database`概念相同，用来管理仓库中的`序列`。

**序列：**

`序列`的概念类似于传统数据库的`table`，其作用是用来存储数据。

时序数据库的`序列`是完全列自由的，用户可以随意向`序列`中打入不同数据格式的内容，字段的定义也在打入数据时指定。

因为时序数据库的完全列自由特性，通常我们把`序列`称为`宽表`，所以时序数据库是不支持Join和Union的。

**索引列（Tag）：**

一条数据通常由多个字段组成，时序数据库允许用户给字段打上`索引标签`，形成`索引列`。

被打上`索引标签`的字段，只能用来分组，无法单独作为查询主体，但可以和普通列同时作为查询主体。


**普通列（Filed）**

普通字段通常是我们的查询主体，它可以进行函数运算，但无法对他进行分组或排序。

**时间戳（TimeStamp）：**

一般来说，时序数据库中的每一条数据，都表示一个事件的发生，而`时间戳`，则是记录这些事件发生的时间，如果某条数据中缺省该值，则默认会使用当前时间。

`时间戳列`可以进行分组或排序，也可以用作条件语句，甚至可以计算，但无法作为查询主体。

**系统关键字：**

命名时请不要与关键字冲突（包括大小写）

```
server  repo  view  tagKey

ILLEGAL  EOF  WS  IDENT  BOUNDPARAM  NUMBER 
INTEGER  DURATIONVAL  STRING  BADSTRING  BADESCAPE   
TRUE  FALSE  REGEX  BADREGEX    

ADD  SUB  MUL  DIV  AND  OR  
EQ  NEQ  EQREGEX  NEQREGEX 
LT  LTE  GT  GTE      
	
LPAREN  RPAREN  COMMA  COLON  DOUBLECOLON  SEMICOLON  DOT         

ALL  ALTER  ANY  AS  ASC  
BEGIN  BY	
CREATE  CONTINUOUS
DATABASE  DATABASES  DEFAULT  DELETE  DESC  DESTINATIONS  DIAGNOSTICS  DISTINCT  DROP  DURATION
END	EVERY  EXISTS  EXPLAIN
FIELD  FOR  FROM
GROUP  GROUPS  
IF  IN  INF  INSERT  INTO  KEY  KEYS	  KILL
LIMIT  
MEASUREMENT  MEASUREMENTS
NAME  NOT  
OFFSET  ON  ORDER
PASSWORD  POLICY  POLICIES  PRIVILEGES
QUERIES  QUERY  READ  REPLICATION	  RESAMPLE  RETENTION  REVOKE	
SELECT  SERIES  SET  SHOW  SHARD  SHARDS  SLIMIT  SOFFSET  STATS  SUBSCRIPTION  SUBSCRIPTIONS
TAG  TO  TIME
VALUES 
WHERE  WITH  WRITE
```

**字段类型关系对应：**

> 消息队列中,字段的类型与时序数据库中的字段类型需要作出如下对应:
> 
> 消息队列类型:string 对应 时序数据库:string 
> 
> 消息队列类型:long 对应 时序数据库:long 
> 
> 消息队列类型:float 对应 时序数据库:float
> 
> 消息队列类型:date 对应 时序数据库:date 
> 
> 消息队列类型:boolean 对应 时序数据库:boolean 

!> 注意：时序数据库不支持复合类型(array[string/long/float])。

### 创建仓库与序列

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

![](/Users/loris/liurui/pandora-docs-old/_media/tsdb1.gif)

### 数据查询

首先我们选择一个数据仓库，然后在SQL编辑框里编写SQL，最后点击数据查询按钮/图表查询按钮即可。

> 提示：
> 
> 1.点击**查询字段信息**下拉列表，选择一个序列，然后点击右侧的Tags/Fileds可以查看这个序列的索引字段和普通字段。
> 
> 2.编写完一条SQL之后，可以点击**保存SQL语句**按钮，将这个SQL语句作为常用SQL，方便操作。

**操作演示：**

![](/Users/loris/liurui/pandora-docs-old/_media/tsdb2.gif)

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

