> Pandora数据源数据上报频率有限制吗？

数据源数据上报的频率没有限制，但是收费跟上报频率有关，同时批量上报数据也有利于传输的性能，建议批量上报数据。但是一个批次的数据压缩前大小不能大于2MB，也支持使用gzip压缩后上报数据，便于节省带宽。

> Workflow支持更新字段吗？

Workflow只支持新增字段。不支持修改字段。

> Workflow前期调试该如何操作最快？

前期调试建议使用Workflow的复制粘贴功能调试，每次复制后，跨界面粘贴到一个新的数据源后面，等到调试完毕后再将多余的Workflow删除。

> 打向Workflow数据源的数据，如果不创建任何导出，保留时长是多久？

最多保留2天，建议尽快导出。

> 界面上出现服务器内部错误是什么情况？

出现服务器内部错误时，请联系我们。

> 数据源中查看数据，发现查看到的数据与机器日志的实际数据有大量延迟是什么原因？

数据源中的数据是Pandora接收到的最新上报的数据，若发现数据源中数据有延迟，首先查看是否现在有的机器上存量数据太多，导致收集出现延迟。 关于使用logkit处理收集导致的延迟，请参见FAQ中logkit部分。

> 如果我们使用了离线工作流，应用市场，还需要开 xspark 吗？

xspark其实是个独立的spark服务，集成了zeppelin可视化。 如果离线workflow能完成你们需求，那么是没必要用xspark的。

> 离线workflow的资源是弹性的吗？能动态改变计算资源么？

这个暂时还不支持。 但可以随时扩容缩容。

> 多个离线任务是共享资源池还是独占的？

考虑到任务间的计算强度差异可能非常大，如果放在一个大的资源池中，可能对用户在某些场景下有误导。如，任务ABCDE，用户可能B任务sql写错了，导致任务运行时间特别特别长，如果用共享资源池，那用户很难一眼看出任务B不正常。所以共享资源池我们后续应该会支持，但是应该对用户有更有好的提示时，才会开放。目前每个任务一个独占的资源。

> 离线任务是否会导致资源浪费？比如，一天下来可能就跑一次半小时的任务。

离线任务有调度模块，用户可以选择 loop, crontab等方式运行，完全不需要一直开着。跑完任务后，会按照设置的调度策略，释放资源，等待下次调度启动。实时计算是一直占着的资源的。


> 导出到对象存储的bucket选择哪个区域？

目前导出到对象存储的bucket区域只支持`华东`。其他区域无法正常导出。

> 导出到对象存储后多久能在对象存储查看到导出的文件？导出到对象存储没有数据堆积，但是确没有看到对象存储的bucket里面有文件出现？

导出到对象存储默认攒一个较大的数据量生成一个文件，避免出现海量小文件影响性能。在数据量小的情况下，导出间隔最长为10分钟。

> 数据源和日志检索之间数据堆积，如果发现很大，要怎么办呢？

堆积达到一定的条件，我们程序会去自动处理堆积。

> 创建导出到日志检索后，多久能查询？

刚创建导出，系统有一个初始化的时间，约有一分钟的延迟。

若导出没有错误，正常情况导出的数据在30s内可以在日志检索查看。

> 导出到日志检索出现导出错误？

导出到日志检索目前在创建导出时严格的类型检查，通常导出的错误主要包含以下情况：

1. `string`类型的数据导出到日志检索的时间(`date`)类型，logdb的date类型只支持`RFC3339`的格式，在数据源为date类型时，可以正确导出，请正确配置数据源的字段类型。
2. `string` 类型导出到logdb 的long类型，也是不支持的。

> 导出到对象存储时填写的账号和公钥指什么？

账号填写的是您注册七牛的邮箱账号，公钥指的是七牛的ak/sk中的ak。

> 导出到对象存储的文件前缀应该怎么写？

导出到对象存储的文件前缀应该写一个利于XSpark调用的方式，建议前缀中包含魔法变量的年月日，如`serviceName/date=$(year)-$(mon)-$(day)/hour=$(hour)/min=$(min)/$(sec)`

> 创建导出字段都是自动填写的为什么提示创建失败？

请检查导出名称，不能有横杠等特殊字符，仅支持数字、字母、下划线。

> 如果为消息队列建立了一个导出到 HTTP 的任务, 如果 http 请求失败会重试吗？会不会重复投递数据呢？

http导出会重试，导出任务看收到的返回码为`200`才会换下一批数据导出。

> 从消息队列导出到 http，那个 http 接口我们要依据那个文档开发呢？

可以参见[数据推送部分](https://qiniu.github.io/pandora-docs/#/api/pipeline?id=%e6%95%b0%e6%8d%ae%e6%8e%a8%e9%80%81)的文档，我们http 导出的数据格式，跟我们收集日志时的接口格式一致。

> 导出时显示错误信息为 `write failed`，其他什么都没有，是什么情况？

`write failed`表示资源不够，存在数据写不进去的情况，**对于非测试用户**，系统会自动调整资源，一般正常的话系统会自动恢复，无需在意。若长久出现该错误，产生大量数据堆积，请联系我们快速增加资源解决数据堆积问题。

> 导出至http服务器地址以及请求资源路径要怎么填？

服务器地址（ip或域名）,例如:`https://pipeline.qiniu.com` 或 `127.0.0.1:7758`
请求资源路径（具体地址,不包含ip或域名）,例如:`/test/repos`

导出到HTTP的详细参数说明可以参见文档站[导出数据至HTTP地址](https://qiniu.github.io/pandora-docs/#/api/pipeline?id=%e5%af%bc%e5%87%ba%e6%95%b0%e6%8d%ae%e8%87%b3http%e5%9c%b0%e5%9d%80)部分。

> 导出到的http服务构建在七牛提供的evm虚拟机上，指定公网IP绑定到evm，为何导出不成功？

Pandora和evm之间在同一机房默认走内网通道，若要联通，请联系我们打通公网IP通道。

> 导出到 http 时，post 的数据是什么格式？

默认情况下导出的是`key=value`的文本格式，跟打点时格式是一致的。也可以选择以json的格式导出。

> 导出到 http 时，选用json格式导出，不存在一个 json 多行的情况吧？

不会，一行一个json，数据中的换行符会被转义。

> 创建 http导出时，经常出现校验导出失败， 是什么原因？

导出校验失败通常是服务器地址有问题，创建前请确认服务器地址能否ping通，及请求资源路径是否存在。如若可以ping通，并且还是校验失败，请联系我们。

> http的导出多久一次，多少数据量一个batch?

目前导出固定每次20000条，导出的数据量依据当前每条数据大小而定。 我们正在上线调整数据量的功能。

> 导出时报错不明白什么意思，去哪里查看一个更详细的说明？

我们将尽快整理有个详细的导出错误说明文档。

> 自定义计算中，编写的时间类型(date)对应的java/scala类型是什么？

对应的类型为 `java.sql.Timestamp`.

> 创建计算任务时从数据源取到的数据是什么时间点开始的数据，是历史的所有数据吗?

计算任务目前暂时只支持从最新的数据开始计算。未来会增加从最老的数据开始计算的功能。最新指的是 在创建计算任务之后，进入数据源的新数据。

> 创建实时计算任务，sql里面需要带 time时间条件吗，如果不带时间条件，创建的计算任务会自己只计算增量的数据吗?

计算任务每次都只会计算增量的数据。

> 计算任务是否会导致数据重复？

除非计算任务报错并执行失败，否则计算任务不会出现任何数据重复。

> 计算任务中sql计算有没有提供生成唯一值uuid的函数?

sql的计算任务暂时没这个函数，使用自定义计算可以完成这个功能，也可以在logkit中增加生成uuid项。

> 计算任务中增加了字段后，怎么样获取原来数据源中的原始字段？

这个只要在自定义计算里面把原来的数据源的值也写上就行了，就相当于在OutputData里面也填上SourceData的那些字段。

> 指定了OutputData中的字段，为什么在工作流界面推导字段时不生效？

这种情况可能是OutputData包名写错了，我们的OutputData和SourceData都要在`userWrite.bean`这个包(package)下面，是固定的，不能改变。

> 希望客户端APP可以上报采集数据，如何生成打点的token鉴权？

可以在页面 http://pandora-toolkits.qiniu.com/auth 生成token鉴权，生成的token鉴权保证只要数据推送接口可以通过鉴权，其他接口均无法使用。
