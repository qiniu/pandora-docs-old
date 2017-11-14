#### 容器日志收集与处理 -- Kubernetes容器云平台日志分析案例

在 [nginx 日志分析搭建案例](https://qiniu.github.io/pandora-docs/#/demo/nginxlog) 中大家已经明白如何通过Pandora快速搭建实时监控与报警平台的整个过程。除了nginx这类常见基础组件之外，您可能本身有一些自研的程序，也需要做出相应的日志收集处理工作，那么操作方法与之类似，只是日志的解析方式不同而已。但是如果您的程序运行在容器云（如Kubernetes）之上，那么该如何处理呢？

本文将以 Kubernetes容器云平台运维 和 Kubernetes容器云用户 两个视角，为您介绍如何使用logkit进行日志收集。


#### Kubernetes容器云平台运维视角

作为Kubernetes容器云平台的运维，想要收集日志非常简单，因为Kubernetes已经将日志统一存放在  `/var/log/containers/` 文件夹下，如下图所示：

![此处输入图片的描述][1]

对于每一个日志，都是一个软连接，连接到实际的日志文件。对于此类日志，使用logkit的文件读取模式(tailx)即可直接读取，接入方式如下。

#### 数据接入

##### 根据您机器的操作系统版本下载logkit

https://github.com/qiniu/logkit/wiki/Download

解压后您可以看到

```
logkit
logkit.conf
public/
confs/
```

其中 `logkit.conf` 为主配置文件，用于配置logkit的web页面及端口配置，其中

* `bind_host` 是设置绑定的端口，启动后可以根据这个页面配置logkit。
* `static_root_path` 是logkit页面的静态资源路径，就是填写public文件夹所在路径，强烈建议写成 **绝对路径**

##### 运行logkit

```
nohup ./logkit -f logkit.conf > logkit.log 2>&1 
```

##### 访问logkit 配置页面

假设我们`bind_host`填写的页面内容为："localhost:3000"，那么我们可以在浏览器打开这个页面`http://localhost:3000`

点击【增加Runner】，第一步 【配置数据源】

![此处输入图片的描述][2]

注意选择数据源类型为 "tailx" 模式，填写日志路径为 `/var/log/containers/*.log`

然后配置解析方式，选择 【raw】方式解析
![此处输入图片的描述][3]

然后依次点击，知道配置发送方式时填写您的七牛账户"ak"，"sk"。

此时logkit部分您就配置完毕了，后续配置可以参考[【运维日志分析 -- 日志搜索和关键字报警】](https://qiniu.github.io/pandora-docs/#/demo/keywordalert)


#### Kubernetes容器云平台用户视角

当你作为Kubernetes用户时，你无法将logkit安装在服务器上收集日志，但是你可以利用 Kubernetes 的 [Daemonset](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) 功能, 将 logkit 以 daemonset 的形式运行在 Kubernetes 的每一个机器上，此时 logkit 容器与您的实际容器通过共享volume的方式收集日志数据。

Docker 的日志统一放置在宿主机的 `/var/lib/docker/containers` 目录上，作为daemonset的logkit会自动探测该目录中新生成的日志并将之收集。

##### 下载配置文件

您可以通过如下命令获取部署到Kubernetes的配置文件。

```
curl -L -O https://raw.githubusercontent.com/qiniu/logkit/develop/deploy/logkit_on_k8s.yaml
```

执行后生成的配置文件为 `logkit_on_k8s.yaml`，具体的配置如下（可能随版本变动下载到的会有所不同）：

```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: logkit-config
  namespace: kube-system
  labels:
    k8s-app: logkit
data:
  k8s.conf: |-
    {
      "name": "k8s_runner",
      "batch_interval": 60,
      "reader": {
        "mode": "tailx",
        "log_path": "/var/log/containers/*.log",
        "read_from": "oldest",
        "datasource_tag": "source_file",
        "expire": "240h",
        "stat_interval": "3m"
      },
      "parser": {
        "type": "raw",
        "timestamp": "true"
      },
      "senders": [
        {
          "sender_type": "pandora",
          "pandora_repo_name": "k8s_log",
          "pandora_ak": "${QINIU_ACCESS_KEY}",
          "pandora_sk": "${QINIU_SECRET_KEY}",
          "pandora_host": "https://pipeline.qiniu.com",
          "pandora_region": "nb",
          "pandora_schema_free": "true",
          "pandora_enable_logdb": "true",
          "pandora_logdb_host": "https://logdb.qiniu.com",
          "pandora_gzip": "true",
          "pandora_uuid": "false",
          "pandora_withip": "true",
          "ft_strategy": "backup_only",
          "ignore_invalid_field": "true",
          "pandora_auto_convert_date": "true"
        }
      ]
    }
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: logkit
  namespace: kube-system
  labels:
    k8s-app: logkit
spec:
  template:
    metadata:
      labels:
        k8s-app: logkit
    spec:
      terminationGracePeriodSeconds: 30
      containers:
      - name: logkit
        image: wonderflow/logkit:v1.3.6
        env:
        - name: QINIU_ACCESS_KEY
          value: change_me_to_your_qiniu_access_key
        - name: QINIU_SECRET_KEY
          value: change_me_to_your_qiniu_secret_key
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        securityContext:
          runAsUser: 0
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
        volumeMounts:
        - name: config
          mountPath: /app/confs/k8s.conf
          readOnly: true
          subPath: k8s.conf
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: varlogcontainers
          mountPath: /var/log/containers
          readOnly: true
        - name: varlogpods
          mountPath: /var/log/pods
          readOnly: true
      volumes:
      - name: config
        configMap:
          defaultMode: 0600
          name: logkit-config
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: varlogcontainers
        hostPath:
          path: /var/log/containers
      - name: varlogpods
        hostPath:
          path: /var/log/pods
      - name: data
        emptyDir: {}
```

##### 修改配置文件


默认情况下，我们的配置文件会使用 `kube-system` 这个 Kubernetes 的 namespace ，所有的部署仅针对该 namespace 生效。如果你想要使用别的 namespace ，只需要修改配置文件的 namespace 部分，将之改为你的 namespace 名称。

另外，这份默认的配置文件，你只需要修改2个基本参数，就可以运行。
 
``` 
 - name: QINIU_ACCESS_KEY
   value: change_me_to_your_qiniu_access_key
 - name: QINIU_SECRET_KEY
   value: change_me_to_your_qiniu_secret_key
```

将 `change_me_to_your_qiniu_access_key` 改为您七牛账号的 AK(access_key) ，将 `change_me_to_your_qiniu_secret_key` 改为您七牛账号的SK(secret_key)。

##### 部署到Kubernetes

部署到Kubernetes非常简单，只需要运行一行命令即可。

```
kubectl create -f logkit_on_k8s.yaml
```

通过以下命令查看部署是否成功：

```
$ kubectl --namespace=kube-system get ds/logkit
```

此时日志就源源不断的流向您的数据源啦，可以在日志检索中查看，后续配置可以参考[【运维日志分析 -- 日志搜索和关键字报警】](https://qiniu.github.io/pandora-docs/#/demo/keywordalert)



  [1]: http://ou3jgt6kj.bkt.clouddn.com/k9slogs.png
  [2]: http://ou3jgt6kj.bkt.clouddn.com/k8slog2.png
  [3]: http://ou3jgt6kj.bkt.clouddn.com/k8slog3.png