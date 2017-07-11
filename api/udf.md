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
```

**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/udf/jars/UdfAggCollection \
-H 'Content-Type: application/java-archive' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-T /path/to/udf-agg-1.0-SNAPSHOT.jar
```


### 修改UDF jar包信息

**请求语法**：

```
PUT /v2/udf/jars/<JarName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "description": "<UDF 描述信息>"
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|JarName |string|是|jar包名称唯一标识一个jar文件，命名规则: `^[a-zA-Z][a-zA-Z0-9_\\.]{0,127}[a-zA-Z0-9_]$`,1-128个字符,支持小写字母、数字、下划线；必须以大小写字母开头|
|description |string|是|对于jar包的一些描述信息|


**响应报文**

```
200 OK
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
|jarName |string|是|jar包名称, 如UdfAggCollection|

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
404 NotFound
Content-Type: application/json
{
    "error":"jar file does not exist"
}
```

### 获取UDF jar包列表

**请求语法**：

```
GET /v2/udf/jars?from=1&size=2&sort=uploadTime
Content-Type: application/json
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|from |int|否|分页条件，从第几条开始展示，不填则返回全部|
|size |int|否|分页条件，返回多少条展示，不填则返回全部|
|sort |string|否|排序字段，格式如列名:排序顺序，如sort=uploadTime:desc, sort=uploadTime:asc，如果不填排序顺序，如sort=uploadTime，则默认顺序排列|

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|jarName |string|是|jar包名称|
|description|string|是|jar包描述|
|uploadTime |string|是|jar包上传时间|

**示例**

```
curl -X GET https://pipeline.qiniu.com/v2/udf/jars?from=1&size=2&sort=uploadTime \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='
```

**响应报文**

```
200 OK
Content-Type: application/json
{
    "result": [
        {
            "jarName": "UdfAggCollection",
            "description":"聚合类UDF集合",
            "uploadTime": "2017-06-01 15:00:00"
        },
        {
            "jarName": "UdfSortCollection",
            "description":"排序类UDF集合",
            "uploadTime": "2017-06-01 14:00:00"
        }
    ]
}
```

### UDF函数注册

**请求语法**：

```
POST /v2/udf/funcs/<FuncName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "jarName": "<Jar Name>",
    "className": "<Full path class name>",
    "funcDeclaration" : "<Function Definition>",
    "description": "<UDF description>"
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|funcName|string|是|udf的自定义函数名称（主键），如sum,avg。【注意】函数名大小写不敏感，将统一转成小写注册|
|className |string|是|自定义udf的全路径名称com.company.biz.udf.SUM|
|jarName|string|是|udf包名称，从中解析出UDF类|
|funcDeclaration|string|否|函数定义式，用来简要表达函数的输入输出，double add(double m, double n)|
|description|string|否|自定义的函数描述|

**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/udf/funcs/str_len \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
    "jarName":"UdfCollection",
    "className": "com.qiniu.udf.StrLength",
    "funcDeclaration" : "int str_len(string s)",
    "description": "计算字符串长度，参数s代表输入字符串，返回值为字符串长度。\n 比如 str_len("abc") 返回值为 3 "
}'
```

**响应报文**

```
200 OK
Content-Type: application/json
```

or

```
409 Conflict
Content-Type: application/json
{
    "error":"funcName already exists"
}
```

### UDF函数注册删除

**请求语法**：

```
DELETE /v2/udf/funcs/<FuncName>
Content-Type: application/json
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|funcNames|array|是|udf的自定义函数名称（主键）数组，如sum,avg，批量删除|

**示例**

```
curl -X DELETE https://pipeline.qiniu.com/v2/udf/funcs/str_len \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
```

**响应报文**

```
200 OK
Content-Type: application/json
```

### UDF 函数查询

**请求语法**：

```
GET /v2/udf/funcs?from=1&size=2&sort=uploadTime:desc&jarName=xxx
Content-Type: application/json
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|from |int|否|分页条件，从第几条开始展示，不填则返回全部|
|size |int|否|分页条件，返回多少条展示，不填则返回全部|
|sort |string|否|排序字段，格式如列名:排序顺序，如sort=jarName:desc, sort=jarName:asc，如果不填排序顺序，如sort=jarName，则默认顺序排列|
|jarName |string|否|选择指定jar包对应的注册函数，jarName=a,b,c，代表查询a,b,c三个jar包的所有注册函数|
|funcName |string|否|选择指定函数名的注册函数，funcName=a,b,c，代表查询a,b,c三个函数名对应的注册函数信息|

**响应报文**

```
200 OK
Content-Type: application/json
{
    result:[
        {
            "jarName": "UdfCollection", 
            "funcName": "str_len",
            "className": "com.qiniu.udf.StrLength",
            "funcDeclaration" : "int str_len(string s)",
            "description": "计算字符串长度，参数s代表输入字符串，返回值为字符串长度。\n 比如 str_len("abc") 返回值为 3 "
        }
    ]
}
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|funcName|string|是|udf的自定义函数名称（主键），如sum,avg|
|jarName|string|是|udf包名称|
|className |string|是|自定义udf的全路径名称，com.company.biz.udf.SUM|
|funcDeclaration|string|否|函数定义式，用来简要表达函数的输入输出，如 double add(double m, double n)|
|description|string|否|自定义的函数描述|

### UDF调试

**请求语法**：

```
POST /v2/udf/debug
Content-Type: application/json
Authorization: Pandora <auth>
{
    "udfExpression": "<UDF debug expression>"
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|udfExpression |string|是|要调试的UDF的表达式，来验证udf的输出是否符合预期，如str_lenth("debug_udf"）|

**说明**
目前仅支持UDF调试，暂不支持UDAF，UDTF调试

**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/udf/debug \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 111e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
    "udfExpression": "select str_len("abcd") as a1, ucase("ab") as a2"
}'
```

**响应报文**

```
200 OK
Content-Type: application/json
{
    result:[
        {"a1":"4", "a2": "AB"}
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

UDF调试调试表达式必须符合 `select udf1(param...) as column2, udf2(param...) as column2` 格式



### UDF内置函数展示

**请求语法**：

```
GET /v2/udf/builtins?from=1&size=2&sort=funcName:desc
Content-Type: application/json
Authorization: Pandora <auth>
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|from |int|否|分页条件，从第几条开始展示，不填则返回全部|
|size |int|否|分页条件，返回多少条展示，不填则返回全部|
|sort |string|否|排序字段，格式如列名:排序顺序，如sort=funcName:desc, sort=funcName:asc，如果不填排序顺序，如sort=funcName，则默认顺序排列|
|category |string|否|选择指定类别对应的内置函数，category=窗口函数，代表查询所有窗口函数|

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|funcName|string|是|udf的内置函数名称(主键)，如sum, avg|
|description|string|否|内置的函数描述|
|category |string|否|函数分类，如日期函数，数学函数等。默认支持：match（数学函数），collection（集合函数），type_conversion（类型转换函数），date（日期函数），condition（条件函数）, string（字符串函数），udaf（聚合函数），udtf(表格函数)|


**示例**

```
curl -X GET https://pipeline.qiniu.com/v2/udf/builtins?from=1&size=2&sort=funcName:desc \
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
        "funcDeclaration" : "double cos(double a) \n bigint cos(bigint a)",
        "description": "绝对值计算",
        "category": "math"
    },
    {
        "funcName":"cos",
        "funcDeclaration" : "double cos(double a)",
        "description": "余弦值计算",
        "category": "math"
    }
    ...
]
```
