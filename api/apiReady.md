### 什么是签名
签名是七牛服务器用来识别用户身份与权限的凭证，我们采用AK/SK(公钥/私钥)、token两种方式来对用户进行身份验证。
### 设计目的
* 对用户身份进行认证，使服务端获知该请求的发送者。
* 防止身份假冒。
* token分发，使用户可以在自己的应用服务器和app之间分发token，达到保护AK/SK的目的。

### 制作签名
##### AK/SK签名包含的内容

|参数|是否必须|说明|
|:---:|:---:|:---|
|method|是|Http method，请求类别|
|content-md5|否|Http content-md5，验证数据完整性|
|content-type|否|Http content-type，描述资源类型|
|date|是|当前时间，要求为GMT格式，如`Sun， 06 Nov 1994 08:49:37 GMT`|
|qiniu-headers|否|表示请求头中的一些自定义参数，以`X-Qiniu-`开头|
|resource|是|目标资源（uri）|

##### 制作AK/SK签名

###### 1.生成待签名的原始字符串

```
strToSign = <method> + "\n"
             + <content-md5> + "\n"
             + <content-type> + "\n"
             + <date> + "\n"
             + <canonicalizedqiniuheaders>
             + <canonicalizedresource>
```

###### 2.使用`SecretKey`对上一步生成的`strTosign`计算`HMAC-SHA1`签名

```
sign = hmac_sha1(strToSign， "<SecretKey>")
```

###### 3.对`sign`进行`URL安全的Base64编码`

```
encodedSign = urlsafe_base64_encode(sign)
```

###### 4.将`AccessKey`和`encodedSign`用英文符号`:`连接起来

```
auth = "<AccessKey>:<encodedSign>"
```

!> 注意：签名字符串中的`content-md5`和`content-type`为空那么相应的位置用空字符串来占位。`Date`参数与服务器时间的偏差不得超过15分钟，用户需要同步校准自己的时钟。频繁返回`401`状态码时请先检查`Date`相关的代码逻辑。


##### token签名包含的内容

|参数|是否必须|说明|
|:---:|:---:|:---|
|method|是|Http method，请求类别|
|content-md5|否|Http content-md5，验证数据完整性|
|content-type|否|Http content-type，描述资源类型|
|expires|是|过期时间，unix秒级时间戳|
|qiniu-headers|否|表示请求头中的一些自定义参数，以`X-Qiniu-`开头|
|resource|是|目标资源（uri）|

##### 制作token签名

###### 1.构造`tokenDescription`

```
tokenDescription = 
'{
  "resource": <Resource>，
  "expires": <Expires>
  "contentType": <ContentType>，
  "contentMD5": <ContentMD5>，
  "method": <Method>，
  "headers": <Headers>
}'
```

###### 2.将`tokenDescription`进行`URL安全的Base64编码`，得到`encodedTokenDescription`

```
encodedTokenDescription = urlsafe_base64_encode(tokenDescription)
```

###### 3.使用`SecretKey`对`encodedTokenDescription`计算`HMAC-SHA1`签名

```
sign = hmac_sha1(encodedTokenDescription， "<SecretKey>")
```

###### 4.对`sign`进行`URL安全的Base64编码`

```
encodedSign = urlsafe_base64_encode(sign)
```

###### 5.将`AccessKey`、`encodedSign`和`encodedTokenDescription`用英文符号`:`连接起来

```
auth = "<AccessKey> + : + <encodedSign> + : + <encodedTokenDescription>"
```

### 签名用法示例


```
POST /v4/repos/<RepoName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "region": <Region>，
    "metadata":{
        "key1":"value1"，
        ...
    }
}
```

#### CanonicalizedQiniuHeaders计算方式：

以`X-Qiniu-`开头的header是七牛的服务自定义的头部，有其特殊意义，因此签名中也需要加进去所有的自定义头部，`CanonicalizedQiniuHeaders`的计算步骤如下：

1. 将所有以`X-Qiniu-`为前缀的HTTP请求头的名字转换成小写字母，例如`X-Qiniu-pipeline-timeout: 20`转换成`x-qiniu-pipeline-timeout:20`；

2. 将上一步得到的所有的HTTP请求头按照名字的字典序进行升序排列；

3. 删除请求头和内容之间分隔符两端出现的空格；

4. 将每一个头和内容用`\n`分隔符分隔拼成最后的`CanonicalizedQiniuHeaders`。

!> 注意：当不存在Qiniu headers的时候无需添加最后的换行符。

#### CanonicalizedResource计算方式：

用户希望访问pipeline的目标资源被称为`CanonicalizedResource`，计算步骤如下：

1. 将`CanonicalizedResource`置为空字符串（""）；

2. 将请求的pipeline资源的uri放入`CanonicalizedResource`，例如`/v2/repos/repox/exports/exportx`；

3. 如果请求的资源包含了子资源，那么将子资源按照字典序升序排列并以`&`为分隔符生成子资源，以`?`为分割符追加在`CanonicalizedResource`字符串的后面，例如`/v2/repos/repos/repox?q1=v1&q2=v2`；







