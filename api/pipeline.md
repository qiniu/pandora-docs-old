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

### 创建资源池
**请求语法**
```
POST /v2/groups/<GroupName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "region": <Region>, 
    "container": {
      "type":  <ContainerType>,
      "count": <ContainerCount>
    },
    "allocateOnStart": <AllocateOnStart>
}```
**请求内容**
|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|GroupName|string|是|资源池名称,用来标识该资源池的唯一性；</br>命名规则:`^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`,1-128个字符,支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
|region|string|是|所属区域,计算与存储所使用的物理资源所在区域,目前支持华东(`nb`)；</br>此参数是为了降低用户传输数据的成本,应当尽量选择离自己数据源较近的区域|
|container|map|是|计算资源的数量及类型|
| container.type|string|是|资源类型,目前支持`M16C4`和`M32C8`分别代表`4核(CPU)16G(内存)`和`8核(CPU)32G(内存)`|
| container.count|int|是|指资源`type`的数量,最小为1,最大为128|
|allocateOnStart|bool|否|`true`代表在创建资源池的时候就分配资源,</br>`false`代表在该资源池中第一个计算任务启动时,再分配资源；</br>  默认为`false`|**示例**
```
curl -X POST https://pipeline.qiniu.com/v2/groups/test_group \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
    "region": "nb", 
    "container": {
       "type":  "M16C4",
       "count": 5
    },
    "allocateOnStart": true
}' ```
### 修改资源池

**请求语法**
```
PUT /v2/groups/<GroupName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "container": {
      "type":  <ContainerType>,
      "count": <ContainerCount>
    }
}```**示例**
```
curl -X POST https://pipeline.qiniu.com/v2/groups/test_group \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='  \
-d '{
    "container": {
       "type":  "M16C4",
       "count": 5
    }
}' ```### 启动/停止资源池

**请求语法**

```POST /v2/groups/<GroupName>/actions/start|stop
Authorization: Pandora <auth>
```

**启动资源池示例**

```
curl -X POST https://pipeline.qiniu.com/v2/groups/test_group/actions/start \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='
```

**停止资源池示例**

```
curl -X POST https://pipeline.qiniu.com/v2/groups/test_group/actions/stop \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='
```

### 查询所有资源池

**请求语法**```
GET /v2/groups 
Authorization: Pandora <auth>```
**响应报文** 
```
Content-Type: application/json
{
    "groups": [
      {
        "name": <GroupName>,
        "region": <Region>,
        "container": {
          "type": <ContainerType>,
          "count": <ContainerCount>
        }
      },
      ...
    ]
}```
### 根据名称查询资源池

**请求语法**

```
GET /v2/groups/<GroupName>
Authorization: Pandora <auth>
```

**响应报文** 

```
{
    "region": <Region>,
    "container": {
      "type": <ContainerType>,
      "count": <ContainerCount>,
      "status": <ContainerStatus>
    },
    "createTime": <CreateTime>,
    "updateTime": <UpdateTime>
  }```
**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| container.status|string|-|运行状态,目前有`running`,`idle`两种,表示运行中和空闲|
|createTime|string|-|创建时间|
|updateTime|string|-|更新时间|

### 根据名称删除资源池

**请求语法**```
DELETE /v2/groups/<GroupName>
Authorization: Pandora <auth>```
### 创建消息队列
**请求语法**
```
POST /v2/repos/<RepoName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "region": <Region>,
    "group": <GroupName>,
    "schema": [
      {
        "key": <Key>,
        "valtype": <ValueType>,
        "elemtype": <ElemType>,
        "required": <Required>,
        "schema": [
            ...
        ]
      },
      ...
    ]
}
```
**请求内容**
|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|RepoName|string|是|消息队列名称,用来标识该消息队列的唯一性；</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`,1-128个字符,支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
|region|string|是|所属区域,计算与存储所使用的物理资源所在区域,目前支持华东(代号`nb`)；</br>此参数是为了降低用户传输数据的成本；应当尽量选择离自己数据源较近的区域|
|group|string|否|指定该消息队列所属的资源池,如果不填写,则说明独享消息队列资源|
|schema|array|是|相当于关系型数据库表中的`字段集`，当valtype取值为map时，该参数需要嵌套，且嵌套深度最大为5|
| schema.key|string|是|相当于关系型数据库表的`字段`,</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`,1-128个字符,支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
| schema.valtype|string|是|描述`key`字段的数据类型,目前仅支持`long`、`date`、`float`、`string`、`array`和`map`,</br>其中`float`的最大精度是`float64`；`array`表示数组类型，数组元素必须为同一类型；`map`表示嵌套类型，类似于json object|
| schema.elemtype|string|否|当数据类型为`array`时，该参数必填，否则将其忽略。该参数表示`array`的元素类型，目前仅支持`long`、`float`、`string`|
| schema.required|bool|否|描述用户在传输数据时`key`字段是否必填|

**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/repos/Test_Repo \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='  \
-d '{
    "region": "nb", 
    "group":"test_group",
    "schema": [
        {
            "key": "userName",
            "valtype": "string",
            "required": true
        },
        {
            "key": "age",
            "valtype": "float",
            "required": true
        },
        {
            "key": "addresses",
            "valtype": "array",
            "elemtype": "long",
            "required": true
        },
        {
            "key": "profile",
            "valtype": "map",
            "required": true,
            "schema": [
              {
                  "key": "position",
                  "valtype": "string",
                  "required": true
              },
              {
                  "key": "salary",
                  "valtype": "float",
                  "required": true
              },
              {
                  "key": "education",
                  "valtype": "array",
                  "elemtype": "string",
                  "required": false
              }
            ]
        }
    ]
}'
```
### 查看所有消息队列
**请求语法**

```GET /v2/reposAuthorization: Pandora <auth>
```
**响应报文** 

```Content-Type: application/json{
    "repos": [
      {
        "name": <RepoName>,
        "region": <Region>,
        "derivedFrom": <TransformName>,
        "group": <GroupName>
      },
      ...
    ]
}```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|derivedFrom|string|-|表示这个repo是由哪个transform生成的,如果此项为空,说明该repo是由用户自行创建的|

### 根据名称查询消息队列
**请求语法**

```GET /v2/repos/<RepoName>Authorization: Pandora <auth>
```
**响应报文** 

```
Content-Type: application/json
{
    "region": <Region>,
    "group": <GroupName>,
    "derivedFrom": <TransformName>,
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
### 根据名称删除消息队列
**请求语法**

```DELETE /v2/repos/<RepoName>Authorization: Pandora <auth>
```### 根据名称更新消息队列

**请求语法**

```
PUT /v2/repos/<RepoName>
Content-Type: application/json
Authorization: Pandora <auth>

{
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
**示例**
```
curl -X PUT https://pipeline.qiniu.com/v2/repos/test_repo \
	 -H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='
	 
{
	"schema": [
	  {
	    "key": "ip",
	    "valtype": "String",
	    "required": flase
	  },
	  ...
	]
}```### 数据推送
**请求语法**
```POST /v2/repos/<RepoName>/dataContent-Type: application/textAuthorization: Pandora <auth>keyName=valName<TAB>keyName=valName ...keyName=valName<TAB>keyName=valName ......
```
**请求内容**|参数|类型|必填|说明|
|:---|:---|:---|:---|
| RepoName |string|  是  |消息队列的名称|
| keyName |string|是|消息队列的`schema`中`key`的值,即字段名称|
| valName |string|是|对应key的数据内容<br/> 注意:如果是`string`类型,那么 `\t`、`\r`、`\n` `\` 需要用`\`转义,空格`' '` 可以不转义|

!> 注意:对于`array`类型，打点格式为`[e1,e2,...,en]`，数组元素采用逗号分割，且所有元素使用`[]`包括，当元素类型为`string`时，需要加上双引号;对于`map`类型，打点格式为json字符串，比如`{"f1":123,"f2":"abc"}`，注意所有元素使用`{}`包括;另外，多个`keyName`和`valName`之间应使用单个 `<TAB>` 分隔,单次分隔的长度不超过100KB**示例**```curl -X POST https://pipeline.qiniu.com/v2/repos/test_Repo/data \-H 'Content-Type: application/text' \-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \-d '{
	userName=小张		age=22   addresses=["beijing","shanghai"] profile={"position":"engineer",salary:15000} 	userName=小王		age=28   addresses=["hangzhou","shenzhen"] profile={"position":"engineer",salary:12000}
	}'```
### 创建计算任务
**请求语法**

```POST /v2/repos/<RepoName>/transforms/<TransformName>/to/<DestinationRepoName>Content-Type: application/jsonAuthorization: Pandora <auth>{
  "plugin": {
    "name": <PluginName>,
    "output": [
      {
        "name": <FieldName1>,
        "type": <FieldType>
      },
      {
        "name": <FieldName2>
      },
      ......
    ],
  },  "mode": <Mode>,  "code": <Code>,  "interval": <Interval>,  "container": {
       "type": <ContainerType>,
       "count": <ContainerCount>
  }}
```
**请求内容**|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| RepoName |string|是|指定一个消息队列的名称|
| TransformName |string|是|为这个计算任务取个名字,用来标识该消息队列的唯一性;</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`,1-128个字符,支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
| DestinationRepoName |string|是|计算结果目标消息队列,如果该消息队列不存在,将自动创建一个|
| plugin |json|否|自定义计算|
| name |string|是|plugin名称|
| output |json|是|输出数组,即这个plugin计算完成后,输出的数据结果的结构和类型,</br>也可以理解为一张表,包含字段名称和字段类型|
|output.name|string|是|输出字段名称,</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`,1-128个字符,支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
|output.type|string|否|输出字段类型,支持`string`、`long`和`float`三种,</br>时间类型使用`long`类型,默认为`string`类型|
| mode |string|否|该计算任务使用的语言类型,目前仅支持`sql`|
| code |string|否|`sql`语句代码|
| interval |string|否|计算任务的运行时间间隔,目前支持`5s`、`10s`、`20s`、`30s`、`1m`、`5m`和`10m`的粒度,如果不指定,系统默认使用`1m`|
|container|map|否|计算资源的数量及类型|
|type|string|否|目前支持`M16C4`和`M32C8`分别代表`4核(CPU)16G(内存)`和`8核(CPU)32G(内存)`|
|count|int|否|指资源`type`的数量,最小为1,没有上限|

!> 注意:`mode`加`code`是基础的数据计算方式,自定义计算(plugin)是更为高级的数据计算方式,要注意`mode/code`和自定义计算两种计算方式可以共存,但不可以一种都不指定。当自定义计算和`mode/code`共存时,系统优先执行自定义计算,后执行mode/code。



**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/repos/test_repo/transforms/transform_job/to/compute_repo \	 -H 'Content-Type: application/json' \	 -H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \	 -d {  		"mode": "sql",  		"code": "select count(*) from test_repo",  		"interval": "1m",  		"container": {
       		"type": "M16C4",
       		"count": 5
  			}		}
```

### 编写自定义计算

### 上传jar包(自定义计算使用)

**请求语法**

```
POST /v2/plugins/<PluginName>
Content-Type: application/java-archive
Content-MD5: <ContentMD5>
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---|:---|
|PluginName|string|是|plugin名称,注:该名称必须与待上传jar包中对应Parser类的全限定名保持一致,用来标识该消息队列的唯一性;命名规则: `^[a-zA-Z][a-zA-Z0-9_\\.]{0,127}[a-zA-Z0-9_]$`,1-128个字符,支持小写字母、数字、下划线；必须以大小写字母开头|
|ContentMD5|string|是|MD5码|

**Plugin说明:**

* 上传的Plugin Jar包最大为20MB。
* Content-MD5头部是可选的。如果上传plugin的时候带上该头部服务器会校验上传数据的校验和,如果两者不一致服务器将拒绝上传。如果不带该头部,服务器不做任何校验和的检查。
* <ContentMD5>是先计算plugin内容的MD5,再对MD5做一次base64编码转化为字符串。例如qiniu这个字符串的Content-MD5是gLL29S04bTCxYd2kCqsEIQ==而不是7b9d6b4d89f6825a196d4cc50fdbedc5
* PluginName必须与用户所编写的Parser类的全限定名保持一致,否则transform执行plugin会失败。 例如NginxLogParser位于com.qiniu包,PluginName须写为com.qiniu.NginxLogParser。

**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/plugins/ComputeSumDataParser \
-H 'Content-Type: application/java-archive'  \
-H 'Content-MD5: 900150983cd24fb0d6963f7d28e17f72'  \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-T ./plugins-1.0-SNAPSHOT.jar \  
```

### 查看所有计算任务

**请求语法**

```
GET /v2/repos/<RepoName>/transforms
Authorization: Pandora <auth>
```

### 查看指定计算任务的信息

**请求语法**

```
GET /v2/repos/<RepoName>/transforms/<TransformName>
Authorization: Pandora <auth>
```

**响应报文** 

```
200 OK
Transform-Type: application/<TransformType>
{
    "name": "<TransformName1>",
    "to": "<DestRepo1>",
    "spec": {
      "plugin": {
        "name": <PluginName>,
        "output": [
          {
            "name": <FieldName1>,
            "type": <FieldType>
          },
          {
            "name": <FieldName2>
          },
          ...
        ],
      },
      "mode": <Mode>,
      "code": <Code>,
      "interval": <Interval>
    }
}
```

**示例**

```
curl -G https://pipeline.qiniu.com/v2/repos/test_repo/transforms/test_transform \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='
```### 修改计算任务

**请求语法**

```
PUT /v2/repos/<RepoName>/transforms/<TransformName>
Content-Type: application/json
Authorization: Pandora <auth>
{
	"plugin": {
      "name": <PluginName>,
      "output": [
        {
          "name": <FieldName1>,
          "type": <FieldType>
        },
        {
          "name": <FieldName2>
        },
        ...
      ],
    },
    "code": <Code>,
    "interval": <Interval>,
    "container": {
      "type":  <ContainerType>,
      "count": <ContainerCount>
    }
}
```

**请求内容**
|参数|类型|必填|说明|
|:---|:---|:---|:---|
|plugin|json|否|自定义计算|
|code|string|否|sql语句|
|interval|string|否|更改计算任务运行时间间隔,目前支持`1m`、`5m`和`10m`|
|container|json|否|更改计算任务配额的类型和个数,类型支持`M16C4`（4核16G内存）、`M32C8`（8核32G内存）,数量最小为1,没有上限|

>**注意:** 
>
>1.更新计算任务,可以同时更新`plugin`、`sql代码`、`运行时间间隔`和`配额`,也可以只更新其中一种,但不能一种都不指定。

>2.在更新`sql`和`plugin的输出字段`时,只能添加新的字段,不能删除和更改已经存在的字段。
### 删除计算任务
**请求语法**
```DELETE /v2/repos/<RepoName>/transforms/<TransformName>
Authorization: Pandora <auth>
```
### 导出数据至HTTP地址
**请求语法**```POST /v2/repos/<RepoName>/exports/<ExportName>Content-Type: application/jsonAuthorization: Pandora <auth>{  "type": <http>,  "whence": <ExportWhence>,  "spec": {
		"host": <Host>,      
		"uri": <RequestURI>  
	}}
```
**请求内容**
|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| RepoName |string|是|需要导出数据的消息队列名称|
| ExportName |string|是|为这个导出任务起一个名字,</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`,1-128个字符,支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
| Type |string|是|导出方式,目前支持`http`、`logDB`、`mongoDB`、`tsdb`、`kodo`,</br>在这里我们选择`http`|
|whence|string|否|导出数据的起始位置,目前支持`oldest`、`newest`,</br>分别表示从指定仓库的`最早`、`最新`数据开始导出,默认值为oldest|| Spec |json|是|导出任务的参数主体,选择不同的`type`,`Spec`也需要填写不同的参数,将在下面分开讲解|| host |string|是|服务器地址（ip或域名）,例如:`https://pipeline.qiniu.com` 或 `127.0.0.1:7758`|
| uri |string|是|请求资源路径（具体地址,不包含ip或域名）,例如:`/test/repos`|

!> 注意: 导出数据格式和`推送数据`相同。**示例**

```curl -X POST https://pipeline.qiniu.com/v2/repos/test_Repo/exports/export_job1 \-H 'Content-Type: application/json' \-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \-d '{  "type": "http",  "spec": {
	"host": "www.qiniu.com",
	"uri": "/test/repos"  }}'
```

### 导出数据至时序数据库

**请求语法**

```POST /v2/repos/<RepoName>/exports/<ExportName>Content-Type: application/jsonAuthorization: Pandora <auth>{  "type": <tsdb>,  "whence": <ExportWhence>,  "spec": {        "destRepoName": <DestRepoName>,        "series": <SeriesName>,        "tags": {            "tag1": <#key1>,            "tag2": <#key2>,            ...        },        "fields": {            "field1": <#key1>,            "field2": <#key2>,            ...        },        "timestamp": <#key1>,        }
}```
**请求内容**
|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| destRepoName |string|是|数据库名称|
| series |string|是|数据库序列名称|
| tags |map|是|标签字段,也就是tsdb序列中的列名,以map形式展示,出现在tags中的列名可以group by,而没有出现的则不可以|
| fields |map|是|列名字段,以map形式展示,出现在fields中的列不可group by|
| timestamp |string|否|会用rfc3339日期格式进行解析,如果格式不正确则会抛弃这一条数据,如果此项为空,则默认使用当前时间。|

> 时序数据库中的timestamp字段的类型必须为 date；
> 消息队列中字段类型为:Long/String/Date 的字段都可以导出至时序数据库中的 timestamp 字段

**示例**

```curl -X POST https://pipeline.qiniu.com/v2/repos/test_Repo/exports/export_job4 \-H 'Content-Type: application/json' \-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \-d '{  "type": "tsdb",  "spec":{      	"destRepoName": "test_tsdb",
		"series": "req_io",
		"tags": {"type": "#type","src": "#src","zone": "#zone","time": "#time","bucket": "#bucket","domain": "#domain"
		},
		"fields": {"hits": "#hits","flow": "#flow"},            	}}'
```
### 导出数据至日志检索服务
**请求语法**
```POST /v2/repos/<RepoName>/exports/<ExportName>Content-Type: application/jsonAuthorization: Pandora <auth>{  "type": <logdb>,  "whence": <ExportWhence>,  "spec": {        "destRepoName": <DestRepoName>,                      "doc": {            "toRepoSchema1": <#fromRepoSchema1>,            "toRepoSchema2": {            	"toRepoSchema3": <#fromRepoSchema3>,            }            ......        }}```

**请求内容**
|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| destRepoName |string|是|日志仓库名称,命名规则遵循消息队列名称,如果输入的名称不存在,系统将会自动创建一个|
| doc |map|否|字段关系说明, `fromRepoSchema`表示源消息队列字段名称,`toRepoSchema`表示目标日志仓库字段名称,如果不填写该字段,则默认按照源消息队列字段信息录入|
> 消息队列中,字段的类型与日志检索服务中的字段类型需要作出如下对应:
> 
> 消息队列类型:string 对应 日志检索服务:string / date
> 
> 消息队列类型:long 对应 日志检索服务:long / date
> 
> 消息队列类型:float 对应 日志检索服务:float
> 
> 消息队列类型:array、map 对应 日志检索服务:object
> 
> 消息队列类型:date 对应 日志检索服务:date **示例**
```curl -X POST https://pipeline.qiniu.com/v2/repos/test_Repo/exports/export_job2 \-H 'Content-Type: application/json' \-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \-d '{  "type": "logDB",  "spec": {
	  "destRepoName": "logDB_testRepo", 	  "doc":{
			"user":"userName",
			"profile":{
				"age":"age"
			}	  }	}}'
```

### 导出数据至对象存储服务

**请求语法**
```POST /v2/repos/<RepoName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "type": <kodo>,
    "whence": <ExportWhence>,
    "spec": {
         "bucket": <Bucket>,
         "keyPrefix": <Prefix>, 
         "email": <Email>,  
         "accessKey": <AccessKey>,    
         "fields": {
             "key1": <#value1>,
             "key2": <#value2>,
             ...
          },
         "rotateInterval": <Interval>,   
         "format": <ExportFormat>,
         "compress": <true|false>,
         "retention": <Retention>
	}
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|bucket|string|是|数据中心名称|
|keyPrefix|string|否|导出的文件名的前缀|
|email|string|是|bucket所属用户的七牛账户名称|
|accessKey|string|是|七牛账户的公钥|
|fields|map|是|字段关系说明,`key`为`kodo-bucket`的字段名,`value`为导出数据的消息队列的字段名|
|rotateInterval|int|否|切分文件的时长,单位为秒(`s`)|
|format|string|否|文件导出格式,支持`json`、`text`、`parquet`三种形式,默认为`json`|
|compress|bool|否|是否开启文件压缩功能,默认为`false`|
| retention |int|否|数据储存时限,以天为单位,当不大于0或该字段为空时,则永久储存|

!> 注1:当一个文件导出到kodo之后,最多间隔`rotateInterval`之后将再次生成文件做导出。默认值30s,最小值为30s,最大值为60s(当`format`为`parquet`时,最大间隔可以到600s)。

!> 注2: `compress` 会压缩成`gzip`格式,但当用户指定`format`为`parquet`时,由于`parquet`已经是压缩好的列存格式,`compress`选项将不起作用。

!> 注3: `keyPrefix`字段表示导出文件名称的前缀,该字段可选,默认值为""(生成文件名会自动加上时间戳格式为`yyyy-MM-dd-HH-mm-ss`),如果使用了一个或者多个魔法变量时不会自动添加时间戳,支持魔法变量,采用`$(var)`的形式求值,目前可用的魔法变量var如下:

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

**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/repos/repox/exports/export1 \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='  \
-d '{
	"type": "kodo", 
	"spec": { 
		"bucket": "bucket2", 
		"keyPrefix": "key1/", 
		"email": "xiaoming@qiniu.com",
		"accessKey": "7H4FDc1M-FxXFZKwjbN_Up1OfY7DotXDjaM5jXzm",
		"fields": { 
			"f1": "#f1", 
			"f2": "#f2" 
		} 
		"format":"json",
		"compress": true
}' 
```
### 导出数据至MongoDB
**请求语法**```POST /v2/repos/<RepoName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "type": <mongodb>,
    "whence": <ExportWhence>,
    "spec": {        "host": <Host>,                                  "dbName": <DatabaseName>,                       "collName": <CollectionName>,                    "mode": [<INSERT>|<UPDATE>|<UPSERT>],                   "updateKey": <UpdateKey>,                       "doc": <Doc>,                                   "version": <MongoVersion>
	}}   ```

**请求内容**
|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| host |string|是|数据库的host地址|
| dbName |string|是|数据库名字|
| collName |string|是|集合名称|
| Mode |string|是|插入方式,分为INSERT/UPDATE/UPSERT三种|
| updateKey |string[]|否|假如插入方式是UPDATE或UPSERT,需要指定该参数|
| doc |json|是|插入或者更新的内容,支持mongo的update `$set` `$inc`等语法,可以用`$#colum`,</br>如果不填写该字段,则默认按照源表信息录入|
| version |int|否|mongo的版本号|**示例**

```curl -X POST https://pipeline.qiniu.com/v2/repos/test_Repo/exports/export_job3 \-H 'Content-Type: application/json' \-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \-d '{  "type": "mongo",  "spec":{        "host": "192.168.61.31:37217",                                  "dbName": "test_mongo",                       "collName": "req_rs_5m",                    "mode": "UPDATE",                   "updateKey": ["uid","zone","time_5m","bucket"],                       "doc": {"$inc": {"hits": "#hits"}},                                   "version": "2.4.7"            	}}'
```### 更新导出任务

**请求语法**

```
PUT /v2/repos/<RepoName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "spec": <Spec>
}
```
!> **注意:仅支持对 `spec` 的修改（不允许修改`type`、`whence`）** 

### 查看所有导出任务

**请求语法**

```
GET /v2/repos/<RepoName>/exports
Authorization: Pandora <auth>
```

**响应报文** 

```Content-Type: application/json
{
    "name": <ExportName>,
    "type": <ExportSchema>,
    "spec": <Spec>,
    "whence": <ExportWhence>
}```

### 根据名称查看导出任务

**请求语法**

```
GET /v2/repos/<RepoName>/exports/<ExportName>
Authorization: Pandora <auth>
```

### 根据名称删除导出任务
**请求语法**

```
DELETE /v2/repos/<RepoName>/exports/<ExportName>
Authorization: Pandora <auth>
```

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


 |名称|类型|必填|描述|
 |:---|:---|:---|:---|
 |DataSourceName|string|是|数据源节点名称|
 |region|string|是|所属区域|
 |type|string|是|数据源类型，可选值为`kodo`和`hdfs`|
 |spec|json|是|指定该数据源自身属性相关的信息|
 |schema|array|是|字段信息|
 |schema.key|string|是|字段名称|
 |schema.valtype|string|是|字段类型，支持`long`、`float`、`string`、`date`|
 |schema.required|bool|是|描述用户在传输数据时`key`字段是否必填|

 当type为`hdfs`的时候spec定义如下:
 |名称|类型|必填|描述|
 |:---|:---|:---|:---|
 |spec.path|array|是|包含一个或者多个hdfs文件路径|
 |spec.fileType|string|是|文件类型，合法取值为`json`、`text`和`parquet`|

 当type为`kodo`的时候spec定义如下:
 |名称|类型|必填|描述|
 |:---|:---|:---|:---|
 |spec.bucket|string|是|对象存储bucket名称|
 |spec.keyPrefix|array|是|包含一个或者多个文件前缀|
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

**请求方法** 

```
POST /v2/jobs/<JobName>
Content-Type: application/json
Authorization: Pandora <auth>
{
	"srcs":[
		{
			"name":<DataSourceName|JobName>,
			"fileFilter":<KeyPrefix/$yyyy-$mm-$dd>,
			"type":<DataSource|Job>
		},
		...
	],
   "computation": {
       "code": <SqlCode>,
       "type": <SQL>
   }, 
   "container": {
       "type": <ContainerType>,
       "count": <ContainerCount>
   },  # type 为 depend的时候，container依赖上游的配置，该配置不填
   "scheduler":{
   		"type": <crontab|loop|once|depend>,
   		"spec" : {
	   		"crontab": <0 0 0/1 * * ?>,  # type 为crontab
			"loop": <1h|3m|....> # type 为 loop，但是不填该字段或者该字段为0，则默认持续运行该任务 
   		} # type 为 once 和depend 的时候spec 可以不填
   },
   "params":[
   		{
   			"name":<ParamName>,
   			"default":<ParamValue>
   		},
   		...
   ]   # type 为 depend的时候，params依赖上游的配置，该配置不填
}
```
 

|名称|类型|必填|描述|
|:---|:---|:---|:---|
|srcs|array|是|数据来源，所有数据来源中最多包含一个离线任务，但可以包括多个离线数据源|
|srcs.name|string|是|数据源名称或离线任务名称|
|srcs.fileFilter|string|否|文件过滤规则，可使用魔法变量|
|srcs.type|string|是|数据来源节点类型|
|code|object|是|代码|
|code.code|string|是|代码|
|code.type|string|是|代码类型。暂时支持SQL|
|container|map|是|计算资源|
|container.type|string|是|规格|
|container.count|int|是|数量|
|scheduler|map|是|调度|
|scheduler.type|string|是|调度方式，定时、循环或单次执行三选一，下游任务是依赖模式|
|scheduler.spec.crontab|string|否|定时执行|
|scheduler.spec.loop|string|否|循环执行|
|params|array|否|自定义参数|
|params.name|string|是|参数名称|
|params.default|string|是|默认值|

注意：

1. scheduler.type 如果是depend 模式，代表这个离线任务依赖某个上游的离线任务。首先srcs内有且仅有一个离线任务数据源。同时该任务不能指定调度的模式、魔法变量和容器规格。这些全部使用上游依赖的离线任务。


### 更新离线计算任务

**请求方法** 

```
PUT /v2/jobs/<JobName>
Content-Type: application/json
Authorization: Pandora <auth>
{
	"srcs":[
		{
			"name":<DataSourceName|JobName>,
			"fileFilter":<KeyPrefix/$yyyy-$mm-$dd>,
			"type":<DataSource|Job>
		},
		...
	],
   "computation": {
       "code": <SqlCode>,
       "type": <SQL>
   }, 
   "container": {
       "type": <ContainerType>,
       "count": <ContainerCount>
   },
   "scheduler":{
   		"type": <crontab|loop|once|depend>,
   		"spec" : {
	   		"crontab": <0 0 0/1 * * ?>,  # type 为crontab
			"loop": <1h|3m|....> # type 为 loop，但是不填该字段或者该字段为0，则默认持续运行该任务 
   		} # type 为 once 和depend 的时候spec 可以不填
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

注意：
1. 更新时候 srcs, code, container, scheduler, params 校验逻辑和创建的时候相同。下游计算任务不指定container, scheduler, params。更新逻辑为全量更新。需要提前获取所有信息。



### 列举离线计算任务信息

**请求方法** 

```
GET /v2/jobs?page=1&size=10&parentRepo=xx&parentJob=yy
Authorization: Pandora <auth>

```

**响应报文** 

```
{
	[
		{
			"srcs":[
				{
					"name":<DataSourceName|JobName>,
					"fileFilter":<KeyPrefix/$yyyy-$mm-$dd>,
					"type":<DataSource|Job>
				},
				...
			],
		   "scheduler":{
		   		"type": <crontab|loop|once|depend>,
   		   		"spec" : {
			   		"crontab": <0 0 0/1 * * ?>,  # type 为crontab
					"loop": <1h|3m|....> # type 为 loop，但是不填该字段或者该字段为0，则默认持续运行该任务 
		   		} # type 为 once 和depend 的时候spec 可以不填
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
		   ]
		}
	]
	
}
```

**查询参数** 


|名称|类型|必填|描述|
|:---|:---|:---|:---|
|page|int|是|分页查询，第几页|
|size|int|是|分页查询，每页多少条|
|parentRepo|string|否|依赖离线数据源名字|
|parentJob|string|是|依赖离线计算任务名字|



### 获取单个离线计算任务信息

**请求方法** 

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
			"fileFilter":<KeyPrefix/$yyyy-$mm-$dd>,
			"type":<DataSource|Job>
		},
		...
	],
   "scheduler":{
   		"type": <crontab|loop|once|depend>,
		"spec" : {
			   		"crontab": <0 0 0/1 * * ?>,  # type 为crontab
					"loop": <1h|3m|....> # type 为 loop，但是不填该字段或者该字段为0，则默认持续运行该任务 
		   		} # type 为 once 和depend 的时候spec 可以不填
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
   ]
}
```



### 启动离线计算

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

|名称|类型|必填|描述|
|:---|:---|:---|:---|
| params |array|否|定义运行时的魔法参数值|
| params.name |string|是|魔法变量名字|
| params.value |string|是|运行时魔法变量值|



### 停止离线计算

```
POST /v2/jobs/<JobName>/actions/stop
Authorization: Pandora <auth>
```


### 获取计算任务schema

**请求方法** 

```
GET /v2/jobs/<JobName>/schema
Authorization: Pandora <auth>
```

**返回内容** 

```
{
	"schema": [
      {
        "key": <Key>,
        "valtype": <ValueType>
      },
      ...
    ]
}


```

### 删除离线计算任务信息

**请求方法** 

```
DELETE /v2/jobs/<JobName>
Authorization: Pandora <auth>
```


### 获取数据源schema

**请求方法** 

```
POST /v2/schemas
Content-Type: application/json
Authorization: Pandora <auth>
{
	"type": <Kodo|HDFS>, 
	"spec": <Spec>
}
```

**Type为Kodo Spec结构**


|名称|类型|必填|描述|
|:---|:---|:---|:---|
| spec.bucket |string|否|对象存储的存储空间|
| spec.keyPrefix |string|是|一个或者多个文件前缀|
| spec.fileType |string|是|文件类型，合法取值为`json`, `parquet`, `text`|

**Type为HDFS Spec结构**


|名称|类型|必填|描述|
|:---|:---|:---|:---|
| spec.paths |string|是|一个或者多个文件路径|
| spec.fileType |string|是|文件类型，合法取值为`json`, `parquet`, `text`|


**返回内容** 

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

### 创建离线计算导出任务

**请求语法**
```POST /v2/jobs/<JobName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "type": <Type>,
    "spec": {
         "bucket": <Bucket>,
         "keyPrefix": <Prefix|Path>,
         "fields": {
             "key1": <#key1>,
             "key2": <#key2>,
             ...
          },   
         "format": <ExportFormat>,
         "compress": <true|false>,
         "retention": <Retention>,
         "partitionBy": <PartitionBy>，
         "fileCount": <FileCount>,
         "overwrite": <Overwrite>
	}
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|type|string|是|导出的类型，目前允许的值为"kodo"|
|bucket|string|是|对象存储bucket名称|
|keyPrefix|string|否|导出的文件名的前缀，当离线任务的`scheduler`是`once`的时候，就是文件名|
|fields|map|是|字段关系说明,`key`为`kodo-bucket`的字段名,`value`为离线任务的数据源中的字段名|
|format|string|否|文件导出格式,支持`json`、`text`、`parquet`三种形式,默认为`parquet`|
|compress|bool|否|是否开启文件压缩功能,默认为`false`|
|retention|int|否|数据储存时限,以天为单位,当不大于0或该字段为空时,则永久储存|
|partitionBy|array|否|指定作为分区的字段，为一个字符数组，合法的元素值是字段名称|
|fileCount|int|是|计算结果导出的文件数量，应当大于0，小于1000|
|overwrite|bool|否|是否对已经存在的文件覆盖写，默认为`true`|

!> 注1: `compress` 会压缩成`gzip`格式,但当用户指定`format`为`parquet`时,由于`parquet`已经是压缩好的列存格式,`compress`选项将不起作用。

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

```
GET /v2/jobs/<jobName>/history?page=1&size=20&sortBy=endTime&order=desc&status=xx&runid=-1
Content-Type: application/json
Authorization: Pandora <auth>


return

{
"total": <TotalCnt>, # 总共运行批次
"history":[
	{
		"id" : <RunCnt>, # 运行批次
		"startTime": <StartTime>,
		"endTime": <EndTime>,
		"status": <Ready|Success|Fail|Running|Cancel>,
		"message": <Message>
	},
]
}
```

**查询参数**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|page|int|否|分页页数|
| size |int|否|分页大小|
| sortBy |string|否|根据哪个字段排序|
| order |string|否|排序asc，desc|
| status |string|否|<Ready|Success|Fail|Running|Cancel>|
| runid |int|否|查询某个运行批次的状态，-1代表最近一次|


**返回体**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|total|int|是|总运行批次次数|
|history|array|是|运行历史|
|history.id|int|是|运行批次|
| history.startTime |string|否|启动时间|
| history.endTime |string|否|终止事件，如果为Running，则为当前时间|
| history.status |string|否|<Ready|Success|Fail|Running|Cancel>|
| history.message |string|否|运行、出错信息，比如运行成功、内存溢出、数据损坏|



### 错误代码及相关说明

| 错误码 | 错误描述 |
| :---  | :----- |
|500   |服务器内部错误 |
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
|400	|E18108: 不正确的时间戳类型|
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
|400   |E18305: 导出任务出现错误|


