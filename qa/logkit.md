> 如何快速试用logkit？

可以查看我们的[logkit快速开始](https://qiniu.github.io/pandora-docs/#/util/logkit?id=logkit)文档。

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

