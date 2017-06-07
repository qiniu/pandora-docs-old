### API接收地址

```
https://pipeline.qiniu.com
```

### API返回内容

**响应报文** 

* 如果请求成功,返回HTTP状态码`200`:

```
HTTP/1.1 200 OK
```

* 如果请求失败,返回包含如下内容的JSON字符串（已格式化,便于阅读）:

```
{
    "error":   "<errMsg    string>"
}
```

* 如果请求包含数据获取,则返回相应数据的JSON字符串；

### 创建离线数据源

**请求语法**

 ```
 POST /v2/datasources/<DataSourceName>
 Content-Type: application/json
 Authorization: Pandora <auth>
 {
     "region": <Region>,
     "type": <Type>,
     "spec": <Spec>,
     "schema": [
       {
         "key": <Key>,
         "valtype": <ValueType>,
         "required": <Required>
       },
       ...
     ]
 }
 ```
 
**请求内容**

 |名称|类型|必填|描述|
 |:---|:---|:---|:---|
 |DataSourceName|string|是|数据源节点名称；</br>命名规则:`^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`,1-128个字符,支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
 |region|string|是|所属区域,计算与存储所使用的物理资源所在区域,目前支持华东(`nb`)；</br>此参数是为了降低用户传输数据的成本,应当尽量选择离自己数据源较近的区域|
 |type|string|是|数据源类型，可选值为`kodo`和`hdfs`|
 |spec|json|是|指定该数据源自身属性相关的信息|
 |schema|array|是|字段信息|
 |schema.key|string|是|字段名称；</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`,1-128个字符,支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
 |schema.valtype|string|是|字段类型，支持`long`、`float`、`string`、`date`|
 |schema.required|bool|是|描述用户在传输数据时`key`字段是否必填|

 当type为`hdfs`的时候spec定义如下:
 
 |名称|类型|必填|描述|
 |:---|:---|:---|:---|
 |spec.paths|array|是|包含一个或者多个hdfs文件路径，例如：`hdfs://192.168.1.1:9000/usr/local`|
 |spec.fileType|string|是|文件类型，合法取值为`json`、`text`和`parquet`|

 当type为`kodo`的时候spec定义如下:
 
 |名称|类型|必填|描述|
 |:---|:---|:---|:---|
 |spec.bucket|string|是|对象存储bucket名称，</br>命名规则：4-63个字符，支持字母、数字、中划线|
 |spec.keyPrefixes|array|否|包含一个或者多个文件前缀；</br>命名规则：0-128个字符，不支持英文 `:` 、`\`、`<`、`>`符号|
 |spec.fileType|string|是|文件类型，合法取值为`json`、`text`和`parquet`|


### 更新离线数据源

**请求语法**

 ```
 PUT /v2/datasources/<DataSourceName>
 Content-Type: application/json
 Authorization: Pandora <auth>
 {
     "spec": <Spec>,
     "schema": [
       {
         "key": <Key>,
         "valtype": <ValueType>,
         "required": <Required>
       },
       ...
     ]
 }
 ```

!> 注意: 更新离线数据源schema的时候，不允许减少字段，也不允许更改字段的类型。


### 按照名称查看离线数据源

**请求语法**

```
GET /v2/datasources/<DataSourceName>
Authorization: Pandora <auth>
```

**响应报文**

 ```
 {
     "region": <Region>,
     "type": <Type>,
     "spec": <Spec>,
     "schema": [
       {
         "key": <Key>,
         "valtype": <ValueType>,
         "required": <Required>
       },
       ...
     ]
 }
 ```


### 列举离线数据源

**请求语法**

```
GET /v2/datasources
Authorization: Pandora <auth>
```

**响应报文**

 ```
 {
   "datasources": [
     {
       "name": <DataSourceName>,
       "region": <Region>,
       "type": <Type>,
       "spec": <Spec>,
       "schema": [
         {
           "key": <Key>,
           "valtype": <ValueType>,
           "required": <Required>
         },
         ...
       ]
     },
     ...
   ]
 }
 ```


### 按照名称删除离线数据源

**请求语法**

```
DELETE /v2/datasources/<DataSourceName>
Authorization: Pandora <auth>
```


### 创建离线计算任务

**请求语法** 

```
POST /v2/jobs/<JobName>
Content-Type: application/json
Authorization: Pandora <auth>
{
	"srcs":[
		{
			"name":<DataSourceName|JobName>,
			"fileFilter":<fileFilter>,
			"type":<DataSource|Job>,
			"tableName": <TableName>
		},
		...
	],
   "computation": {
       "code": <Code>,
       "type": <SQL>
   }, 
   "container": {
       "type": <ContainerType>,
       "count": <ContainerCount>
   },  
   "scheduler":{
       "type": <crontab|loop|manual|depend>,
       "spec": {
           "crontab": <0 0 0/1 * * ?>,
           "loop": <1h|3m|....>
       } 
   },
   "params":[
       {
           "name":<ParamName>,
           "default":<ParamValue>
       },
       ...
   ]
}
```
 

> scheduler.type 为 depend的时候，container依赖上游的配置，该配置不填
> 
> type 为 manual 和depend 的时候spec 可以不填
> 
> type 为 loop时，不填该spec.loop或者该字段为0，则默认持续运行该任务
> 
> type 为 depend的时候，params依赖上游的配置，该配置不填

**请求内容**

|名称|类型|必填|描述|
|:---|:---|:---|:---|
|srcs|array|是|数据来源，所有数据来源中最多包含一个离线任务，但可以包括多个离线数据源|
|srcs.name|string|是|数据源名称或离线任务名称|
|srcs.fileFilter|string|否|文件过滤规则，可使用魔法变量，</br>命名规则：0-64个字符|
|srcs.type|string|是|数据来源节点类型|
|srcs.tableName|string|是|数据来源表名称，</br>命名规则：1-128个字符，支持字母、数字、下划线，必须以字母开头|
|computation|object|是|计算|
|computation.code|string|是|代码片段，可以使用魔法变量|
|computation.type|string|是|代码类型，支持SQL|
|container|map|是|计算资源|
|container.type|string|是|规格，目前支持：1U2G ，1U4G，2U4G，4U8G，4U16G ，8U16G ，8U32G ，16U32G，16U64G|
|container.count|int|是|数量，所选规格 * 数量 <= 100U|
|scheduler|map|是|调度|
|scheduler.type|string|是|调度方式，定时、循环或单次执行三选一，下游任务是依赖模式|
|scheduler.spec.crontab|string|否|定时执行，当调度方式选择为'定时'，此项必填；必须为crontab类型|
|scheduler.spec.loop|string|否|循环执行，当调度方式选择为'循环'，此项必填；</br>其值以`m`(分钟)和`h`(小时)为单位，由数字与单位组成，例如:`5m`|
|params|array|否|魔法变量，系统默认自带6个魔法变量：`$(year)=当前年份`、`$(mon)=当前月份`、`$(day)=当前日期`、`$(hour)=当前小时`、`$(min)=当前分钟`、`$(sec)=当前秒数`；用户也可以自行定义魔法变量|
|params.name|string|是|变量名称,命名规则：1-64个字符，支持大小写字母、数字、下划线，大小写字母或下划线开头|
|params.default|string|是|默认值，命名规则：0-64个字符|

!> 注1：scheduler.type 如果是depend 模式，代表这个离线任务依赖某个上游的离线任务。首先srcs内有且仅有一个离线任务数据源。同时该任务不能指定调度的模式、魔法变量和容器规格。这些全部使用上游依赖的离线任务。

!> 注2：srcs.tableName在此任务的srcs中以及依赖的所有上游任务的srcs中不能重复。


### 更新离线计算任务

**请求语法** 

```
PUT /v2/jobs/<JobName>
Content-Type: application/json
Authorization: Pandora <auth>
{
	"srcs":[
		{
			"name":<DataSourceName|JobName>,
			"fileFilter":<fileFilter>,
			"type":<DataSource|Job>,
			"tableName": <TableName>
		},
		...
	],
   "computation": {
       "code": <Code>,
       "type": <SQL>
   }, 
   "container": {
       "type": <ContainerType>,
       "count": <ContainerCount>
   },
   "scheduler":{
       "type": <crontab|loop|manual|depend>,
       "spec": {
           "crontab": <0 0 0/1 * * ?>,
           "loop": <1h|3m|....>
       }
   },
   "params":[
       {
           "name":<ParamName>,
           "default":<ParamValue>
       },
       ...
   ]
}
```

!> 注意：更新时候 srcs, code, container, scheduler, params 校验逻辑和创建的时候相同。下游计算任务不指定container, scheduler, params。更新逻辑为全量更新。需要提前获取所有信息。



### 列举离线计算任务信息

**请求语法** 

```
GET /v2/jobs?srcDatasource=[DataSourceName]&srcJob=[JobName]
Authorization: Pandora <auth>

```

**响应报文** 

```
{
	"jobs":[
		{
		    "name": <JobName>,
			"srcs":[
				{
					"name":<DataSourceName|JobName>,
					"fileFilter":<fileFilter>,
					"type":<DataSource|Job>,
					"tableName": <TableName>
				},
				...
			],
		   "scheduler":{
		       "type": <crontab|loop|manual|depend>,
		       "spec": {
		           "crontab": <0 0 0/1 * * ?>,
		           "loop": <1h|3m|....>
		       }
		   },
		   "computation": {
		       "code": <Code>,
		       "type": <SQL>
		   }, 
		   "container": {
		       "type": <ContainerType>, 
		       "count": <ContainerCount>
		   },
		   "params":[
		       {
		           "name":<ParamName>,
		           "default":<ParamValue>
		       },
		       ...
		   ]，
		   "schema": [
		       {
		           "key": <Key>,
		           "valtype": <ValueType>
		       },
		       ...
		   ]
		},
		...
	]
}
```

**请求内容** 


|名称|类型|必填|描述|
|:---|:---|:---|:---|
|srcDatasource|string|否|依赖离线数据源名字|
|srcJob|string|否|依赖离线计算任务名字|

!> 注意: srcDataSource和srcJob不能同时存在。当两个参数都不指定的时候列举出所有的离线计算任务。


### 获取单个离线计算任务信息

**请求语法** 

```
GET /v2/jobs/<JobName>
Content-Type: application/json
Authorization: Pandora <auth>

```

**响应报文** 

```
{
	"srcs":[
		{
			"name":<DataSourceName|JobName>,
			"fileFilter":<fileFilter>,
			"type":<DataSource|Job>,
			"tableName": <TableName>
		},
		...
	],
   "scheduler":{
       "type": <crontab|loop|manual|depend>,
       "spec": {
           "crontab": <0 0 0/1 * * ?>,
           "loop": <1h|3m|....> 
		  } 
	},
   "computation": {
       "code": <SqlCode>,
       "type": <SQL>
   }, 
   "container": {
       "type": <ContainerType>, 
       "count": <ContainerCount>
   },
   "params":[
   		{
   			"name":<ParamName>,
   			"default":<ParamValue>
   		},
   		...
   ]，
   "schema": [
      {
        "key": <Key>,
        "valtype": <ValueType>
      },
      ...
    ]
}
```



### 启动离线计算

**请求语法**

```
POST /v2/jobs/<JobName>/actions/start
Authorization: Pandora <auth>
{
	"params":[
   		{
   			"name":<ParamName>,
   			"value":<ParamValue>
   		},
   		...
   ]
}
```

**请求内容**

|名称|类型|必填|描述|
|:---|:---|:---|:---|
| params |array|否|定义运行时的魔法参数值|
| params.name |string|是|魔法变量名字|
| params.value |string|是|运行时魔法变量值|



### 停止离线计算

**请求语法**

```
POST /v2/jobs/<JobName>/actions/stop
Authorization: Pandora <auth>
```


### 删除离线计算任务信息

**请求语法** 

```
DELETE /v2/jobs/<JobName>
Authorization: Pandora <auth>
```


### 获取数据源schema

**请求语法** 

```
POST /v2/schemas
Content-Type: application/json
Authorization: Pandora <auth>
{
	"type": <Kodo|HDFS>, 
	"spec": <Spec>
}
```

**请求内容**

Type为Kodo时，Spec结构：

|名称|类型|必填|描述|
|:---|:---|:---|:---|
| spec.bucket |string|是|对象存储的存储空间|
| spec.keyPrefixes |array|否|一个或者多个文件前缀|
| spec.fileType |string|是|文件类型，合法取值为`json`, `parquet`, `text`|

Type为HDFS时，Spec结构：

|名称|类型|必填|描述|
|:---|:---|:---|:---|
| spec.paths |array|是|一个或者多个文件路径|
| spec.fileType |string|是|文件类型，合法取值为`json`, `parquet`, `text`|


**响应内容** 

```
json or parquet return :
{
	"schema": [
		{
			"key": <Key>,
			"valtype": <ValueType>
		},
		...
	]
}

text return :
{
	"schema": [
		{
			"key": <text>,
			"valtype": <String>
		},
		...
	]
}

```

### 创建离线计算导出任务-云存储

**请求语法**

```
POST /v2/jobs/<JobName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "type": <kodo>,
    "spec": {
         "bucket": <Bucket>,
         "keyPrefix": <Prefix|Path>,
         "format": <ExportFormat>,
         "compression": <compression>,
         "retention": <Retention>,
         "partitionBy": <PartitionBy>，
         "fileCount": <FileCount>,
         "saveMode": <SaveMode>
	}
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|type|string|是|导出的类型，目前允许的值为"kodo"|
|bucket|string|是|对象存储bucket名称|
|keyPrefix|string|否|导出的文件名的前缀，当离线任务的`scheduler`是`manual`的时候，就是文件名</br>命名规则：0-128个字符，不支持英文 `:` 、`\`、`<`、`>`符号|
|format|string|否|文件导出格式,支持`json`、`csv`、`text`、`orc`、`parquet`五种类型|
|compression|string|否|压缩类型, 具体支持类型与`format`值相关，详见`注1`|
|retention|int|否|数据储存时限,以天为单位,当不大于0或该字段为空时,则永久储存|
|partitionBy|array|否|指定作为分区的字段，为一个字符数组，合法的元素值是字段名称|
|fileCount|int|是|计算结果导出的文件数量，应当大于0，小于等于1000|
|saveMode|string|否|计算结果的保存模式：overwrite(默认) 文件已经存在则覆盖掉， append 在已有的文件上追加，errorIfExists 文件已经存在的时候报错误，ignore 文件已经存在 则认为跑成功了，不报错|

!> 注1: 当用户指定`format`为`json`、`csv`或`text`时, `compression`仅支持`none`(不压缩)、`bzip2`, `gzip`, `lz4`, `snappy`和`deflate`; 当用户指定`format`为`orc`时, `compression`仅支持`none`(不压缩)、`snappy`, `zlib`和`lzo`; 当用户指定`format`为`parquet`时, `compression`仅支持`none`(不压缩)、`snappy`, `gzip`和`lzo`。

!> 注2: `keyPrefix`字段表示导出文件名称的前缀,该字段可选,默认值为""(生成文件名会自动加上时间戳格式为`yyyy-MM-dd-HH-mm-ss`),如果使用了一个或者多个魔法变量时不会自动添加时间戳,支持魔法变量,采用`$(var)`的形式求值,目前可用的魔法变量var如下:

* `year` 上传时的年份
* `mon` 上传时的月份
* `day` 上传时的日期
* `hour` 上传时的小时
* `min` 上传时的分钟
* `sec` 上传时的秒钟


### 创建离线计算导出任务-HDFS

**请求语法**

```
POST /v2/jobs/<JobName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "type": <hdfs>,
    "spec": {
         "path": <path_to_write>,
         "format": <ExportFormat>,
         "compression": <compression>,
         "partitionBy": <PartitionBy>，
         "fileCount": <FileCount>,
         "saveMode": <SaveMode>
	}
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|type|string|是|导出的类型，目前允许的值为"kodo"|
|namespace|string|是|hdfs的uri，如：hdfs://host:9000|
|path|string|是|导出的路径。命名规则：0-128个字符，不支持英文 `:` 、`\`、`<`、`>`符号|
|format|string|否|文件导出格式,支持,`json`、`csv`、`text`、`orc`、`parquet`五种类型|
|compression|string|否|压缩类型, 具体支持类型与`format`值相关，详见`注1`|
|partitionBy|array|否|指定作为分区的字段，为一个字符数组，合法的元素值是字段名称|
|fileCount|int|是|计算结果导出的文件数量，应当大于0，小于等于1000|
|saveMode|string|否|计算结果的保存模式：overwrite(默认) 文件已经存在则覆盖掉， append 在已有的文件上追加，errorIfExists 文件已经存在的时候报错误，ignore 文件已经存在 则认为跑成功了，不报错|

!> 注1: 当用户指定`format`为`json`、`csv`或`text`时, `compression`仅支持`none`(不压缩)、`bzip2`, `gzip`, `lz4`, `snappy`和`deflate`; 当用户指定`format`为`orc`时, `compression`仅支持`none`(不压缩)、`snappy`, `zlib`和`lzo`; 当用户指定`format`为`parquet`时, `compression`仅支持`none`(不压缩)、`snappy`, `gzip`和`lzo`。

!> 注2: `keyPrefix`字段表示导出文件名称的前缀,该字段可选,默认值为""(生成文件名会自动加上时间戳格式为`yyyy-MM-dd-HH-mm-ss`),如果使用了一个或者多个魔法变量时不会自动添加时间戳,支持魔法变量,采用`$(var)`的形式求值,目前可用的魔法变量var如下:

* `year` 上传时的年份
* `mon` 上传时的月份
* `day` 上传时的日期
* `hour` 上传时的小时
* `min` 上传时的分钟
* `sec` 上传时的秒钟


### 更新离线计算导出任务

**请求语法**

```
PUT /v2/jobs/<JobName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "spec": <Spec>
}
```


### 按照名称查看离线计算导出任务

**请求语法**

```
GET /v2/jobs/<JobName>/exports/<ExportName>
Authorization: Pandora <auth>
```

**响应报文**

```
{
    "type": <Type>,
    "spec": <Spec>
}
```


### 列举离线计算导出任务

**请求语法**

```
GET /v2/jobs/<JobName>/exports
Authorization: Pandora <auth>
```

**响应报文**

```
Content-Type: application/json
{
  "exports": [
    {
      "name": <ExportName>,
      "type": <Type>,
      "spec": <Spec>
    },
    ...
  ]
}
```


### 按照名称删除离线计算导出任务

**请求语法**

```
DELETE /v2/jobs/<JobName>/exports/<ExportName>
Authorization: Pandora <auth>
```


### 查看历史任务

**请求语法**

```
GET /v2/jobs/<jobName>/history?page=1&size=20&sortBy=endTime&order=desc&status=xx&runId=-1
Content-Type: application/json
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|page|int|否|分页页数|
|size |int|否|分页大小|
|sortBy |string|否|根据哪个字段排序|
|order |string|否|排序asc，desc|
|status |string|否|<Ready、Successful、Failed、Running、Canceled>|
|runId |int|否|查询某个运行批次的状态，-1代表最近一次|


**响应报文**

```
{
"total": <TotalCnt>, # 总共运行批次
"history":[
	{
		"runId" : <RunId>, # 运行批次
		"schedTime": "<SchedTime>",
		"startTime": <StartTime>,
		"endTime": <EndTime>,
		"duration": <Duration>,
		"status": <Ready、Successful、Failed、Running、Canceled>,
		"message": <Message>
	},
  ]
}
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|total|int|是|总运行批次次数|
|history|array|是|运行历史|
|history.runId|int|是|运行批次|
|history.schedTime|string|是|调度时间|
|history.startTime |string|否|启动时间|
|history.endTime |string|否|终止事件，如果为Running，则为当前时间|
|history.duration |int|否|批次运行时间，单位秒|
|history.status |string|否|<Ready、Successful、Failed、Running、Canceled>|
|history.message |string|否|运行、出错信息，比如运行成功、内存溢出、数据损坏|

### 停止批次任务

**请求语法**

```
POST /v2/batch/actions/stop
Content-Type: application/json
Authorization: Pandora <auth>
{
    "jobName": "<jobName>",
    "runId": "<runId>"
}

```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|jobName|string|是|job名称|
|runId|string|是|待操作的运行批次ID|


**响应报文**

```
{
  "jobName": "jobName",
  "runId": "runId",
  "preStatus": "Running",
  "postStatus": "Cancelled"
}
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|jobName|string|是|job名称|
|runId|string|是|操作job的RunId|
|preStatus|string|是|停止前状态|
|postStatus|string|是|停止后状态|

**说明**

|状态|说明|
|:---|:---|
|Ready| 不允许重跑操作，允许停止|
|Success| 不允许停止操作，允许重跑|
|Failed| 不允许停止操作，允许重跑|
|Running| 允许重跑及停止操作（需先停止后重跑）|
|Cancelld| 不允许停止操作，允许重跑|


### 重跑批次任务

**请求语法**

```
POST /v2/batch/actions/rerun
Content-Type: application/json
Authorization: Pandora <auth>
{
    "jobName": "<jobName>",
    "runId": "<runId>"
}

```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|jobName|string|是|job名称|
|runId|string|是|待操作的运行批次ID|


**响应报文**

```
{
  "jobName": "jobName",
  "runId": "runId",
  "preStatus": "Failed",
  "postStatus": "Running",
  "rerunCount": 2
}
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|jobName|string|是|job名称|
|runId|string|是|操作job的RunId|
|preStatus|string|是|重跑前状态|
|postStatus|string|是|重跑后状态|
|rerunCount|int|是|重跑次数|

**说明**

|状态|说明|
|:---|:---|
|Ready| 不允许重跑操作，允许停止|
|Success| 不允许停止操作，允许重跑|
|Failed| 不允许停止操作，允许重跑|
|Running| 允许重跑及停止操作（需先停止后重跑）|
|Cancelld| 不允许停止操作，允许重跑|


## UDF

### 上传UDF jar包

**请求语法**：

```
POST /v2/udf/jars/<JarName>
Content-Type: application/java-archive
Content-MD5: <ContentMD5>
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|JarName |string|是|jar包名称唯一标识一个jar文件，命名规则: `^[a-zA-Z][a-zA-Z0-9_\\.]{0,127}[a-zA-Z0-9_]$`,1-128个字符,支持小写字母、数字、下划线；必须以大小写字母开头|
|Content-MD5 |string|是|jar包的MD5码|

**UDF Jar包说明:**

+ 上传的UDF Jar包最大为100MB。

+ Content-MD5头部是可选的。如果上传UDF Jar包的时候带上该头部服务器会校验上传数据的校验和,如果两者不一致服务器将拒绝上传。如果不带该头部,服务器不做任何校验和的检查。

+ 先计算UDF Jar内容的MD5,再对MD5做一次base64编码转化为字符串。例如qiniu这个字符串的Content-MD5是gLL29S04bTCxYd2kCqsEIQ==而不是7b9d6b4d89f6825a196d4cc50fdbedc5


**响应报文**

```
200 OK
Content-Type: application/json
{
    "jarName":"UdfCollection",
    "jarFile":"udf-agg-1.0-SNAPSHOT.jar",
    "uploadTime": "2017-06-06 15:15:03"
}
```

**响应内容**

|参数|类型|说明|
|:---|:---|:---|
|jarName|string|jar包名称|
|jarFile|string|jar包文件名|
|uploadTime |string|jar包的上传时间|

**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/udf/jars/UdfAggCollection \
-H 'Content-Type: application/java-archive' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-T /path/to/udf-agg-1.0-SNAPSHOT.jar
```

### 删除UDF jar包

**请求语法**：

```
DELETE /v2/udf/jars/<JarName>
Content-Type: application/json
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|jarName |string|是|jar包名称,如果UdfAggCollection|

**示例**

```
curl -X DELETE https://pipeline.qiniu.com/v2/jars/UdfAggCollection \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' 
```

**响应报文**

```
200 OK
Content-Type: application/json
```

or

```
400 BadRequest
Content-Type: application/json
{
    "error":"invalid jar name"
}
```

or

```
404 NotFound
Content-Type: application/json
{
    "error":"jar file does not exist"
}
```

### 获取UDF jar包列表

**请求语法**：

```
GET /v2/udf/jars?page=1&size=5&sortBy=uploadTime&order=desc
Content-Type: application/json
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|jar |string|是|jar包名称|
|descrption|string|是|jar包描述|
|uploadTime |string|是|jar包上传时间|

**示例**

```
curl -X GET https://pipeline.qiniu.com/v2/udf/jars?page=1&size=2&sortBy=uploadTime&order=desc \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='
```

**响应报文**

```
200 OK
Content-Type: application/json
[
    {
        "jar":"udf-agg.jar",
        "descrption":"聚合类UDF集合",
        "uploadTime": "2017-06-01 15:00:00"
    },
    {
        "jar":"udf-sort.jar",
        "descrption":"排序类UDF集合",
        "uploadTime": "2017-06-01 14:00:00"
    }
]
```

### UDF函数注册

**请求语法**：

```
POST /v2/udf/register
Content-Type: application/json
Authorization: Pandora <auth>
{
    "funcName": "<UDF function name>",
    "className": "<Full path class name>",
    "jar": "<JarFileName>",
    "signature": "<UDF Signature>",
    "description": "<UDF description>",
    "category": "<UDF category>"
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|funcName|string|是|udf的自定义函数名称（主键），如sum,avg|
|className |string|是|自定义udf的全路径名称com.company.biz.udf.SUM|
|jar|string|是|udf包名称，从中解析出UDF类|
|signature |string|是|函数的签名，如：DOUBLE abs(DOUBLE a) |
|description|string|否|自定义的函数描述|
|category |string|否|函数分类，如日期函数，数学函数等。默认支持：match（数学函数），collection（集合函数），type_conversion（类型转换函数），date（日期函数），condition（条件函数）, string（字符串函数），udaf（聚合函数），udtf(表格函数)|


**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/udf/register \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
    "funcName": "str_len",
    "className": "com.qiniu.udf.StrLength",
    "jar":"udf-common.jar",
    "signature": "int str_len(string A)",
    "description": "计算字符串长度",
    "category": "string"
}'
```

**响应报文**

```
200 OK
Content-Type: application/json
```

or

```
400 BadRequest
Content-Type: application/json
{
    "error":"funcName already exists"
}
```

### UDF函数注册删除

**请求语法**：

```
POST /v2/udf/deregister
Content-Type: application/json
Authorization: Pandora <auth>
{
    "funcName":<UDF Function Name>
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|funcName|string|是|udf的自定义函数名称（主键），如sum,avg|

**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/udf/deregister \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
    "funcName": "str_len"
}'
```

**响应报文**

```
200 OK
Content-Type: application/json
```

or

```
400 BadRequest
Content-Type: application/json
{
    "error":"funcName does not exist"
}
```

### UDF调试

**请求语法**：

```
POST /v2/udf/debug
Content-Type: application/json
Authorization: Pandora <auth>
{
    "funcName":<UDF function name>,
    "udfExpression": "<UDF debug expression>"
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|funcName|string|是|udf的自定义函数名称（主键），如sum,avg|
|udfExpression |string|是|要调试的UDF的表达式，来验证udf的输出是否符合预期，如str_lenth("debug_udf"）|

**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/udf/debug \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
    "funcName": "str_len",
    "udfExpression": "str_len("abcd") as a1, str_len("ab") as a2"
}'
```

**响应报文**

```
200 OK
Content-Type: application/json
{
    result:[
        {"a1":4, "a2": 2}
    ]
}
```

or

```
400 BadRequest
Content-Type: application/json
{
    "error":"invalid udf debug expression"
}
```

**说明**

UDF调试调试表达式必须符合 `udf1(param...) as column2, udf2(param...) as column2` 格式



### UDF内置函数展示

**请求语法**：

```
GET /v2/udf/builtins?page=1&size=2&sortBy=funcName&order=desc
Content-Type: application/json
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|funcName|string|是|udf的内置函数名称(主键)，如sum, avg|
|signature |string|是|内置函数的签名，如：DOUBLE abs(DOUBLE a)|
|description|string|否|内置的函数描述|
|category |string|否|函数分类，如日期函数，数学函数等。默认支持：match（数学函数），collection（集合函数），type_conversion（类型转换函数），date（日期函数），condition（条件函数）, string（字符串函数），udaf（聚合函数），udtf(表格函数)|


**示例**

```
curl -X GET https://pipeline.qiniu.com/v2/udf/builtins?page=1&size=2&sortBy=funcName&order=desc&category=math \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
```

**响应报文**

```
200 OK
Content-Type: application/json
[
    {
        "funcName": "abs",
        "signature": "DOUBLE abs(DOUBLE a)",
        "description": "绝对值计算",
        "category": "math"
    },
    {
        "funcName":"cos",
        "signature": "DOUBLE cos(DOUBLE a)",
        "description": "余弦值计算DOUBLE cos(DOUBLE a)",
        "category": "math"
    }
    ...
]
```



### 错误代码及相关说明

| 错误码 | 错误描述 |
| :---  | :----- |
|500	|服务器内部错误 |
|411	|E18003: 缺少内容长度|
|400	|E18004: 无效的内容长度|
|413	|E18005: 请求实体过大|
|400	|E18006: 请求实体为空|
|400	|E18007: 请求实体格式非法|
|400	|E18008: 字段长度超过限制|
|409	|E18101: 仓库已经存在|
|404	|E18102: 仓库不存在|
|400	|E18103: 无效的仓库名称|
|400	|E18104: 无效的日期格式|
|400	|E18105: 仓库Schema为空|
|400	|E18106: 无效的字段名称|
|400	|E18107: 不支持的字段类型|
|400	|E18108: 源仓库数量超过限制|
|400	|E18109: 无效的仓库模式|
|400	|E18110: 无效的字段格式|
|404	|E18111: 字段不存在|
|400	|E18112: 仓库上存在着级联的转换任务或者导出任务|
|409	|E18113: 仓库处于删除状态中|
|400	|E18117: Plugin名称不合法|
|404	|E18120: 共享资源池不存在|
|404	|E18122: 导出的仓库在logd中不存在|
|202	|E18124: 仓库处于创建中|
|400	|E18125: 读取gzip的打点请求体出错|
|409	|E18201: 计算任务已经存在|
|404	|E18202: 计算任务不存在|
|415	|E18203: 计算任务类型不支持|
|409	|E18204: 计算任务的源仓库与目的仓库相同|
|409	|E18205: 目的仓库已经被其他转换任务占用|
|400	|E18206: 目的仓库必须通过删除计算任务的方式删除|
|400	|E18207: 计算任务描述格式非法|
|400	|E18208: 计算任务interval非法|
|400	|E18209: 计算任务中的SQL语句非法|
|400	|E18211: 计算任务中plugin输出字段类型非法|
|400	|E18212: 仓库的区域信息和数据中心不相符|
|400	|E18213: 计算任务中容器类型非法|
|400	|E18214: 计算任务中容器数量非法|
|409	|E18215: 共享资源池处于使用中|
|404	|E18216: Plugin不存在|
|409	|E18217: Plugin已存在|
|409	|E18218: 共享资源池已存在|
|400	|E18219: Plugin上传内容长度不一致|
|400	|E18220: Plugin上传内容的MD5不一致|
|400	|E18221: 共享资源池名称非法|
|400	|E18222: 共享资源池的区域信息不一致|
|400	|E18223: 不能向计算任务的目标仓库中打点|
|409	|E18301: 导出任务已经存在|
|404	|E18302: 导出任务不存在|
|400	|E18303: 提交导出任务失败|
|400	|E18304: 删除导出任务失败|
|400	|E18305: 导出任务出现错误|







