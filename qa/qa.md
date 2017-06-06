# 常见问题汇总

本页面汇集了使用Pandora遇到的常见问题，如您的疑问没有在此页面展示，请在对应的客服群中提出，或将您的疑问发送邮件至 pandora[AT]qiniu.com 请在邮件中注明您的公司及联系方式。

## Pandora产品

> 如何开通试用Pandora大数据平台产品?

目前Pandora平台产品处于有限开放、免费试用阶段，可以联系七牛的销售或客服申请开通试用，也可以发送邮件给 pandora[AT]qiniu.com 注明您的公司名称及联系方式，申请试用。我们收到申请后将在一个工作日内会为您审核。

> 试用Pandora需要申请开通哪些服务？

目前Pandora平台包括大数据相关服务，以及应用市场的相关应用，需要申请开通大数据服务以及应用市场相关应用。

> Pandora是否支持私有部署？

全套Pandora产品均支持私有部署。

> 七牛官网开通Pandora服务后，点进去Workflow页面什么都看不到？

请尝试使用其他版本浏览器或者更新您的浏览器版本。

## logkit数据上报工具

> 如何快速试用logkit？

可以查看我们的[logkit快速开始](https://qiniu.github.io/pandora-docs/#/quickstart/workflow?id=%e4%bd%bf%e7%94%a8logkit%e5%90%91%e5%b7%a5%e4%bd%9c%e6%b5%81%e4%b8%ad%e6%8e%a8%e9%80%81%e6%95%b0%e6%8d%ae)文档。

> 在哪里可以查看logkit的代码及详细的文档？

logkit是七牛Pandora开源的日志上报工具，github仓库的地址为：https://github.com/qiniu/logkit, 同时在github上有详细的wiki文档： https://github.com/qiniu/logkit/wiki。

欢迎向logkit提出建议(创建Issues)，并贡献代码(Pull Request)，我们会第一时间review，并merge通用的功能。

> logkit支持哪些数据源数据的上报？

logkit目前支持文本文件、Kafka、MongoDB、ElasticSearch、MySQL、MicroSoft SQL Server多种数据源的数据上报。

> 我配置好了logkit为什么没有数据进到Workflow？

* 首先检查logkit的运行日志中是否有`ERROR`字样的日志出现，若出现，则根据错误日志检查是什么原因造成的，若无法理解错误内容，请联系我们，或在`https://github.com/qiniu/logkit`创建issue。
* logkit的配置中`read_from`如果设置为`newest`，则从日志的最新位置开始，若logkit启动后没有新日志写入，则logkit不会发送日志
* logkit发送到Pandora时会检查Pandora工作流中的数据源是否创建，若没有创建，则无法发送，请使用 `padnora_auto_create`字段，自动创建repo，或者前往Pandora工作流创建。
* logkit 的Pandora Sender中，`pandora_schema`如果填了，只选择填了的字段打向pandora，注意是否填错。

> logkit 上报数据是否可以控制带宽流量？

在[Pandora Sender](https://github.com/qiniu/logkit/wiki/Pandora-Sender)中，使用`flow_rate_limit`可以限制每秒发送流量大小；使用`request_rate_limit`可以限制每秒请求数。流量限制的算法为[bucket token算法](https://en.wikipedia.org/wiki/Token_bucket)。

> logkit 上报数据如何节省带宽？

在[Pandora Sender](https://github.com/qiniu/logkit/wiki/Pandora-Sender)中，开启`pandora_gzip`选项可以使用gzip压缩的形式发送数据。gizp压缩的比例在5:1左右，能节省大量带宽，但是对CPU有一定消耗，请权衡使用。

> logkit 配置文件错误如何校验？

我们提供一个自动生成配置文件的服务，可以在这个[logkit配置自动生成](http://l28nlm49.nq.cloudappl.com/)页面自动生成配置文件。

我们的配置文件都是以json格式存储的，所以一般情况下需要先检查配置文件是否符合正常的json格式，比如多个逗号，少个逗号，中文符号的逗号等等问题。推荐使用https://jsonformatter.curiousconcept.com/ 进行json配置文件的校验。

> logkit 如何配置同时收集多个文件的日志数据？

使用 [file reader](https://github.com/qiniu/logkit/wiki/File-Reader)中的tailx模式可以同时读取多个文件的日志数据。

> logkit `read_from`参数中`newest`指的是读取最新文件里边的所有内容还是指读取最新的文件里边新增加的内容？

是指读取最新的文件里边新增加的内容？

> logkit 上报的时间类型数据为什么和实际的时间偏差8小时？

如果原始数据的时间字段中没有包含时区，logkit会把日志记录的时间当成UTC时间来收集。grok parser提供对收集的时间进行校正的功能，首先需要制定grok pattern收集的字段类型为date，然后使用`timezone_offset`，写"-8"，修正回CST中国北京时间。
文档参见: https://github.com/qiniu/logkit/wiki/Grok-Parser

> logkit 收集的文本文件日志是单行还是多行？

logkit收集日志默认情况下是单行，也支持读取多行。在读取多行时，需要配置一个辨别分隔符的一个正则表达式，具体参见：[file reader](https://github.com/qiniu/logkit/wiki/File-Reader)中的`head_pattern`参数。

> logkit这个可以设置监听的IP吗？

在logkit主配置文件中，`bind_host` 选项可以配置监听的host及端口地址，默认不填则随机选择一个4000及以上的可用端口监听localhost。

> logkit 以怎样的频率发送日志？

logkit读取日志的平率取决于三个参数, `batch_len`(batch大小,单位byte),`batch_size`(日志条数),`batch_interval`(发送时间间隔), 三者满足其一就会发送出一批。

> logkit收集的速度慢于日志产生的速度，导致数据收集产生大量延迟如何解决？

延迟的产生有多方面原因，包括发送效率低，解析速度慢两大原因。

* 解析速度慢取决于使用的解析器效率，尤其是使用Grok Pattern，基于正则表达式的解析，建议测试一下pattern的benchmark，也可以考虑使用性能更好的pattern解析日志。

* 提供发送效率方面，又分为提高每个请求的数据量，以及并发发送两部分。
  * [增大请求的数据量](https://github.com/qiniu/logkit/wiki/Runner%E9%85%8D%E7%BD%AE)就是增加runner的`batch_size`参数，提高每批发送的数据大小。注意batchsize的上限为2MB，Pandora默认接受的单个请求最大数据量为2MB。
  * [并发发送](https://github.com/qiniu/logkit/wiki/Senders)，则在Sender中设置`fault_tolerant`为"true"，且`ft_strategy`设置为`always_save`，同时设置`ft_procs`加大并发，`ft_procs`设置越大并发度越高，发送越快，对机器性能带来的影响也越大。并发发送的原理是构建一个磁盘队列，然后在队列读取的那一端并发发送，所以也需要设置`ft_save_log_path`目录，存放磁盘队列中的数据。

* 最后要注意，如果`ft_procs`增加已经不能再加大发送日志速度，可以看下磁盘写入速度，默认情况下logkit磁盘写入速度限制为10MB/s，若达到限制就需要加大`ft_write_limit`限制，为logkit的队列提升磁盘的读写速率。

> logkit是否会重复发送数据？

logkit发送数据是保证不重不漏的，但是在读文件时，错误的配置文件可能导致重复读取文件。主要是reader为tailx模式时，需要注意重复的情况，具体参见[file reader文档](https://github.com/qiniu/logkit/wiki/File-Reader)。

> logkit可以删除收集完毕的历史日志吗？

可以，请使用[cleaner功能](https://github.com/qiniu/logkit/wiki/Cleaner)

> logkit可以直接在使用时创建Pandora的Repo吗？

可以，使用`pandora_auto_create`即可创建，参见[文档](https://github.com/qiniu/logkit/wiki/Pandora-Sender)

> 我要收集的字段是时间相关的，logkit该怎么配置？

首先选择对应的parser，如grok parser直接支持在pattern里填写转义为时间数据；若parser不支持获取时间，则可以解析为字符串。

无论Parser中解析到的是字符串还是时间，时间类型的数据请在Pandora工作流中创建的数据源指定字段为时间类型(date)，logkit的Pandora Sender中会将解析到的字符串、数字做一次类型转换，转为时间类型。目前支持的类型转换[参见文档](https://github.com/qiniu/logkit/wiki/Pandora-Sender#%E5%B8%B8%E8%A7%81%E5%8A%9F%E8%83%BD)

> logkit显示无法解析我日志中的字段为时间类型该怎么办？

在logkit主配置文件中可以自定义时间类型的格式，参见[文档](https://github.com/qiniu/logkit/wiki/logkit%E4%B8%BB%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)

## 工作流(Workflow)

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

### 导出

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

> 导出是报错不明白什么意思，去哪里查看一个更详细的说明？

我们将尽快整理有个详细的导出错误说明文档。

### 计算任务

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


## 日志检索服务(LogDB)

> 日志服务中是否会出现数据重复？

日志搜索在正常情况下不会出现重复数据，在资源不足时会由于请求超时导致部分数据产生重复，重复数据不影响正常的搜索。如需保证日志服务中完全不出现重复数据，请联系我们进行相关VIP配置。

> 日志检索服务的搜索API中如何指定精确到秒的时间范围？

日志检索的文档中包含了[时间范围的指定参数设置方法](https://qiniu.github.io/pandora-docs/#/api/logdb?id=%e6%9f%a5%e8%af%a2%e6%97%a5%e5%bf%97), 例如在请求参数中加
`date:[2017-01-01T120027.87+08:00 TO 2017-02-01T120027.87+08:00]`，注意时间的格式为RFC3339格式。

> 日志检索这边直接用中文作为关键字搜索是搜不了的吗?

日志检索默认使用标准分词器，对于中午，每一个中文字符就会进行分词，比如搜索 `title:*大数据*`，可能搜索不到，但是搜索 `title:大 AND title:数 AND title:据`，就可以搜到。
如果要精确匹配中文的大数据，那么就需要在创建repo时指定字段为中文分词器。
也可以选择不使用分词器，这样默认不使用倒排索引，搜索的性能会大打折扣，但是可以使用`title:*大数据*`这样的方式搜索包含中英文字符及标点符号的文本。类似于Linux Shell的grep方式。

> 日志检索的API接口搜索如何使用高亮功能？

可以在文档中查看[搜索结果高亮示例](https://qiniu.github.io/pandora-docs/#/api/logdb?id=%e6%9f%a5%e8%af%a2%e6%97%a5%e5%bf%97),使用API的查询高亮功能。

> 日志检索界面每次只能查找200条结果，我想要查找更多数据怎么办？

可以使用API查找数据，同时我们也提供日志检索的命令行工具logctl，我们已经将该工具开源于[github](https://github.com/qiniuts/logctl)

> 日志检索数据存储时限为一个月，是一个月之前的数据会删除吗？

是的，会按照存储时限，删除过期的数据。


## 时序数据库服务(TSDB)

## Xspark

> Xspark资源监控页面的grafana密码是什么？

grafana默认密码，我们xspark的域名都是阅后即焚的url，不支持跨浏览器的，很安全。放心使用。

## Pandora SDK

> Pandora有哪些语言的SDK？

目前Pandora的SDK主要有 go、java、php、python四种。其中go语言的sdk是最全的，java包含主要的打点和logdb查询的API，而php和python主要支持打点的API。

需要别的语言的API或者当前提供的sdk中不支持您需要的某些API，可以联系我们支持。

> Golang SDK有什么创建Pandora数据源的简单方法吗？

golang SDK 创建工作流数据源支持DSL语言，其创建语法参见 logkit `pandora_auto_create`的写法, 参见[文档](https://github.com/qiniu/logkit/wiki/Pandora-Sender)

## Grafana/Kibana

> 应用市场上的Grafana、Kibana应用如何开通

应用市场上的应用都需要单独申请开通，请联系我们。

> 应用市场的Kibana很多功能不能用？

应用市场的Kibana是一个精简版的Kibana，主要用于用惯了Kibana的用户可以平滑的使用我们的日志检索产品，后续会开放更多Kibana的功能。如紧急需要某些功能，请联系我们。

## 计费

请联系我们。