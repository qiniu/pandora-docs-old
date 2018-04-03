### 创建数据源
在工作流编辑器中，我们首先看到的节点是数据源，这个节点用来接收用户的实时数据，也就是说，当这个节点被创建后，用户需要将自己的数据推送至这个数据源中，才可以继续进行下一步。

数据源节点包含两部分内容： 

1.名称：用来标识这个节点的唯一性，同时这个名字也作为工作流的名称。

2.字段信息：用来定义数据结构；在类型方面，目前支持以下类型：

```
string
float
long
date
boolean
array[string/long/float]
map
jsonstring
```

!> 注意：`float`类型的精度是64位。

!> 注意：`jsonstring`类型的取值必须为符合json格式的字符串（以`{}`或者`[]`包括，例如`{"user": "Jone", "age": 25}`、`["abc", 123]`等）。`jsonstring`类型适用于json内部字段名称、数目不确定的使用场景，例如某一`jsonstring`类型可以取值为`{"user": "Jone", "age": 25}`，也可以取值为`{"user": "Jone", "weight": "60KG"}`，字段名称或者数目发送变化时不需要对消息队列的schema进行任何更改，便于导出到`logdb`类型为`object`的字段中，然后就可以使用json内部字段名称进行常规搜索。

!> 注意：`jsonstring`类型存在局限性：某一`jsonstring`类型的字段经过transform后将被转换为普通的`string`类型。

!> 注意：每一个数据源/消息队列中的每一条数据都将被保存2天时间，超过时间后自动删除。

> 数据源其实也是一个消息队列，技术人员可以将它理解成kafka中的一个topic。

**数据源节点填写参数说明：**

* 名称：必填项；命名规则: 1-128个字符,支持小写字母、数字、下划线；必须以大小写字母或下划线开头。
* 字段名称：必填项；用来标识字段唯一性；命名规则: 1-128个字符,支持小写字母、数字、下划线；必须以大小写字母或下划线开头。
* 字段类型：必填项；描述字段的数据类型。

填写完成相应信息后，点击**创建**按钮，第一个工作流就创建成功了。

**操作演示：**

![](/Users/loris/liurui/pandora-docs-old/_media/flow1.gif)

### 计算任务

我们可以对数据源节点中的数据进行计算，计算完成后，会生成一个新的消息队列来存放计算结果，我们也可以对消息队列节点中的数据进行计算，这样就形成了一个复杂的工作流程。

计算的方式分为两种：标准SQL计算和自定义代码计算，它们两者可以并存，执行的顺序是先执行自定义代码计算，后执行标准SQL计算；在一个计算任务中，至少需要指定一种计算方式。

SQL计算就是自己编写SQL语句，对数据源或消息队列中的数据进行分批处理，并且以一定的时间粒度(可以自行设置，最小的粒度为5秒)进行聚合，将结果自动发送到一个新的消息队列中。

计算任务是消耗物理资源的，所以每一个计算任务节点都需要为之分配**计算资源**（CPU & 内存）， 不同的计算任务之间的计算资源互相隔离，互不影响。

!> 注意：SQL中，from关键字之后的table名称为**stream**

>高级用法：自定义计算请见下文

**操作演示：**

![](/Users/loris/liurui/pandora-docs-old/_media/flow2.gif)

### 使用UDF

UDF(User-Defined Function)用户自定义函数。

在使用SQL方式进行计算时，pandora提供了大量的udf函数供用户使用，如果这些函数不能满足需求，那么用户可以自行上传Jar包，注册自定义udf。

在Pandora中，UDF和Plugin的差别如下：

* UDF注册的函数只适用于SQL语句中；

#### 系统提供的UDF
系统默认提供了上百种udf，分别为：数学函数、日期函数、字符串函数、聚合函数和窗口函数，我们可以在工作流列表的右上角 `UDF管理` 中查看。

![](/Users/loris/liurui/pandora-docs-old/_media/flow11.gif)

#### 自定义UDF

我们以一个示例来讲解自定义udf的方法。

首先，在下方中，下载 UDF-java 项目工程。

点击右侧链接下载 -> [UDF-Java.zip](http://onzeipdi1.bkt.clouddn.com/UDF-Java.zip)

解压后，使用Java IDE导入Pandora-UDF项目。

等待项目依赖加载完成后，可以在 `src/main/java/com.pandora/` 目录下查看一个简单的示例。

这个示例中包含了一个名为 `SimpleUdf`的Class，在这个Class中有4个方法：

```
1. String parseTime(String t)
将 Input RFC3339 格式转为 date time 时间格式
@param input rfc3339 格式，形如 2017-04-05T16:41:42.651614Z
@return 返回date time格式时间 形如 2017-04-05 16:41:42
如 parseTime("2017-04-05T16:41:42.651614Z")

2. String parseTime(long t)
将时间戳转为 date time 时间格式
@param input 时间戳，单位为毫秒
@return 返回date time格式时间 形如 2017-04-05 16:41:42
如 parseTime(1499324233000)

3. String parseTime(long t, String unit)
将 Input RFC3339 格式转为 date time 时间格式
@param input 时间戳，单位为毫秒
@param unit 指定时间戳的单位，支持 s （秒）, ms（毫秒）, us（微妙）, ns (纳秒)
@return 返回date time格式时间 形如 2017-04-05 16:41:42
如 parseTime(1499324233000, "ms")

3. String parseTime(long t, String unit)
将 Input RFC3339 格式转为 date time 时间格式
@param input 时间戳，单位为毫秒
@param unit 指定时间精度，1毫秒等于多少该精度单位时间
@return 返回date time格式时间 形如 2017-04-05 16:41:42
如 parseTime(149932423300000000, 100000) 解析百纳秒时间戳
```

![](/Users/loris/liurui/pandora-docs-old/_media/flow12.gif)

我们可以在 `src/main/java/com.pandora/` 目录下新建Class和方法，并在方法中编写udf逻辑，代码编写完成后，需要将这个工程打成Jar包并上传至Pandora，然后就可以注册并使用这个udf了。

!>注意：Jar包名称中不可包含`-`号。

![](/Users/loris/liurui/pandora-docs-old/_media/flow13.gif)

上传Jar包并注册UDF成功后，即可在SQL中使用，示例：

```
SELECT
  parseTime(t1) t1,
  parseTime(t2) t2,
  parseTime(t3, "s") t3, 
  parseTime(t4, 100000) t4 
from 
  stream
```

### 自定义计算（Plugin）- Java

#### 下载&加载项目

**操作流程：**

首先，在下方中，下载 Plugin-Java 项目工程。

点击右侧链接下载 -> [Plugin-Java.zip](http://7xtfy5.com1.z0.glb.clouddn.com/Pandora-Plugin-Java.zip)

解压后，使用Java IDE导入Pandora-Plugin项目。

等待项目依赖加载完成后，可以在 `src/main/java/demo/` 目录下查看一个简单的示例。

而 `src/main/java/userWrite/` 目录下是用户自行编写代码的地方。

**操作演示：**

![](/Users/loris/liurui/pandora-docs-old/_media/flow3.gif)

#### 编写输入&输出类

**操作流程：**

首先，我们要定义我们编写代码的输入和输出。

输入通常是工作流中的消息队列，那么我们只需要将消息队列中的字段信息原封不动的复制一份即可，而输出是由我们自己编写的代码决定的，所以也需要提前定义好我们的输出信息。

负责数据源的类是 `src/main/java/userWrite/bean/SourceData.java`

负责输出数据的类是 `src/main/java/userWrite/bean/OutputData.java`

我们只需要根据 `src/main/java/demo/` 目录下的示例进行编写即可，代码中也有很详细的注释。

#### 编写业务逻辑代码

业务逻辑代码的编写在 `src/main/java/userWrite/customLogic/customlogic.java` 中的**parse**方法中。

!>注意：customlogic.java 此类名称可自由更改；最终的返回必须是List&lt;OutputData&gt;。

#### 打包上传

代码编写完成后，需要打成Jar包，然后上传至Pandora平台中。

!>注意：Jar包名称必须和 `src/main/java/userWrite/customLogic/` 目录下包含parse方法的类名称一致。

**操作演示：**

![](/Users/loris/liurui/pandora-docs-old/_media/flow4.gif)


### 数据导出

任何一个数据源和消息队列节点中的数据，都可以导出。

目前我们支持将数据导出到对象存储、时序数据库、日志检索服务、HTTP服务器，并且支持导出到RDS，如果您有这方面的需求，请通过工单或客服联系我们。

### 导出数据至HTTP

**导出任务节点填写参数说明：**

* 名称：必填项；用来标识这个导出任务的唯一性；命名规则: 1-128个字符,支持小写字母、数字、下划线；必须以大小写字母或下划线开头。
* 服务器地址：必填项；服务器地址（ip或域名），例如:`https://pipeline.qiniu.com` 或 `127.0.0.1:7758`。
* 请求资源路径：必填项；请求资源路径（具体地址,不包含ip或域名），例如:`/test/repos`。

!> 注意: 导出数据格式和`推送数据`相同。

**操作演示：**

![](/Users/loris/liurui/pandora-docs-old/_media/flow5.gif)

### 导出数据至对象存储

**导出任务节点填写参数说明：**

* 名称：必填项；用来标识这个导出任务的唯一性；命名规则: 1-128个字符,支持小写字母、数字、下划线；必须以大小写字母或下划线开头。
* 空间名称：必填项；对象存储中的Bucket名称。
* 文件前缀：必填项；导出的文件名称的前缀。
* 导出账号：必填项；空间名称所属用户的七牛账户名称。（目前允许跨账号导出）
* 账号公钥：必填项；七牛账户的公钥，个人面板-密钥管理中查看`AK`即公钥。
* 导出类型：必填项；可以将文件导成三种格式：json、text、parquet；其中json和text可以选择是否将文件压缩，而parquet无需选择，默认自动压缩，压缩比大概为3-20倍。
* 最大文件保存天数：必填项；文件存储时效，超过这个时间范围的文件会被自动删除，为0的话则永久存储。

!> 关于文件前缀,默认值为""(生成文件名会自动加上时间戳格式为`yyyy-MM-dd-HH-mm-ss`),如果使用了一个或者多个魔法变量时不会自动添加时间戳,支持魔法变量,采用`$(var)`的形式求值,目前可用的魔法变量var如下:

* `year` 上传时的年份
* `mon` 上传时的月份
* `day` 上传时的日期
* `hour` 上传时的小时
* `min` 上传时的分钟
* `sec` 上传时的秒钟

>举例说明:
>
* 假如keyPrefix取值为kodo-parquet/date=$(year)-$(mon)-$(day)/hour=$(hour)/min=$(min)/$(sec),且生成某一文件时的北京标准时间为`2017-01-12 15:30:00`, 则keyPrefix将被解析为kodo-parquet/date=2017-01-12/hour=15/min=30/00,其中的魔法变量$(year)、$(mon)、$(day)、$(hour)、$(min)、$(sec)分别对应文件生成时间`2017-01-12 15:30:00`的年、月、日、时、分、秒。
* 假如keyPrefix使用默认值,且生成某一文件时的北京标准时间为`2017-01-12 15:30:00`, 则keyPrefix将被解析为`2017-01-12-15-30-00`。 

**操作演示：**

![](/Users/loris/liurui/pandora-docs-old/_media/flow6.gif)

### 导出数据至日志检索服务

**导出任务节点填写参数说明：**

* 名称：必填项；用来标识这个导出任务的唯一性；命名规则: 1-128个字符,支持小写字母、数字、下划线；必须以大小写字母或下划线开头。
* 数据仓库：选择或输入一个数据仓库，数据将会被导入到这个仓库当中。
* 字段信息：需要将消息队列和日志仓库中的字段信息一一对应。

!>注意：如果您还没有创建日志仓库，那么请参见[日志检索服务](/quickstart/logdb)

> 消息队列中,字段的类型与日志检索服务中的字段类型需要作出如下对应:
> 
> 消息队列类型:string 对应 日志检索服务:string / date
> 
> 消息队列类型:long 对应 日志检索服务:long / date
> 
> 消息队列类型:float 对应 日志检索服务:float
> 
> 消息队列类型:date 对应 日志检索服务:date 
> 
> 消息队列类型:boolean 对应 日志检索服务:boolean
> 
> 消息队列类型:array[string] 对应 日志检索服务:string
> 
> 消息队列类型:array[long] 对应 日志检索服务:long
> 
> 消息队列类型:array[float] 对应 日志检索服务:float
> 
> 消息队列类型:jsonstring 对应 日志检索服务:object

**操作演示：**

![](/Users/loris/liurui/pandora-docs-old/_media/flow7.gif)

### 导出数据至时序数据库

**导出任务节点填写参数说明：**

* 名称：必填项；用来标识这个导出任务的唯一性；命名规则: 1-128个字符,支持小写字母、数字、下划线；必须以大小写字母或下划线开头。
* 数据仓库：必填项；选择或输入一个数据仓库。
* 序列：必填项；选择或输入一个序列，数据将会被导入到这个序列当中。
* 时间戳：非必填项；指定一个时间，会用rfc3339日期格式进行解析,如果格式不正确则会抛弃这一条数据,如果此项为空,则默认使用当前时间。
* 字段信息：

!>注意：如果您还没有创建数据仓库和序列或不了解其具体含义，那么请参见[时序数据库](/quickstart/tsdb)

**操作演示：**

![](/Users/loris/liurui/pandora-docs-old/_media/flow8.gif)

### 复杂的业务逻辑

工作流编辑器可以完成非常复杂的数据业务，它的整个架构是完全`拓扑逻辑`的。

依靠`消息队列`作为数据中间件，数据的计算可以往复循环，以达到满足复杂业务的需求。

**操作演示：**

![](/Users/loris/liurui/pandora-docs-old/_media/flow9.gif)

### 便捷操作

`复制&粘贴`动作，快速完成重复工作。

`复制&粘贴`动作可以仅复制数据结构，也可以复制数据结构+内容。

**操作演示：**

![](/Users/loris/liurui/pandora-docs-old/_media/flow10.gif)
