#### 运维日志分析 -- apache 日志分析搭建案例

在 [nginx 日志分析搭建案例](https://qiniu.github.io/pandora-docs/#/demo/nginxlog) 中大家已经明白如何通过 Pandora 快速搭建实时监控与报警平台的整个过程。除了 nginx 之外，apache(httpd) 服务也是现代 web 服务栈中重要的组件之一，本文档作为一个补充，介绍通过 logkit 来收集 apache 的 access.log，同时介绍通过页面方式配置 logkit，数据进入到 Pandora 以后的搜索和分析部分与 nginx 日志分析的案例相同，本文档就不再赘述。

#### 快速开始

#### 数据接入

##### 根据您机器的操作系统版本下载 logkit

https://github.com/qiniu/logkit/wiki/Download

解压后您可以看到

```
logkit
logkit.conf
public/
confs/
```

其中 `logkit.conf` 为主配置文件，用于配置 logkit 的 web 页面及端口配置，其中

* `bind_host` 是设置绑定的端口，启动后可以根据这个页面配置logkit。
* `static_root_path` 是 logkit 页面的静态资源路径，就是填写 public 文件夹所在路径，强烈建议写成**绝对路径**

##### 运行 logkit

```
nohup ./logkit -f logkit.conf > logkit.log 2>&1 
```

##### 访问 logkit 配置页面

假设我们`bind_host`填写的页面内容为："localhost:3000"，那么我们可以在浏览器打开这个页面`http://localhost:3000`

默认进入的是 Runner 列表页面，在这里可以看到已经在运行的 Runner 有哪些，以及它们的配置情况。

![Runner列表](http://ou3jgt6kj.bkt.clouddn.com/logkitapa1.png)

其次点击第一步 "配置数据来源" 便开始我们的 Apache 日志配置过程，一般情况下，您的日志是跟 nginx 日志一样写入同一个文件的话，就选择 file 模式，file 模式会不断读取文件尾追加的数据。

![配置数据来源](http://ou3jgt6kj.bkt.clouddn.com/logkitapa2.png)

配置好数据来源以后，就开始配置数据解析方式，我们使用 grok 表达式解析 Apache 日志，我们提供了一些默认表达式的支持，其中 `%{COMMON_LOG_FORMAT}` 就可以解析 Apache 日志。

对于 grok 表达式，玩过 ELK 的用户不会陌生，我们完全支持 logstash 的 grok 表达式，对于自己配置的一些特殊 Apache 日志，可以访问我们的 wiki 页面了解进阶用法: [学习如何调试您的 grok 表达式](https://github.com/qiniu/logkit/wiki/Grok-Parser#%E5%A6%82%E4%BD%95%E8%B0%83%E8%AF%95%E6%82%A8%E7%9A%84grok-patterngrokdebug%E7%94%A8%E6%B3%95).

配置好后，您还可以在该页面尝试解析您的样例数据噢！

![grok 解析](http://ou3jgt6kj.bkt.clouddn.com/logkitapa3.png)

在配置完解析方式以后，就进入到发送端的配置中，最简单的，您只需要填写工作流名称以及七牛的 ak/sk 即可。

![sender 配置](http://ou3jgt6kj.bkt.clouddn.com/logkitapa4.png)

最后，点击生成配置文件，然后添加，就大功告成啦！

![添加配置](http://ou3jgt6kj.bkt.clouddn.com/logkitapa5.png)