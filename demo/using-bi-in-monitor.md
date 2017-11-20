#### 使用BI Studio工具对服务质量进行报表展示

BI Studio是Pandora用于报表展示的工具，文档详见[BI Studio](https://qiniu.github.io/pandora-docs/#/quickstart/report)

在运维监控中，通常需要对某段时间的服务质量进行汇总展示，并以此为依据对服务质量进行改善。

在之前的几篇文章中，我们详细介绍了使用pandora对[nginx日志进行监控和搜索](https://qiniu.github.io/pandora-docs/#/demo/nginxlog)、[对服务器性能监控](https://qiniu.github.io/pandora-docs/#/demo/monitoring)，这篇文章我们详细介绍一下使用pandora bi-studio对nginx服务质量进行报表展示的流程。

最终的效果图如下：

![](http://pandora-kibana.qiniu.com/bi-demo-result-3.png)

##### 收集nginx数据

参考[nginx日志进行监控和搜索](https://qiniu.github.io/pandora-docs/#/demo/nginxlog)中使用logkit收集日志的相关配置。

在配置成功后，portal上会出现一个数据源`nginx`，其字段如下：

```

http_x_forwarded_for string
host string
request string
machine string
remote_user string
remote_addr string
request_time float
bytes_sent long
http_user_agent string
status long
body_bytes_sent long
upstream_addr string
time_local date
http_referer string
sent_http_x_reqid string

```

##### 创建计算任务

在数据成功打入pandora工作流之后，接下来的一步是创建计算任务，对打入的数据进行聚合，以便更好的在报表中进行展示。

报表展示不同于实时监控，实时监控需要每个具体的值，以便能对异常值进行监控和报警；而报表展示关注的某一段时间的平均值或者整体值，所以在导出到报表之前进行数据汇聚能有限减少成本。

通常计算任务的具体逻辑取决于最后想要呈现的报表，在这篇文章中，我们关注如下报表指标

1. 全天API的调用次数
2. 不同状态码的数量以及占比
3. 总体响应时间 & 不同状态码的响应时间
4. 按照机器分布的机器流量

根据以上我们关注的指标，配合数据源字段，我们创建如下计算任务

```
SELECT
  count(*) as total,
  time_local,
  host,
  status,
  machine,
  mean(request_time) as mean_request_time,
  sum(bytes_sent) as sum_bytes_sent,
  sum(body_bytes_sent) as sum_body_bytes_sent
FROM
  stream
GROUP BY
  host,
  status,
  machine,
  time_local

```
以上计算任务根据机器、host、状态码和时间对请求时间(request_time)，请求体(body_bytes_sent)和请求返回的数据(bytes_send)进行了汇聚。

![](http://pandora-kibana.qiniu.com/bi-demo-transform.png)

其中运行间隔可以根据自己的需求来选择，在这篇文章中，我们选择10分钟。


##### 创建导出任务

在工作流上创建导出到报表的任务

![](http://pandora-kibana.qiniu.com/bi-demo-export-report.png)

只需要在工作流界面上选择导出到report即可。

##### 创建报表

单张图表是报表系统的最小单位，仪表盘是根据业务对多个报表的集合，为此我们首先创建一个空的仪表盘。

![](http://pandora-kibana.qiniu.com/bi-demo-dashboard-1.png)

![](http://pandora-kibana.qiniu.com/bi-demo-dashboard-2.png)

在创建了一个空的仪表盘之后，接下来需要创建报表填充到这个仪表盘中。点击单个数据表出现创建报表的页面，关于这个页面的详细操作文档可以参见[文档](https://qiniu.github.io/pandora-docs/#/quickstart/report)

![](http://pandora-kibana.qiniu.com/bi-demo-edit.png)

![](http://pandora-kibana.qiniu.com/bi-demo-edit-table.png)

###### 全天API的调用次数

这个图表，我们选择`数字和趋势线`这种图表类型

![](http://pandora-kibana.qiniu.com/bi-demo-api.png)

点击`执行`，然后保存到仪表盘中

![](http://pandora-kibana.qiniu.com/bi-demo-save.png)

这个时候，我们转到nginx这个仪表盘中，可以看到刚才的图表已经成功展示。

![](http://pandora-kibana.qiniu.com/bi-demo-dashboard-3.png)

##### 不同状态码的数量以及占比

通常我们是用饼状图来表示占比

点击nginx数据表，出现编辑图表的页面

![](http://pandora-kibana.qiniu.com/bi-demo-pie-1.png)

以上是状态码的数量，点击标签类型可以切换成按照比例显示。

![](http://pandora-kibana.qiniu.com/bi-demo-pie-2.png)


##### 不同状态码的响应时间和总体响应时间

我们用时间序列图按天来展示总体的响应时间和不同响应码的响应时间。

![](http://pandora-kibana.qiniu.com/bi-demo-series-1.png)

上图显示的为不同响应码的响应时间；
在配置项中去掉group by的字段，即为总体的响应时间

##### 按照机器分布的机器流量

nginx流量可能分布在不同的机器，展示不同机器的的流量用时间序列柱状图


![](http://pandora-kibana.qiniu.com/bi-demo-series-2.png)



##### 数据联动

对于一个仪表盘中所有的图表，一旦图表制作完成既保持不变，为此报表系统提供了一种`提示器`的图表类型，这种类型的图表能够使一个仪表盘中的图表根据提示器中不同的过滤条件而动态变化。

根据上面的步骤我们已经做出如下所示的仪表盘

![](http://pandora-kibana.qiniu.com/bi-demo-result-1.png)


如果我们想显示不同的host，或者不同的机器，或者不同时间段的报表怎么办？

增加一个`提示器`的图表。

![](http://pandora-kibana.qiniu.com/bi-demo-result-2.png)

第一个名为过滤器的图表可以进行一些选择，这个仪表盘的所有图表都会根据过滤器中的条件进行联动。

最终的效果如下

![](http://pandora-kibana.qiniu.com/bi-demo-result-3.png)

