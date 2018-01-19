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


### 创建工作流Worklfow


**请求语法**

```
POST /v2/workflows/<WorkflowName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "region": <Region>,
    "comment": <Comment>
}
```

**请求内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|WorkflowName|string|是|工作流名称</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`</br>1-128个字符，支持小写字母、数字、下划线</br>必须以大小写字母或下划线开头|
|region|string|是|计算与存储所使用的物理资源所在区域</br>目前仅支持“nb”(华东区域)|
|comment|string|否|当前工作流的备注|


!> 注意:
1. `region`参数是为了降低用户传输数据的成本，请尽量选择离自己数据源较近的区域。

**示例**

```
curl -X POST https://pipeline.qiniu.com/v2/workflows/Test_Workflow \
-H 'Content-Type: application/json' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y='  \
-d '{
    "region": "nb", 
    "comment": "workflow_for_test", 
}'
```

### 查看所有工作流

**请求语法**

```
GET /v2/workflows
Authorization: Pandora <auth>
```

**响应报文** 

```
Content-Type: application/json
{
  [
    {
      "name": <RepoName>,
      "region": <Region>,
      "comment": <Comment>,
      "createTime": <CreateTime>,
      "updateTime": <UpdateTime>,
      "status": <Status>,
      "canStart": <CanStart>
    },
    ...
  ]
}
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|name|string|-|当前工作流名称|
|region|string|-|当前工作流所在区域|
|comment|string|-|当前工作流注释|
|createTime|string|-|当前工作流创建时间|
|updateTime|string|-|当前工作流更新时间|
|status|string|-|当前工作流状态|
|canStart|bool|-|当前工作流是否容许启动，只有当存在实时计算、消息队列的导出或者离线计算的导出时，工作流才容许启动|



### 根据名称查询工作流Worklfow

**请求语法**

```
GET /v2/workflows/<WorkflowName>
Authorization: Pandora <auth>
```

**响应报文** 

```
Content-Type: application/json
{
    "name": <RepoName>,
    "region": <Region>,
    "comment": <Comment>,
    "createTime": <CreateTime>,
    "updateTime": <UpdateTime>,
    "status": <Status>,
    "canStart": <CanStart>,
    "nodes": {
       "<NodeID>": {
         "name": "<NodeName>",
         "type": "<NodeType>",
         "data": {
           <NodeData>
         },
         "parents": [{"name": <parentName>, "type": <parentType>},...],
         "children": [{"name": <childName>, "type": <childType>},...]
       },
      ...
    }
}
```

**响应内容**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|name|string|-|当前工作流名称|
|region|string|-|当前工作流所在区域|
|comment|string|-|当前工作流注释|
|createTime|string|-|当前工作流创建时间|
|updateTime|string|-|当前工作流更新时间|
|status|string|-|当前工作流状态|
|canStart|bool|-|当前工作流是否容许启动，只有当存在实时计算、消息队列的导出或者离线计算的导出时，工作流才容许启动|
|NodeID|string|-|工作流中当前节点ID，格式为"<NodeName>-<NodeType>" |
|NodeName|string|-|工作流中当前节点的名称|
|NodeType|string|-|工作流中当前节点的类型，取值范围为`repo`，`transform`，`export`，`datasource`，`job`和`exportBatch`|
|NodeData|json|-|工作流中当前节点的配置`Spec`|
|parentName|json|-|等价于工作流中当前节点的父节点名称|
|parentType|json|-|等价于工作流中当前节点的父节点类型，取值范围等同于<NodeType>|
|childName|json|-|工作流中当前节点的子节点名称|
|childType|json|-|工作流中当前节点的子节点类型，取值范围等同于<NodeType>|

**说明**
* 对于`消息队列`节点，`NodeType`为`repo`。
* 对于`实时计算任务`节点，`NodeType`为`transform`。
* 对于`实时导出任务`节点，`NodeType`为`export`。
* 对于`离线数据源`节点，`NodeType`为`datasource`。
* 对于`离线计算任务`节点，`NodeType`为`job`。
* 对于`离线计算导出`节点，`NodeType`为`exportBatch`。


### 启动工作流

**请求语法**

```
POST /v2/workflows/<WorkflowName>/start
Authorization: Pandora <auth>
```

!> 注意：只有当工作流中存在实时计算、消息队列的导出或者离线计算的导出时，当前工作流才容许启动。


### 停止工作流

**请求语法**

```
POST /v2/workflows/<WorkflowName>/stop
Authorization: Pandora <auth>
```



### 根据名称删除工作流

**请求语法**

```
DELETE /v2/workflows/<WorkflowName>
Authorization: Pandora <auth>
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
|400  |E18134: schema数量超出限制|
|400  |E18135: 给定的schema数量限制少于当前已经存在的schema数量|
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
|401  |bad token：鉴权不通过 、token已过期、机器时间未同步|
|400  |E18639: 工作流名称不合法|
|409  |E18640: 工作流已存在|
|404  |E18641: 工作流不存在|
|400  |E18642: 工作流的格式非法|
|400  |E18644: 当前工作流禁止启动|
|400  |E18645: 当前工作流禁止停止|

