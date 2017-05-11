### API 接收地址

`https://logdb.qiniu.com`

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

### 创建日志仓库

**请求语法**

```
POST /v5/repos/<RepoName>
Content-Type: application/json
Authorization: Pandora <auth>
{
  "region": <Region>,
  "retention": <Retention>,
  "schema": [
    {
      "key": <Key>,
      "valtype": <ValueType>,
      "searchWay":<SearchWay>,
      "analyzer":<AnalyzerName>
    },
    "analyzers":[
    	{
    		"analyzerName":<AnalyzerName>,
    		"options":{
    			<key>:<Value>,
    			....
    		}
    	}
    ],
    "children":{
    	<ChildName>:{
    		"schema":[
    			....
    		]
    	}
    },
    "similarity":{
    	"name":<Name>,
    	"options":{
    		"type":<Type>,
    		<Key>:<Value>,
    		....
    	}
    }
    ...
  ]
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|RepoName|string|是|日志仓库名称，用来标识该日志仓库的唯一性；</br>命名规则: `^[a-z][a-z0-9_]{0，127}$`，1-128个字符，支持小写字母、数字、下划线；</br>必须以小写字母开头|
|region|string|是|所属区域,计算与存储所使用的物理资源所在区域,目前支持华东区域(代号`nb`)；</br>此参数是为了降低用户传输数据的成本，应当尽量选择离自己数据源较近的区域|
|retention|string|是|数据存储时限，1天=`1d`，最大支持30天|
|schema|json|是|字段信息|
|key|string|是|字段名称，用来标识该字段的唯一性；</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`，1-128个字符，支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
|valtype|string|是|字段类型，目前支持`string`、`float`、`long`、`boolean`,`date`，`ip`,`geo_point`和`object`共8种类型；</br>其中`date`支持`RFC3339Nano`和`RFC3339Nano(Numeric time zone offsets format)`，</br>例：`2006-01-02T15:04:05.999999999Z07:00`和`2006-01-02T15:04:05.999999999+08:00`;`geo_point`为经纬度坐标，如 `[ -71.34, 41.12 ]`|
| searchWay |string|否|检索方式，支持`keyword`（单词检索）和`full_text`（全文检索）两种方式；</br>当`valtype`为`string`类型时，指定这个参数才有效，其他类型无需关注，</br>如果没有传递该参数，默认为'full_text'方式|
| schema.analyzer |string|否|文本分词方式，支持`standard`,`simple`,`whitespace`,`stop`,`index_ansj`(中文分词)5种标准分词方式，同时支持自定义分词器，见`analyzers`定义。
| analyzers |json|否|自定义分词器，解释见详解。
| children|json|否| 定义子repo，详情见children详解。

**searchWay 详解:**

假设目前用户有一条数据的key为`name`，value为 `"zhang xiaoming"`；

另一条数据的key为`city`，value为 `"shanghai"`；

那么使用`keyword`方式查询第一条时，查询条件为`q=name:zhang`或`q=name:xiaoming`，则无法查询到该条数据；

而使用`full_text`方式查询第一条时，查询条件为`q=name:zhang`或`q=name:xiaoming`，则可以正常查询到该条数据；

使用`keyword`方式查询第二条时，查询条件为`q=city:shanghai`，则可以正常查询到该条数据；

也就是说，`keyword`方式只能查询整个字段内容为一个完整单词时使用；

**自定义分词方式analyzers 详解:**

目前支持只支持 type="pattern"。

下面定义一个type为`pattern`的例子。

`stopwords`表示停词，会在索引时被过滤掉。在下面例子中，如果文本中有‘bug'，那么搜索时则不会被搜索到。

`pattern`使用正则表达如何分词，比如 `"[^\\w]+"`表示按 非字母分词，那么text=`type_1-type_4`，会被分为：`"type_1","type_4"`两个词。

定义好后，则可以在schema中的analyzer字段指明使用`testAnalyzerName`。

例子：
```
{
            "analyzerName":"testAnalyzerName",
            "options":{
                 "type":"pattern",
                 "stopwords":["bug"],
                 "pattern":"[^\\w]+"
            }
}
```

**children详解:**

一个repo中使用children对Docs做父子关联

Docs父子关联是指在docs之间建立一个`parent-child`关系，用于相互关联表示一对多的关系。和嵌套类型相比较而言，有以下优势：

1. 被指定为parent的docs可以直接更新，而不用更新所有的子doc。如果使用嵌套类型，则需要更新整个doc。

2. children的docs可以做增删改查操作，而不用影响parent功能。当子doc的数量非常庞大并且需要经常性的增加或者修改时，这个特性非常关键。

 使用示例

创建一个LogDB的Repo`testRepo`示例

```
{
  "region": "nb",
  "retention": "-1",
  "children": {
    "myChildRepo": {
      "schema": [
        {
          "key": "k2",
          "valtype": "string",
          "primary": true
        }
      ]
    }
  },
  "schema": [
    {
      "key": "k1",
      "valtype": "geo_point"
    },
    {
      "key": "id",
      "valtype": "string",
      "primary": true
    }
  ]
}
                 
                 
```
* `children` 表示子Repo。（我们创建的`testRepo`默认为`parent`）
* `myChildRepo` 表示用户自定义的子repo名字，需要对child repo定义`schema`，schema中必须选择一个string字段作为主键`primary:true`



 打点示例

* 打点分为两类，一类是直接打点给parent repo，第二类是给`children`下的repo打点。Parent repo打点时和普通打点一样。

* 打点到parent repo

```
{
	 "k1":[1,2],
	 "id":"mainRepoUuid
}
```



下面介绍`children` Repo打点时需要关联parent docs时的打点方式。

* 打点到child repo

```
{
	"k2":"testk2data",
    "__parent":"mainRepoUuid",
    "__type":"myChildRepo"
}
```
                 
   
   * `__parent` 字段指明需要关联的Doc ID，这里等于刚才主Repo中的数据中`id=mainRepoUuid` 
   * `__type` 字段指明该Doc属于哪个`children`，这里使用的是创建repo时指定的`myChildRepo`

   
   
关联查询示例

 比如现在想要查询 `myChildRepo`中关联了 `id=mainRepoUuid`的doc
 
 `q=id:mainRepoUuid&scope=myChildRepo&has_parent=testRepo`    
 
 * `q`是关联条件，这里一般为 `<指定的ID Key Name>:<ID Value>`
 * `scope`是搜索的范围，比如这里我要搜索的是`myChildRepo`下的关联了`mainRepoUuid`的数据。

 ####反过来
 
 比如现在想要查询主repo中关联了`myChildRepo`中`k2:testk2data`的doc
 
 `q=k2:testk2data&scope=testRepo&has_child=myChildRepo`

**schema.valtype.object 类型：**

`object`是一个json对象，下面例子中的`work`字段即为`object`类型，其中`key`为`string`类型，`value`的类型会根据实际数据进行推断。凡是`object`类型的字段会被自动索引，被索引后，可以通过`work.address`作为key进行搜索。

```
   [
        {
            "name":"testName",
            "work":{
                "address":"workAddress",
                "name":"workName",
                "salary":111.00
            },
            "timestamp":"2016-05-10T15:00:00+08:00"
        }
    ]   
```

**示例**

```
curl -X POST https://logdb.qiniu.com/v5/repos/test_repo \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
  "region": "nb",
  "retention": "1d"
  "schema": [
    {
      "key": "userName",
      "valtype": "string",
      "searchWay":"keyword"
    },
    {
      "key": "createDate",
      "valtype": "date"
    }
  ]
}'
```

### 查询日志

**请求语法**

```
GET /v5/repos/<RepoName>/search?q=<QueryString>&sort=<fieldName:asc>&from=<from>&size=<size>
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---|:---|
|RepoName|string|是|日志仓库名称|
|q|string|否|查询表达式，参照查询语法|
|sort|string|否|排序，`field1:asc,field2:desc, … `。field 是实际字段名，asc代表升序，desc 代表降序。用逗号进行分隔。|
|from|int|否|日志开始的位置|
|size|int|否|返回数据数量|
|fields|string|否|选择返回的数据中只展示部分字段。比如 fields=k1,k2，则返回的结果中只有k1,和k2字段|

**查询语法**

**关键字**

|名称|语义|
|:--|:--|
|AND|query1 AND query2，查询交集|
|OR|query1 OR query2，查询并集|
|NOT|query1 NOT query2，表示符合query1，不符合query2的结果|
|()|把一个或多个query合并成一个query，提升优先级|
|[]|区间查询，包括边界|
|{}|区间查询，不包括边界|
|\|转义字符|
|>,=,<,<=,>=|区间查询|

**查询举例**

* 字段field包括a的log：field:a
* 字段field包括a,b的log：field:a OR field:b
* 字段field包含a或者b,不包含c：(field:a OR field:b) AND (NOT field:c)
* 字段field1包括a,b同时域field2包括c的log：(field1:a OR field1:b) AND (field2:c)
* 2016-1-1到2016-1-2的数据：date:[2016-01-01 TO 2016-01-02]
* 在数字1-5之间的log： count:[1 TO 5]
* 大于5的log： count:>5 
* 包含hello,world，同时二者之间有5个单词相隔：field:"hello world"~5

**响应报文**

```
200 OK
Content-Type: application/json
{
    "total": <Total>,
    "partialSuccess": <partialSuccess>,
    "data": [
        {
           "id": "<ID>",
           "<Key>": "<Value>",
           ...
        },
        ...
    ]
}
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---|:---|
|total|int|-|共有多少结果数据|
|partialSuccess|bool|-|为`true`时，说明该次查询涉及的数据量比较大，需要较久的时间，我们将提前结束，并返回部分结果|
|data|json|-|数据结果集|
|id|string|-|系统自动生成，表示该条数据的唯一性|
|key|string|-|字段名称，`value`为此条数据内容|

**请求报文示例**

```
curl -G https://logdb.qiniu.com/v5/repos/search?q="count:>=10 AND <20"&sort="userName:asc"&from=1&size=100 \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' 
```

**响应报文示例**

```
200 OK
Content-Type: application/json
{
    "total": 100,
    "partialSuccess": true,
    "buckets": [
        {
           "id": "AVczSkfVrDpcIhsMRsgK",
           "userName": "xiaowang",
        },
        ...
    ]
}
```

### 根据时间聚合日志

**请求语法**

```
GET /v5/repos/<RepoName>/histogram?q=<QueryString>&from=<StartTimestamp>&to=<EndTimestamp>&field=<AggregationField>
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---|:---|
|RepoName|string|是|日志仓库名称|
|q|string|否|查询表达式，参照查询语法|
|from|long|是|聚合的开始时间戳，例如： `1470163291000`|
|to|long|是|聚合的结束时间戳，例如： `1470336091000`|
|field|string|是|聚合的字段，该字段必须为`date`|

!> 注意： `根据时间聚合日志`返回的是用户选择的时间段内日志的分布情况。

### 更新日志仓库

**请求语法**

```
PUT /v5/repos/<RepoName>
Content-Type: application/json
Authorization: Pandora <auth>
{
  "retention": <Retention>
  "schema": [
    {
      "key": <Key>,
      "valtype": <ValueType>,
      "searchWay":<SearchWay>
    },
    ...
  ]
}
```

**更新repo的注意事项:**

更新repo的schema是延迟生效的，一般情况下，更新的repo生效时间需要2天，即今天更新的repo schema，要后天凌晨才生效。
在更新的repo schema生效之前，数据依旧只能以原来的schema写入, 且schema更新后，也只对新数据生效。

**示例**

```
curl -X POST https://logdb.qiniu.com/v5/repos/test_repo \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
  "retention": "1d"
  "schema": [
    {
      "key": "userName",
      "valtype": "string"
    },
    {
      "key": "createDate",
      "valtype": "date"
    }
  ]
}'
```

### 删除日志仓库

**请求语法**

```
DELETE /v5/repos/<RepoName>
Authorization: Pandora <auth>
```

### 根据名称查询日志仓库

**请求语法**

```
GET /v5/repos/<RepoName>
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---|:---|
|RepoName|string|是|日志仓库名称|

**响应报文**

```
200 OK
{
  "region": <Region>,
  "retention": <Retention>
  "schema": [
    {
      "key": <Key>,
      "valtype": <ValueType>
    },
    ...
  ],
  "createTime": <CreateTime>,
  "updateTime": <UpdateTime>
}
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---|:---|
|region|string|-|所属区域|
|retention|string|-|存储时限|
|schema|json|-|字段信息|
|createTime|string|-|创建时间|
|updateTime|string|-|更新时间|

### 查询所有日志仓库

**请求语法**

```
GET /v5/repos
Authorization: Pandora <auth>
```

**响应报文**

```
200 OK
Content-Type: application/json
{
  repos: [
    {
      "name": <RepoName>,
      "region": <RegionName>,
      "retention": <Retention>
      "createTime": <CreateTime>,
      "updateTime": <UpdateTime>
    },
    ...
  ]
}
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---|:---|
|name|string|-|日志仓库名称|
|region|string|-|所属区域|
|retention|string|-|存储时限|
|createTime|string|-|创建时间|
|updateTime|string|-|更新时间|



**创建和更新的分词器生效时间同更新repo后**

### 创建Analyzer 

**请求语法**

```
POST /v5/repo/<RepoName>/analyzer
Authorization: Pandora <auth>
Content-Type: application/json
[
    {
        "key":"<KEY>",
        "analyzer":{
            "analyzerName":"<AnalyzerName>",
            "options":{
                 "type":"<TYPE>",
                 "stopwords":["word1","word2"...],
                 "pattern":"PATTERN"
            }
            
        }
    }
]
```

返回包：

```
200 OK
```



#### 更新分词器

请求包：

```
PUT /v5/repo/<RepoName>/analyzer
Authorization: Pandora <auth>
Content-Type: application/json
[
    {
        "key":"<KEY>",
        "analyzer":{
            "analyzerName":"<AnalyzerName>",
            "options":{
                 "type":"<TYPE>",
                 "stopwords":["word1","word2"...],
                 "pattern":"PATTERN"
            }
        }
    }
]
```

返回包：

```
200 OK
```



#### 查询分词器

请求包：

```
GET /v5/repo/<RepoName>/analyzer
Authorization: Pandora <auth>
```

返回包
```
Content-Type: application/json
[{
    "analyzerName":"<AnalyzerName>",
    "options":{
         "type":"<TYPE>",
         "stopwords":["word1","word2"...],
         "pattern":"PATTERN"
    }
}]
```

#### 删除分词器

请求包：

```
DELETE /v5/repo/<RepoName>/analyzer/<AnalyzerName>
Authorization: Pandora <auth>
```

返回包：

```
200 OK
```



## 错误代码及相关说明

**通用错误**

|http code|error message|说明|
|:---|:------------|:--|
|400|E8001: Invalid arguments|参数不合法|
|500|E8004: Internal server error|服务器内部错误|

**仓库(Repo)相关错误**

|http code|error message|说明|
|:---|:------------|:--|
|400|E8101: Missing required Repo|reponame为空|
|400|E8102: Missing required Retention|retention为空|
|400|E8103: Missing required Region|region为空|
|400|E8104: The data you provided is not match with your repo schema|打点格式错误|
|400|E8105: The Repo you provided is invalid|提供的reponame不合法|
|400|E8106: The Retention you provided is invalid|提供的retention不合法|
|400|E8107: The Region you provided is invalid|提供的region不合法|
|400|E8108: The Schema you provided is invalid|提供的Schema不合法|
|404|E8111: The specified repo does not exist under the provided appid|repo不存在|
|409|E8112: The specified repo already exists under the provided appid|repo已存在|
|400|E8113: The request frequency is beyond limit. Try later.|请求过于频繁,请稍候重试|
|413|E8114: Request entity too large	|请求体太大,请确认后重试|
|400|E8115: Request entity empty|请求体为空,请确认后重试|
|400|E8116: Request entity malformed|请求体不合法,请确认后重试|

**Indice(索引)相关错误**

|http code|error message|说明|
|:---|:------------|:--|
|400|E8202: The q you provided is invalid|请求参数q不合法|
|400|E8203: The from you provided is invalid|请求参数from不合法|
|400|E8204: The size you provided is invalid|请求参数size不合法|
|400|E8205: The sort you provided is invalid|请求参数sort不合法|
|400|E8206: The time range parameter value cannot be empty!|请求参数from,to不能为空|
|400|E8207: The end time you provided is invalid|请求参数to不合法|
|400|E8208: The end time you provided must be later than from time!|请求参数to必须小于from|
|400|E8209: The aggregation field cannot be empty!|必须指定聚合的字段|

**其它错误**

|http code|error message|说明|
|:---|:------------|:--|
|400|E8301: The Params you provided is invalid|提供的参数不合法,请检查|