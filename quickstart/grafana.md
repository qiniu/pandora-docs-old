### Grafana 简介
[Grafana](https://oiw6da4op.qnssl.com/181919_0KHT_865233.jpg)
是开源的用来实时展示、分析和报警的软件，在七牛的应用中心提供Grafana应用，Pandora 时序数据库、日志检索服务都适配了Grafana，可以用Grafana实时展示、分析时序数据库和日志检索服务中的数据，并报警。

#### 特点

> 可视化

> 报警

> 动态面板展示

> 混合数据源展示

> 多种报警通知方式

更多特定可以通过[Grafana提供的demo](http://play.grafana.org/)发现更多用法。


### 创建应用

1. 首先登陆七牛portal的应用平台 找到Grafana
![grafana](https://oiw6da4op.qnssl.com/grafana/img1.png)

2. 点击立即部署按钮开始创建Grafana应用
![grafana](https://oiw6da4op.qnssl.com/grafana/img2.png)

输入您的app名和应用别名，选择部署区域(注意！目前TSDB的数据在`华东`，所以Grafana只可部署在华东区域)，点击确定创建

应用名称：账号内唯一应用名称，且只能满足以下条件：（1. 只能包含字母、数字和减号，首尾字符只能为字母或数字。 2. 字符长度不能超过 30）
应用别名：供显示使用的标题名。

3. 等待app启动后，输入密码（密码长度必须>=6），点击 确认配置

> 注意，因为Grafana App具有公网域名，所以建议设置一个高强度的密码（此密码在进入Grafana App后可以修改）。

![Grafana](https://oiw6da4op.qnssl.com/grafana/img3.png)

4. 访问Grafana进入Grafana页面

> 注意，该Grafana App是暴露在公网上的，可收藏地址用于后续访问。

![Grafana](https://oiw6da4op.qnssl.com/grafana/img5.png)

### TSDB数据源

在Grafana中使用Pandora TSDB之前，我们需要先添加数据源。

1. 登录grafana，点击菜单中的 Data Sources
![点击菜单中的 Data Sources](https://oiw6da4op.qnssl.com/grafana/QQ20170308-1@2x.png)

2. 点击 Add data source 按钮
![点击 Add data source 按钮](https://oiw6da4op.qnssl.com/grafana/QQ20170308-0@2x.png)

3. 在Name填入数据源的名字, Type 选择 Pandora TSDB
![在Name填入数据源的名字, Type 选择 Pandora TSDB](https://oiw6da4op.qnssl.com/grafana/QQ20170308-2@2x.png)

4. 填入相应参数，点击添加按钮即可
> 注意： url 必须填入 http://localhost:8999
![填入相应参数，点击添加按钮即可](https://oiw6da4op.qnssl.com/grafana/QQ20170308-4@2x.png)

其中，repo名可以在七牛portal的时序数据库页面中找到

![repo名可以在七牛portal的时序数据库页面中找到](https://oiw6da4op.qnssl.com/grafana/DD847726796988BEDEDEDB26809B2D1C.jpg)

5. 添加完数据源后，在菜单中选择 dashbords -> new 就可以新建自己的dashbords来展示数据了

![新建dashboard](https://oiw6da4op.qnssl.com/grafana/QQ20170308-5@2x.png)

6. 选择graph可以创建新的图表

![新建图表](https://oiw6da4op.qnssl.com/grafana/QQ20170308-6@2x.png)

7. 点击图标标题，然后点击弹出菜单的Edit即可编辑图标

![编辑图标](https://oiw6da4op.qnssl.com/grafana/QQ20170308-7@2x.png)

8. 编辑界面中的 datasource 选择刚才添加的datasource，即可实时预览和智能提示

![编辑界面中的 datasource 选择刚才添加的datasource，即可实时预览和智能提示](https://oiw6da4op.qnssl.com/grafana/QQ20170308-8@2x.png)

9. 编辑完成后点击 保存 按钮保存新添加的模板
![保存模板](https://oiw6da4op.qnssl.com/grafana/QQ20170308-9@2x.png)

### LogDB数据源

在grafana中使用pandora logdb之前，我们需要先添加数据源。

1、首先，登陆grafana，点击菜单中的 Data Sources

![img](https://oiw6da4op.qnssl.com/grafana/QQ20170308-1@2x.png)

2、点击 Add data source 按钮

![img](https://oiw6da4op.qnssl.com/grafana/QQ20170308-0@2x.png)

3、填入相应参数，点击添加按钮即可


![img](http://oji8s4dhx.bkt.clouddn.com/QQ20170321-0.png)

	注意： 
	1. url 必须填入 http://localhost:8999/logdb
	2. index 名字填写时候必须选择Daily，同时[repoName-]YYYY.MM.DD，用实际的数据仓库名字替换repoName。注意中括号内不能包含任何诸如空格、制表位等空白和特殊字符。
	3. Time field name 是指定数据仓库中的时间字段，默认值@timestamp，必须替换为实际的字段名字，否则将无法生效。
	4. ES 版本固定选择2.x 

其中，repoName 名可以在七牛portal的logdb页面中找到

![img](http://oji8s4dhx.bkt.clouddn.com/QQ20170321-1.png)

4、完数据源后，在菜单中选择 dashbords -> new 就可以新建自己的dashbords来展示数据了

![img](https://oiw6da4op.qnssl.com/grafana/QQ20170308-5@2x.png)

5、 选择graph可以创建新的图表

![img](https://oiw6da4op.qnssl.com/grafana/QQ20170308-6@2x.png)

6、点击图标标题，然后点击弹出菜单的Edit即可编辑图标

![img](https://oiw6da4op.qnssl.com/grafana/QQ20170308-7@2x.png)

7、编辑界面中的 datasource 选择刚才添加的datasource，即可实时预览和智能提示

![img](https://oiw6da4op.qnssl.com/grafana/QQ20170308-8@2x.png)

8、编辑完成后点击 保存 按钮保存新添加的模板

![img](https://oiw6da4op.qnssl.com/grafana/QQ20170308-9@2x.png)


### 折线图


#### LogDB 折线图配置

0. 点击创建，或者编辑图表的时候，会出现图表编辑器
1. Query 是Logdb 支持的查询语言，如`host:$host AND status:$status AND request_time:>=$request_time`，其中`$`符号引用的变量是下文中将要介绍到的模板变量
2. Metric 是我们对数据进行的聚合运算种类，LogDB 现在支持
	1. Count 计数聚合
	2. Sum, Avg, Min, Max 求和、平均、最大、最小
	3. Extended Stats 包含以上的常见聚合指标，同时计算了标准差等高级指标
	4. Percentiles 分位数，计算任意指定的分位数，常用来计算中位数、95值等
	5. Unique Count 去重计数，常用来求UV
	6. Moving Average 滑动平均值，需要首先定义另一个聚合指标，有Simple, Linear, Exponentially Weighted, Holt Linear, Holt Winters 五种模型支持
	7. Derivative 差分，需要首先定义另一个聚合指标，计算滑动窗口的差值
	8. Raw Document 原始文档
3. Group By 条件，聚合的分组函数
	1. Terms 根据某个字段的分词结果/字段值（如果没有分词的话）进行聚合，求top N
	2. Filters 可以设置多个查询条件，每个查询条件结果作为一个分组，比如可以将 `a:<500`, `a:>=500 AND a:<1000` 这两组条件作为Filter，则每个分组都只包含符合该分组查询条件的结果
	3. Date Histogram 按照时序方式进行聚合，需要指定一个时间字段，和时间间隔，比如对timestamp 字段进行时序聚合，每1分钟聚合一组数据（折线图必须指定一个Date Histogram的聚合条件）
	4. Histogram 类似于Date Histogram 分组，但是可以不限于时间字段，只要指定一个字段和步长，则对这个字段按照指定的步长进行分组。比如对响应时间，可以按照100ms 步长进行分组，分组结果构成是 0-100ms, 100-200ms, 200-300ms 等


![LogDB折线图](https://pandora-kibana.qiniu.com/grafana-demo/grafana%20%E5%9B%BE%E8%A1%A8-%E6%8A%98%E7%BA%BF%E5%9B%BE.png)

### 柱状图/散点图

1. 选择Display 选框
2. Bars 代表将图像切换成柱状图模式
3. Lines 是折线图
4. Points 是散点图

![柱状图](https://pandora-kibana.qiniu.com/grafana-demo/grafana%20%E5%9B%BE%E8%A1%A8-%E6%9F%B1%E7%8A%B6%E5%9B%BE.png)

### 堆叠图

1. 选择Display 选框
2. Stack 代表选择堆叠图模式，多条曲线结果将会堆叠在图像上
3. Percent 代表堆叠过程中，将数据转化为百分比而非绝对值

![堆叠图](https://pandora-kibana.qiniu.com/grafana-demo/grafana%20%E5%9B%BE%E8%A1%A8-%E5%A0%86%E5%8F%A0%E5%9B%BE.png)

### 饼状图

Step 1 

选择饼状图，进行创建。或者从已有的饼状图进行编辑。

![选择饼状图](https://pandora-kibana.qiniu.com/grafana-demo/grafana%20%E5%9B%BE%E8%A1%A8-%E9%A5%BC%E7%8A%B6%E5%9B%BE.png)

Step 2

按照跟其他的报表一样过程，配置图中1-5，查询条件、聚合函数、以及Group by 条件。因为饼状图表达的是分类关系，所以除了Date Histogram之外一定要有一个其他的Group by 条件。比如Terms，Filters 或者Histogram 条件。图中示意的是Terms 条件。

![饼状图配置](https://pandora-kibana.qiniu.com/grafana-demo/grafana%20%E5%9B%BE%E8%A1%A8-%E9%A5%BC%E7%8A%B6%E5%9B%BE-%E9%85%8D%E7%BD%AE.png)

Step 3

在Options 有更多进行图表调整的功能。

1. Value 可以选择min, max, avg, current, total 四个选项。可以确定饼图中具体数值是通过最近一段时间怎么计算得来，分别是最近一段时间的最小值、最大值、平均值、当前值、总计值。这个选项非常重要，图中数值依赖此配置进行计算。
2. 2-5 选项是图例的设置。Show Legend 选择是否展示图例，Position 是代表图例在饼图的哪个方向（下方或者右方等），以及是否展示图例的真实值和百分比。

![饼状图配置1](https://pandora-kibana.qiniu.com/grafana-demo/grafana%20%E5%9B%BE%E8%A1%A8-%E9%A5%BC%E7%8A%B6%E5%9B%BE-legend.png)


### 数据表格

Step 1 

选择数据表格，进行创建。或者从已有的数据表格进行编辑。

![选择数据表格](https://pandora-kibana.qiniu.com/grafana-demo/grafana%20%E5%9B%BE%E8%A1%A8-%E6%95%B0%E6%8D%AE%E8%A1%A8%E6%A0%BC-%E9%80%89%E6%8B%A9.png)

Step 2-1 查询时序数据计算结果

按照跟其他的报表一样过程，配置图中1-5，查询条件、聚合函数、以及Group by 条件。

![数据表格-计算结果](https://pandora-kibana.qiniu.com/grafana-demo/grafana%20%E5%9B%BE%E8%A1%A8-%E6%95%B0%E6%8D%AE%E8%A1%A8%E6%A0%BC-%E8%AE%A1%E7%AE%97%E7%BB%93%E6%9E%9C%E5%B1%95%E7%A4%BA.png)

Step 2-2 查询时序数据原始数据

如果想要查看原始数据搜索结果，请在这里选择Metric 为 Raw Document。

![数据表格-原始数据](https://pandora-kibana.qiniu.com/grafana-demo/grafana%20%E5%9B%BE%E8%A1%A8-%E6%95%B0%E6%8D%AE%E8%A1%A8%E6%A0%BC-%E5%8E%9F%E5%A7%8B%E6%95%B0%E6%8D%AE.png)

Step 3 挑选展示的列

在Options 中可以在Columns 设置中挑选你想要展示在界面上的数据表的列。

![数据表格-挑选列](https://pandora-kibana.qiniu.com/grafana-demo/grafana%20%E5%9B%BE%E8%A1%A8-%E6%95%B0%E6%8D%AE%E8%A1%A8%E6%A0%BC-%E6%8C%91%E9%80%89%E5%B1%95%E7%A4%BA%E5%88%97.png)

### 中国地图

Grafana官方的市场里只有一个[世界地图](https://grafana.com/plugins/grafana-worldmap-panel),无法展示具体省份

七牛容器应用市场提供了的Grafana内置了全国地图，名字叫[pili map panel](https://github.com/pre-dem/dem-grafana/tree/master/src/github.com/grafana/grafana/data/plugins/pili-map-panel),是七牛PiLi团队开发的一个开源的中国地图。

注意该中国地图的精度只到省份，无法显示市区县的数据，并且只支持Pandora TSDB这种数据源(Data Source)

下面是使用该中国地图的方法:

Step 1. 新建一个pili map panel的图表

![新建一个pili map panel的图表](http://oo6e9ks0k.bkt.clouddn.com/QQ20170619-0.png)

Step 2. 点击panel title，开始编辑图表

![点击panel title，开始编辑图表](http://oo6e9ks0k.bkt.clouddn.com/QQ20170619-1.png)

Step 3. 配置图表

![配置图表](http://oo6e9ks0k.bkt.clouddn.com/QQ20170619-2.png)

选择数据源(目前只支持Pandora TSDB类型的数据源)

选择series

选择字段和聚合函数

去掉group by time(此处必须去掉，否则图出不来)

增加对province的group by(此处要求数据字段中必须有一个tag key为province)

![增加对province的group by](http://oo6e9ks0k.bkt.clouddn.com/QQ20170619-3.png)


### 模板变量

Grafana 提供了非常强大的模板变量功能。如下图所示，用户可以在监控面板上配置多个模板变量，通过下拉列表或者输入框的形式输入用户关心的选项。在页面上可以做到不同查询条件轻松切换，让报表更加生动活泼。

![Grafana模板变量功能](https://pandora-kibana.qiniu.com/grafana-demo/grafana%E5%8F%98%E9%87%8F-%E7%A4%BA%E4%BE%8B.png)

![Grafana模板变量动图](https://pandora-kibana.qiniu.com/grafana-demo/grafana%E5%8F%98%E9%87%8F-%E7%A4%BA%E4%BE%8B.gif)


#### 时间间隔

设置步骤

![创建时间间隔变量](https://pandora-kibana.qiniu.com/grafana-demo/grafana%E5%8F%98%E9%87%8F-%E5%88%9B%E5%BB%BA-%E6%97%B6%E9%97%B4%E9%97%B4%E9%9A%94%E5%8F%98%E9%87%8F.gif)

1. 如下图所示可以在设置按钮中选择 templating 按钮
2. 选择New 按钮新建一个模板变量
3. 选择Interval 变量类型，我们可以用这种变量表达时间间隔，同时设置Name 和 Label，Name是变量名称，实际引用的时候用`$变量名称`进行引用；Label 本身无实际作用，主要是用来展示在界面，让用户更加容易理解的。
4. 我们可以看到在Values 中，已经有大量预置的时间间隔，我们可以在其中增加，诸如1m（1分钟）,1h（1小时）,1d（1天）等时间间隔变量
5. 在界面，我们可以见到已经生成了名为时间间隔的下拉框列表，列表中包括了我们设置的时间间隔预设值
6. 我们将时序查询的interval 设置为 $t (t 为我们设置的变量Name)。此时在下拉框里选择不同的时间间隔，图表将随之进行切换。

![创建时间间隔变量-引用变量](https://pandora-kibana.qiniu.com/grafana-demo/grafana%E5%8F%98%E9%87%8F-%E5%88%9B%E5%BB%BA-%E6%97%B6%E9%97%B4%E9%97%B4%E9%9A%94%E5%8F%98%E9%87%8F-%E6%95%88%E6%9E%9C.gif)



#### 基于查询结果的下拉列表


0. 前置步骤请参考**时间间隔**变量设置
1. 选择**Query** 类型
2. Data source 选择你查询的目标数据源
3. Query 是查询所有可能值的查询语句，ES/Logdb 的查询方式是`{"find": "terms", "field": "status"}`，其中`status`  是我们查询的目标字段，在这里可以替换成你需要的字段。TSDB/Influxdb 的查询方式是`SHOW TAG VALUES WITH KEY = "status" `，查询`status` 字段的所有出现值。这里更深入的语法请参考ES 和Influxdb 的官方文档。
4. Regex 可以选择对于返回的状态值进行正则表达式过滤
5. Sort 选择排序方式
6. Multi-value 控制下拉框是否可以支持多选，如果不选中则只能单选
7. Include all value 控制是否可以支持All选项，支持全选所有的值，只在多选的模式下有效果
8. Preview of values 可以预览这个字段的所有值 

![创建查询结果变量](https://pandora-kibana.qiniu.com/grafana-demo/grafana%E5%8F%98%E9%87%8F-%E5%88%9B%E5%BB%BA-%E6%9F%A5%E8%AF%A2%E5%8F%98%E9%87%8F.png)

在设置完成后，我们将会获得一个下拉列表，而下拉列表中所有的值都是在数据源中出现的真实数据列表。

#### 数据源

数据源变量是为了达到在多个数据源进行数据切换功能的变量

0. 前置步骤请参考**时间间隔**变量设置
1. 选择**Datasource** 类型
2. Type 选择数据源类型，此处以ES 为例子
3. Instance Name filter 对于数据源的正则表达式过滤符
4. Preview of values 所有符合条件数据源的预览


![创建数据源变量](https://pandora-kibana.qiniu.com/grafana-demo/grafana%E5%8F%98%E9%87%8F-%E5%88%9B%E5%BB%BA-%E6%95%B0%E6%8D%AE%E6%BA%90%E5%8F%98%E9%87%8F.png)

在设置完成后，我们将会获得一个下拉列表，而下拉列表中所有的值都是在数据源中出现的真实数据列表。

![数据源变量效果](https://pandora-kibana.qiniu.com/grafana-demo/grafana%E5%8F%98%E9%87%8F-%E5%88%9B%E5%BB%BA-%E6%95%B0%E6%8D%AE%E6%BA%90%E5%8F%98%E9%87%8F-%E6%95%88%E6%9E%9C.png)


#### 自定义变量

自定义变量一般是由用户自定义设置取值范围的变量类型

0. 前置步骤请参考**时间间隔**变量设置
1. 选择**Custom** 类型
2. 此处是用户设置的所有值，可以在选框中进行选择，用逗号分隔所有的合法值
3. Include all value 控制是否可以支持All选项，支持全选所有的值，只在多选的模式下有效果
4. Multi-value 控制下拉框是否可以支持多选，如果不选中则只能单选
5. Include all value 控制是否可以支持All选项，支持全选所有的值，只在多选的模式下有效果
6. Preview of values 可以预览这个字段的所有值 

![自定义变量](https://pandora-kibana.qiniu.com/grafana-demo/grafana%E5%8F%98%E9%87%8F-%E5%88%9B%E5%BB%BA-%E8%87%AA%E5%AE%9A%E4%B9%89%E5%8F%98%E9%87%8F.png)

#### 交互式查询

当用户想要进行数据探索的时候，交互式查询变量是非常有效的一个工具，可以在这里自由添加查询条件，进行数据的筛选。交互式查询配置相对简单，只要指定一个目标的查询数据源。

![交互式查询效果](https://pandora-kibana.qiniu.com/grafana-demo/grafana%E5%8F%98%E9%87%8F-%E5%88%9B%E5%BB%BA-%E4%BA%A4%E4%BA%92%E5%BC%8F%E6%9F%A5%E8%AF%A2.gif)


#### 常量

![常量的配置方式](https://pandora-kibana.qiniu.com/grafana-demo/grafana%E5%8F%98%E9%87%8F-%E5%88%9B%E5%BB%BA-%E5%B8%B8%E9%87%8F.png)

![常量的效果](https://pandora-kibana.qiniu.com/grafana-demo/grafana%E5%8F%98%E9%87%8F-%E5%88%9B%E5%BB%BA-%E5%B8%B8%E9%87%8F-%E6%95%88%E6%9E%9C.png)

### 报警

#### 定义通知方式

![定义通知方式](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%885.58.37.png)

Alert list用来显示最近出现的报警；

![Alert list用来显示最近出现的报警](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%885.58.48.png)

Notification用来定义通知方式，比方说`邮件`,`slack`,`钉钉`,`WebHook`等；

![Notification用来定义通知方式](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%885.59.23.png)

#### 设定阈值

打开报警设置页面

![打开报警设置页面](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%886.04.54.png)

选择起止时间，序列，阈值

![选择起止时间，序列，阈值](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%886.05.39.png)

测试设定的阈值

![测试设定的阈值](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%886.05.53.png)

查看报警的历史记录

![查看报警的历史记录](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%886.06.30.png)

#### 删除报警规则

![删除报警规则](http://oo6e9ks0k.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-15%20%E4%B8%8B%E5%8D%886.06.41.png)

### 常见问题

##### 点击访问后，看不到Grafana

弹出窗口可能会被浏览器拦截，点击允许弹出即可

##### 看不到图
看不到图的问题多种多样，建议按照下面顺序排查

1. 在图中所选的时间段内，是否有数据打进TSDB
2. 数据量是否很少导致点无法连成线

![数据量是否很少导致点无法连成线](http://oo6e9ks0k.bkt.clouddn.com/1327FAE695E586CE266566B445C0391B.jpg)
尝试将上图中的选项都选上，以排除是由于点太少造成的图无法显示。

3. 图表类型太少，没有需要的图表类型

七牛应用市场的图表类型目前有，折线图，柱状图，堆叠图，饼状图，如果有需要其他模板的，请联系我们。


##### 报警不支持模板变量

因为模板变量是前端动态展示的，每个不同用户每次进入系统这个值都有可能不同。但报警的时候需要固定的配置，所以暂时还不支持在报警中使用模板变量。

##### Grafana 数据源配置问题

要注意以下几个常见的问题：

1. 选择数据源的时候必须选择Daily的模式，同时[repoName-]YYYY.MM.DD 除了repoName 需要替换成真实的数据仓库名字之外，**其它部分必须保持一致，不能包含额外空格**
2. 时间字段默认为@timestamp 必须要修改为真实的时间字段，**要注意字段不能带@符号**
3. 数据源默认时间间隔，**必须要有时间单位**，比如10s,1m等，不能只输入数字，否则查询会保护问题

##### Grafana 数据源和导入Json 不匹配

如果你是从其它的地方获得Grafana 的模板，导入Json文件创建dashboard的话，经常会出现所有的监控图都无法展示。此时请不要惊慌，可能是因为在创建数据源的时候名字和json中选择的数据源名字并不匹配。或者忘记创建该数据源了。

![错误配置数据源名称](https://pandora-kibana.qiniu.com/grafana-demo/grafana%20%E5%9B%BE%E8%A1%A8-%E6%95%B0%E6%8D%AE%E6%BA%90%E9%85%8D%E7%BD%AE%E9%94%99%E8%AF%AF.png)

此时仔细观察左上角的报错信息，将出错信息中提到的数据源名称设置到数据源配置中就可以了。