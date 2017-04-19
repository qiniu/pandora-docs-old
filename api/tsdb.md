### API接收地址

```
https://tsdb.qiniu.com
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

### 创建数据仓库

**请求语法**

```
POST /v4/repos/<RepoName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "region": <Region>,
    "metadata":{
        "key1":"value1",
        ...
    }
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|RepoName|string|是|数据仓库名称,用来标识该数据仓库的唯一性；</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`,1-128个字符,支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
|region|string|是|所属区域,计算与存储所使用的物理资源所在区域,目前支持华东(代号`nb`)；</br>此参数是为了降低用户传输数据的成本,应当尽量选择离自己数据源较近的区域|
| metadata |map|否|备注信息|
| key |string|否|备注信息标题,</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`,1-128个字符,支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
| value |string|否|备注信息内容,1-500个字符长度|

**示例**

```
curl -X POST https://tsdb.qiniu.com/v4/repos/test_repo \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
    "region": "nb", 
    "metadata":{
        "info":"this repo is be for the use of test"
    }
}' ```

### 创建序列

**请求语法**

```
POST /v4/repos/<RepoName>/series/<SeriesName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "retention": <Retention>,
    "metadata": {
    	"key": "value",
    	 ...
    }
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|RepoName|string|是|所属数据仓库名称|
|SeriesName|string|是|序列名称,用来标识该序列的唯一性；</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`,1-128个字符,支持小写字母、数字、下划线；</br>必须以大小写字母或下划线开头|
|retention|string|是|数据存储时限,目前支持:`1d` - `30d`,1-30天|
|metadata|string|否|备注信息|

**示例**

```
curl -X POST https://tsdb.qiniu.com/v4/repos/test_repo/series/test_series \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
{
    "retention": "1d",
    "metadata": {
    	"author": "Lynk Lee"
    }
}
```

### 数据查询
**请求语法**

```
GET /v4/repos/<RepoName>/query?q=<Sql>[&timezone=<TimeZone>]
Authorization: Pandora <auth>
```
or

```
POST /v4/repos/<RepoName>/query[?timezone=<TimeZone>]
Content-Type: application/json
Authorization: Pandora <auth>
{
    "sql" : <Sql>
}
```

or

```
POST /v4/repos/<RepoName>/query[?timezone=<TimeZone>]
Content-Type: text/plain
Authorization: Pandora <auth>
<Sql>
```

**请求内容**

* 查询默认时区为`+00`或者`-00`,即`utc`

* 存储在tsdb中的数据的时间戳都是`UTC`时间,`timezone`参数用于指定查询者所在的时区相对`UTC`时间的偏移量

* `TimeZone`取值范围为`[-12, +14]`,表示与`utc`时区的偏移量,如东八区(中国)应写成`+08`,会将查询范围整体加上8小时,西五区应该写成`-05`,查询范围的时间则被减去5小时。

**举例说明:**

`1466510400000000000`是UTC时间`2016-06-21 12:00:00`,和东八区时间`2016-06-21 20:00:00`的`unixnano`表示

```
select value from cpu where time > '2016-06-21T12:00:00Z'
```


当以上查询语句的`timezone`参数为`+08`的时候,该sql语句能够检索的数据的时间范围为

`(2016-06-21 04:00:00 UTC,当前时间)`

当以上查询语句的`timezone`参数为`+00`的时候,该sql语句能够检索的数据的时间范围为

`('2016-06-21 12:00:00 UTC', 当前时间]`

当写入的数据点的时间戳为`1466510400000000000`的时候,如果想要查到该点,可以

```
select value from cpu where time = '2016-06-21T12:00:00Z' //timezone=0
```

或者

```
select value from cpu where time = '2016-06-21T20:00:00Z' //timezone=8
```


**响应报文**

```
200OK
type QueryRet struct {
    Results []Result     `json:"results,omitempty"`
}
type Result struct {
    Series []Serie      `json:"series,omitempty"`
}
type Serie struct {
    Name    string            `json:"name,omitempty"`
    Tags    map[string]string `json:"tags,omitempty"`
    Columns []string          `json:"columns,omitempty"`
    Values  [][]interface{}   `json:"values,omitempty"`
}
```
以上是tsdb返回的结果序列化之前的结构体

* 每个`QueryRet`对应的是一个请求返回的结果
* `QueryRet中`每个`Result`对应一条`sql`的查询结果
* `Result`中的每个`Series`对应一个`tagSet`的结果,即`Result`每有一个不同的`tags`组合,就会有一个`Serie`的结果添加到`Result.Seires`这个`slice`中


一个简单的查询,如`SELECT value FROM cpu_load_short WHERE region='us-west' `结果类似如下json字符串:

```
{
    "results": [
        {
            "series": [
                {
                    "name": "cpu_load_short",
                    "columns": [
                        "time",
                        "value"
                    ],
                    "values": [
                        [
                            "2015-01-29T21:55:43.702900257Z",
                            0.55
                        ],
                        [
                            "2015-01-29T21:55:43.702900257Z",
                            23422
                        ],
                        [
                            "2015-06-11T20:46:02Z",
                            0.64
                        ]
                    ]
                }
            ]
        }
    ]
}
```

### 根据名称查询数据仓库信息

**请求语法**

```
GET /v4/repos/<RepoName>
Authorization: Pandora <auth>
```

**响应报文**

```
200 OK
Content-Type: application/json
{
    "name": <RepoName>,
    "region": <Region>,
    "metadata": <Metadata>,
    "createTime": <CreateTime>,
    "deleting": <"true"|"false">
}
```

* 如果请求成功,且没有数据,则返回一个空列表:

```
200 OK
Content-Type: application/json
[ ]
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---|:---|
|createTime|string|-|创建时间|
|deleting|string|-|该repo是否处在删除中状态,返回`true` or `false`|

### 查询所有数据仓库信息

**请求语法**

```
GET /v4/repos
Authorization: Pandora <auth>
```

**响应报文**

```
200 OK
Content-Type: application/json
[
    {
        "name": <RepoName>,
        "region": <Region>,
        "metadata": <Metadata>,
        "createTime": <CreateTime>,
        "deleting": <"true"|"false">
    },
    ...
]
```

* 如果请求成功,且没有数据,则返回一个空列表:

```
200 OK
Content-Type: application/json
[ ]
```


### 查询某个数据仓库下所有序列信息

**请求语法**

```
GET /v4/repos/<RepoName>/series[?showMeta=<true|false>]
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|RepoName|string|是|数据仓库名称|
|showMeta|bool|是|是否显示备注信息,`true` or `false` | 

**响应报文**

```
200 OK
Content-Type: application/json
[
    {
        "name": <SeriesName>,
        "retention": <Retention>,
        "metadata": <Metadata>, 
        "createTime": <CreateTime>,
        "type":<Type>,
        "deleting": <"true"|"false">
    },
    ...
]
```

* 如果请求成功,且没有数据,则返回一个空列表:

```
200 OK
Content-Type: application/json
[ ]
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---|:---|
|seriesName|string|-|序列名称|
|retention|string|-|存储时限| 
|createTime|string|-|创建时间| 
|type|string|-|创建类型,为`1`是表示这个序列由用户直接创建,默认为1| 
|deleting|string|-|是否在删除中| 
|metadata|string|-|备注信息| 

### 更新数据仓库的meta data

**请求语法**

```
POST /v4/repos/<RepoName>/meta
Content-Type: application/json
Authorization: Pandora <auth>
{
    "metadata": {
        "key1":"value1",
        ...
    }
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|RepoName|string|是|数据仓库名称|
|metadata|string|是|备注信息|
|key|string|是|备注信息标题|
|value|string|是|备注信息内容|

**示例**

```
curl -X POST https://tsdb.qiniu.com/v4/repos/test_repo/meta \
-H 'Content-Type: application/json \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
    "metadata": {
        "author":"Lynk Lee"
    }
}'
```

### 删除数据仓库的meta data

**请求语法**

```
DELETE /v4/repos/<RepoName>/meta
Authorization: Pandora <auth>
```

### 删除数据仓库

**请求语法**

```
DELETE /v4/repos/<RepoName>
Authorization: Pandora <auth>
```

### 更新序列的meta data

**请求语法**

```
POST /v4/repos/<RepoName>/series/<SeriesName>/meta
Content-Type: application/json
Authorization: Pandora <auth>
{
    "metadata": <Metadata>
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|RepoName|string|是|所属数据仓库名称|
|SeriesName|string|是|序列名称|
|metadata|string|是|备注信息|

### 删除序列的meta data

**请求语法**

```
DELETE /v4/repos/<RepoName>/series/<SeriesName>/meta
Authorization: Pandora <auth>
```

### 删除序列

**请求语法**

```
DELETE /v4/repos/<RepoName>/series/<SeriesName>
Authorization: Pandora <auth>
```

### 错误代码及相关说明
 
**创建、管理相关接口报错**
 
| http code | error message | 说明 |
|:---- |:---- |:---- |
| 500 | E9001: server internal error | 服务器内部错误,请联系管理员解决 |
| 400 | E6000: Request data format not supplied as expected | 传入的body格式不对 |
| 404 | E6004: The request resource not found | 请求的资源不存在。 |
 
**仓库(Repo)层错误**
 
| http code | error message | 说明 |
|:---- |:---- |:---- |
| 400 | E6100: RepoName is limited between 1 and 128 characters, which the first charactor must be a letter and the others are letters, numbers and underlines! | 数据仓库名称格式错误 |
| 400 | E6101: Region should not be empty! | 创建数据仓库时需要加上所属区域字段。 |
| 400 | E6102: RepoName already exists, please use a new RepoName! | 该数据仓库已经存在。 |
| 400 | E6103: Region does not exist! | 所属区域参数错误。 |
| 400 | E6106: Resouce of the RepoName is in process of deleting, please wait a moment | 该数据仓库正在被删除中。 |
| 409 | E6110: Repo is currently modifying, please wait a moment | 该数据仓库正在被修改,需要等待。 |
  
**序列(Series)层错误**
  
| http code | error message | 说明 |
|:---- |:---- |:---- |
| 404 | E6205: Retention does not exists | 存储时限参数错误 |
| 400 | E6300: Series name is limited between 1 and 128 characters, which the first character must be a letter and the others are letters, numbers and underlines! | 序列名字格式错误 |
| 400 | E6301: Series name contains key words, please rename! | 序列命名时用到了关键字,请重新创建。 |
| 409 | E6302: Series already exists  | 该序列已经存在 |
| 404 | E6303 Series does not exist | 该序列不存在 |
| 400 | E6305: Series is in process of deleting, please wait a moment | 该序列正在删除中 |
| 409 | E6307: Series is currently modifying, please wait a moment | 该序列正在被修改 |
| 400 | E6308: Max Series number exceeded | 序列创建已达上限 |
 
**查询语句相关**
 
| http code | error message | 说明 |
|:---- |:---- |:---- |
| 400 | E7200: Invalid sql | sql语句不合法,出错原因: \<error message\> |
| 400 | E7202: Invalid time zone | 时区设置错误 |
| 400 | E7204: Query execution was interrupted, max execute time exceeded | 查询时间执行过长。 |
| 400 | E7212: Execute Sql error | 执行sql出错,出错原因: \<error message\> |
| 500 | E9002: executor query chan closed without result error | 查询语句被中断  |
