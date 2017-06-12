### 创建XSpark应用

**操作流程：**

在七牛控制台最下方的`应用市场`进入**XSpark**首页。

然后点击**新建部署**来创建我们第一个XSpark应用。

填写相应内容之后，点击**确定创建**，等待系统部署完成。

**填写参数说明：**

|参数|必填|说明|
|:--|:---|:--|
|应用名称|是|用来标识该应用的唯一性|
|应用别名|否|备注标识，支持中文，没有特殊含义|
|部署区域|是|计算与存储所使用的物理资源所在区域；</br>为了降低成本，请尽量选择离数据源较近的区域|

**操作演示：**

![](_media/xspark-deploy.gif)


### 初始化配置

**操作流程：**

部署完成后，我们需要选择此应用的`Worker`规格以及数量，选择完成后点击**提交**按钮，等待应用初始化完成即可。

**填写参数说明：**

|参数|必填|说明|
|:--|:---|:--|
|Worker规格|是|不同大小的`CPU`与`内存`|
|Worker数量|是|选择的`Worker规格`使用数量|

!> Worker的概念： Spark集群的执行节点，用来执行**分布式计算任务**，一般来说，数量越多执行效率越高，规格与数量的大小与数量和执行效率成正比，请酌情选择。

> 除Worker外，还有一个Master（调度节点）的概念，调度节点是用来管理和分发**分布式计算任务**，一般来说，当获取较大数据源或计算结果较大时，调度节点配置越高，性能越好；它使用的资源包含在资源容器当中。

**操作演示：**

![](_media/xspark-deploy1.gif)


### 开始使用XSpark

#### Spark scala 示例

**操作流程：**

点击**开始使用**，在新弹出的页面上选择**XSpark Tutorial**进入代码编辑窗口。

在代码编辑窗口中，操作演示所使用的示例是读取**七牛对象存储的Bucket中的companies.json**文件，然后输出这个文件的所有字段信息。

意思就是，在代码编辑窗口中，我们需要指定一个文件路径，然后就可以对该文件中的数据进行计算，计算的代码由用户自行编写。

**操作演示：**

![](_media/xspark-start.gif)

#### 加载第三方依赖

**操作流程：**

点击代码编辑窗口之间的缝隙，可以增加一个新的代码编辑窗口。

我们再顶部打开一个新的代码编辑窗口，然后输入

```
%spark.dep
```

换行后，即可添加第三方依赖，代码示例如下：

```
z.load("第三方依赖地址")
```

?>我们也支持添加第三方镜像仓库来加速依赖下载，示例如下：
```
z.addRepo("3rdRepo").url("http://maven.aliyun.com/nexus/content/groups/public/")
```

!>注意：如果遇到如下提示，需要停止XSpark已经在运行的任务，才能使用加载第三方依赖功能，停止的方法见下图：
```
Must be used before SparkInterpreter (%spark) initialized
Hint: put this paragraph before any Spark code and restart Zeppelin/Interpreter
```

![](_media/stopjob.png)



**操作演示：**

![](_media/xspark-start1.gif)


#### Spark SQL 示例

**操作流程：**

我们可以在代码编辑窗口中编写Spark SQL来对数据进行分析。

首先我们需要将读取的文件注册成一张表，示例代码如下：

```
// 首先从对象存储中读取 companies.json文件
var dataPath = "qiniu://resourves/companies.json"

// 然后使用json的方式来读取文件
var table = sqlContext.read.json(dataPath)

// 将读取的文件注册为一张表
table.registerTempTable("companies")
```

将文件注册成表之后，我们即可编写 SQL 对数据进行分析：

```
%sql select name, sum(number_of_employees) number from companies group by name order by number desc limit 5
```

!> 注意： 编写SQL时， 头部必须包含`%sql`

**操作演示：**

![](_media/sparksql.png)

#### 定时任务

**操作流程：**

XSpark使用`cron表达式`来配置定时任务的执行频率。

点击页面上的时钟图标，来配置定时

快捷选择项中，我们支持1分钟、5分钟、1小时、3小时、6小时、12小时、1天；

也可以手动输入`cron表达式`自定义定时周期。

建议勾选：`auto-restart interpreter on cron execution`使得每次运行都是独立的运行环境。

**操作演示：**

![](_media/dingshi.png)


#### 基于XSpark的Python语言支持

![](_media/xspark-python.png)


#### 基于XSpark的R语言支持

![](_media/xspark-R.png)

#### 基于XSpark的机器学习

![](_media/xspark-mllib.png)


#### Spark UI
  对于有一定经验的Spark开发人员，可以访问`SparkUI`来看当前运行的任务状态。
  
![](_media/xspark-sparkui.png)
  


#### XSpark容器监控

 XSpark提供了容器级别的CPU，内存，磁盘监控。可以给Spark任务的调优和故障排查提供有力的支持。

![](_media/xspark-monitor.png)
  

#### XSpark邮件告警功能

邮件告警经常配合定时任务来使用，当任务失败时，会有邮件发送具体的失败信息到指定的邮箱内。

![](_media/xspark-email.png)


#### XSpark快捷重启Interpreter功能

重启Interpretr会初始化Spark解释器。重启Interpreter会结束掉当前正在运行的spark任务，释放资源。

![](_media/xspark-restart.png)

### Spark-jobserver 使用
为了能够在提供让用户自己提交代码，管理任务的目的。我们在XSpark里提供了Restful风格的Spark-jobserver。用户可以通过Spark-jobserver的api在非notebook的情况下使用XSpark集群。

首先我们需要获取到你自己的JobServer地址及使用方式。

![](_media/spark-job-server-get.gif)

**API使用方式：**

```http --auth-qiniu=~/.qiniu/your_ak_sk.conf GET $JOBSERVER_REST_URL```

!> 注意：

* `~/.qiniu/your_ak_sk.conf `是访问七牛服务的ak/sk，需要创建一个`your_ak_sk.conf`文件，然后把内容替换成你自己的ak/sk.

```
{
    "access_key": "***************************************",
    "secret_key": "***************************************",
    "auth": "qiniu/mac"
}

```

* `$JOBSERVER_REST_URL` 替换为你获取的地址


> 注：上面使用到的http命令是httpie的工具。[httpie](https://github.com/kirk-enterprise/httpie)是一个 HTTP 的命令行客户端。其目标是让 CLI 和 web 服务之间的交互尽可能的人性化。
安装方式：
```pip install --upgrade https://github.com/kirk-enterprise/httpie/tarball/master```

#### Rest API 介绍
大体上我们提供了下面几个用于操作Spark任务的api。

#### 1. Jars
##### 1.1 提交Jar
```
http --auth-qiniu=~/.qiniu/your_ak_sk.conf $JOBSERVER_REST_URL/jars/$JAR_NAME @job-server-tests/target/scala-2.10/job-server-tests_$VER.jar
```
其中：`$JAR_NAME` 替换成自行定义的jar名字即可

返回值：

```
200 OK
{
    "result": "Jar uploaded",
    "status": "SUCCESS"
}
```

##### 1.2 查询Jar
```
 http --auth-qiniu=~/.qiniu/your_ak_sk.conf $JOBSERVER_REST_URL/jars
```
返回值：

```
200 OK
{
    "test": "2017-06-07T15:18:58.581+08:00",
}
```

#### 2. Contexts
##### 2.1 创建Context
```
http --auth-qiniu=~/.qiniu/your_ak_sk.conf POST '$JOBSERVER_REST_URL/contexts/$CONTEXT_NAME?num-cpu-cores=4&memory-per-node=512m'
```
其中：

* `$CONTEXT_NAME` 替换成你自行定义的context name
* `num-cpu-cores` 配置SparkContext使用cpu核心数
* `memory-per-node` 配置每个节点使用的内存大小

返回值：

```
200 OK
{
    "result": "Context created",
    "status": "SUCCESS"
}
```

##### 2.2 查看Context
```
http --auth-qiniu=~/.qiniu/your_ak_sk.conf GET $JOBSERVER_REST_URL/contexts
```
返回值：

```
200 OK
[
    "test-context"
]
```

##### 2.3 删除Context
```
http --auth-qiniu=~/.qiniu/your_ak_sk.conf DELETE '$JOBSERVER_REST_URL/contexts/$CONTEXT_NAME'
```
其中：`$CONTEXT_NAME`为你要删除的Context名称

返回值：

```
200 OK
{
    "result": "",
    "status": "Success"
}
```

#### 3. Jobs
##### 3.1 提交Job
```
http --auth-qiniu=~/.qiniu/your_ak_sk.conf POST '$JOBSERVER_REST_URL/jobs?appName=$APP_NAME&classPath=$CLASSNAME&sync={true|false}&context=$CONTEXT_NAME' < input.json
```
其中：

* `$APP_NAME` 为前面提交的Jar的名字
* `$CLASSNAME` 为Jar里的Job类路径
* `$CONTEXT_NAME` 为前面提交的Context名称
* `sync={true|false}` 控制Job的提交是否是同步的
* `input.json` 为提交任务的参数信息，如：

```
{
	input.string:"a b c a b see"
}
```

返回值：

```
200 OK
同步：
{
    "jobId": "58da6978-3c22-43f0-89d6-31e14268f03c",
    "result": ...
}

异步：
{
    "classPath": "spark.jobserver.WordCountExample",
    "context": "test-context",
    "duration": "Job not done yet",
    "jobId": "58da6978-3c22-43f0-89d6-31e14268f03c",
    "startTime": "2017-06-07T15:25:06.171+08:00",
    "status": "STARTED"
}
```
##### 3.2 获取Jobs

```
http --auth-qiniu=~/.qiniu/your_ak_sk.conf $JOBSERVER_REST_URL/jobs
```
返回值：

```
200 OK
[
    {
        "classPath": "spark.jobserver.WordCountExample",
        "context": "test-context",
        "duration": "0.049 secs",
        "jobId": "58da6978-3c22-43f0-89d6-31e14268f03c",
        "startTime": "2017-06-07T15:25:06.171+08:00",
        "status": "FINISHED"
    },
    ...
]
```

##### 3.3 获取Job结果

```
http --auth-qiniu=~/.qiniu/your_ak_sk.conf $JOBSERVER_REST_URL/jobs/$JOB_ID
```
其中：`$JOB_ID`是任务Id，可以从提交的任务返回的信息，及获取所有Jobs时获取。

返回值：

```
200 OK
{
    "jobId": "58da6978-3c22-43f0-89d6-31e14268f03c",
    "result": ...
}
```
##### 例子：

(1). 同步提交，等待结果返回

```
http --auth-qiniu=~/.qiniu/your_ak_sk.conf POST '$JOBSERVER_REST_URL/jobs?appName=test&classPath=spark.jobserver.WordCountExample&sync=true&context=test-context' < input.json
```

(2). Ad-hoc方式提交，不使用已有context

```
http --auth-qiniu=~/.qiniu/your_ak_sk.conf POST '$JOBSERVER_REST_URL/jobs?appName=test&classPath=spark.jobserver.WordCountExample' < input.json
```
(3). 附带Jar包依赖

```
http --auth-qiniu=~/.qiniu/your_ak_sk.conf POST '$JOBSERVER_REST_URL/jobs?appName=test&classPath=spark.jobserver.WordCountExample&sync=true&context=test-context' < input.json
{
	dependent-jar-uris = ["file:///myjars/deps01.jar", "file:///myjars/deps02.jar"],
	input.string:" a b c a b see"
}

```


到此为止，基本上你应该清楚了怎么来使用Spark-jobServer api了，那么接下来我们来学习下如何编写Spark-Jobserver的应用

#### 编写Spark-JobServer App
这里我们直接使用Spark-JobServer的模板来创建一个Spark-JobServer 的app。

```
 sbt new spark-jobserver/spark-jobserver.g8
```
根据提示输入你的project信息

```
organization [com.example]: 
name [wordcount]:
version [0.1.0-SNAPSHOT]:
package [com.example.wordcount]:
scala_version [2.10.6]:
sbt_version [0.13.12]:
sjs_version [-SNAPSHOT]:

Template applied in ./wordcount
```
这里为了演示说明我们直接使用默认的wordcount来讲解：

```
object WordCountExample extends SparkJob {
  def main(args: Array[String]) {
    val conf = new SparkConf().setMaster("local[4]").setAppName("WordCountExample")
    val sc = new SparkContext(conf)
    val config = ConfigFactory.parseString("")
    val results = runJob(sc, config)
    println("Result is " + results)
  }

  override def validate(sc: SparkContext, config: Config): SparkJobValidation = {
    Try(config.getString("input.string"))
      .map(x => SparkJobValid)
      .getOrElse(SparkJobInvalid("No input.string config param"))
  }

  override def runJob(sc: SparkContext, config: Config): Any = {
    sc.parallelize(config.getString("input.string").split(" ").toSeq).countByValue
  }
}

```

从上面代码中我们可以看到，编写一个spark-jobserver 的app需要实现`SparkJob`这个trait及重写两个核心方法。

```
object SampleJob extends SparkJob {
    override def runJob(sc: SparkContext, jobConfig: Config): Any = ???
    override def validate(sc: SparkContext, config: Config): SparkJobValidation = ???
}
```
如果需要修改jar包依赖
`cd $project` 修改 `build.sbt`


**编译打包**

```
cd $project
sbt package
```

到此编译好的jar可以根据前面的提交jar的方式来测试你的第一个Spark-Jobserver app了


> 注：新版的Spark-jobserver api

```
object SampleJob extends NewSparkJob {
    def runJob(sc: SparkContext, runtime: JobEnvironment, data: JobData): JobOutput = ???
    def validate(sc: SparkContext, runtime: JobEnvironment, config: Config):
    JobData Or Every[ValidationProblem] = = ???
}
```

更多详情可以参考 [Spark-JobServer](https://github.com/spark-jobserver/spark-jobserver/tree/v0.7.0#new-sparkjob-api)
