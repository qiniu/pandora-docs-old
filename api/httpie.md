### httpie

Pandora服务使用七牛统一的签名鉴权服务，我们针对开源的[httpie](https://github.com/jakubroztocil/httpie)服务，增加了七牛的签名认证功能，使用httpie来发送API请求非常便捷。

Pandora httpie github地址：`https://github.com/kirk-enterprise/httpie`

### 安装命令行工具


在命令行中输入以下内容安装:

```
pip install --upgrade https://github.com/kirk-enterprise/httpie/tarball/master
```

> 小提示：
> `pip` 是一个Python包管理工具，主要是用于安装 PyPI 上的软件包，使用pip需要保证您的电脑上有python环境，并且已安装pip。
> 
> `pip` 的简单安装方法： `sudo easy_install pip`

!> 注意：若提示安装失败，可在命令行头部加上 `sudo` 命令

安装后的工具命令为 `http`。

### 使用httpie

#### 发送POST请求

1. 发送POST请求需要先将请求的body写在文件中

```
echo '{
    "region": "nb",
    "schema": [
        {
            "key": "userName",
            "valtype": "string"
        },
        {
            "key": "age",
            "valtype": "float"
        },
        {
            "elemtype": "long",
            "key": "addresses",
            "valtype": "array"
        },
        {
            "key": "profile",
            "schema": [
                {
                    "key": "position",
                    "valtype": "string"
                },
                {
                    "key": "salary",
                    "valtype": "float"
                },
                {
                    "elemtype": "string",
                    "key": "education",
                    "valtype": "array"
                }
            ],
            "valtype": "map"
        }
    ]
}' > body.json

```

2. 发送POST请求 

```
http --verbose --ak=<qiniu_ak> --sk=<qiniu_sk> --auth-qiniu-type=pandora/mac POST https://pipeline.qiniu.com/v2/repos/testdemo < body.json
```

得到返回:

```
HTTP/1.1 200 OK

{}
```

此时我们创建了一个名为`testdemo`的消息队列；
POST请求发送和返回完成。

#### 发送GET请求

```
http --ak=<qiniu_ak> --sk=<qiniu_sk> --auth-qiniu-type=pandora/mac GET https://pipeline.qiniu.com/v2/repos/testdemo
```

得到返回

```
HTTP/1.1 200 OK

{
    "derivedFrom": "",
    "group": "",
    "region": "nb",
    "schema": [
        {
            "key": "userName",
            "required": false,
            "valtype": "string"
        },
        {
            "key": "age",
            "required": false,
            "valtype": "float"
        },
        {
            "elemtype": "long",
            "key": "addresses",
            "required": false,
            "valtype": "array"
        },
        {
            "key": "profile",
            "required": false,
            "schema": [
                {
                    "key": "position",
                    "required": false,
                    "valtype": "string"
                },
                {
                    "key": "salary",
                    "required": false,
                    "valtype": "float"
                },
                {
                    "elemtype": "string",
                    "key": "education",
                    "required": false,
                    "valtype": "array"
                }
            ],
            "valtype": "map"
        }
    ]
}
```





