---

layout: post
title: 《Prometheus监控实战》笔记
category: 技术
tags: Go
keywords: Prometheus 实战

---

## 前言

* TOC
{:toc}

## 监控理念

devops基本理念：
1. if you can't measure it,you can't improve it
2. you build it,you run it, you monitor it.  谁开发，谁运维，谁监控，

四种主要的监控方式
1. Logging
2. Tracing
3. Metric
4. Healthchecks

监控是分层次的， 以metric 为例

1. 系统层，比如cpu、内存监控，面向运维人员
2. 应用层，应用出错、请求延迟等，业务开发、框架开发人员
3. 业务层，比如下了多少订单等，业务开发人员

metric 种类

1. counter（计数器），始终增加，比如http请求数、下单数
2. gauge（测量仪），当期值的一次快照测量，可增可减。比如磁盘使用率、当前同时在线用户数
3. Histogram（直方图），通过分桶方式统计样本分布
4. Summary（汇总），根据样本统计出百分位，比如客户端计算

4个黄金指标可以在服务级别帮助衡量终端用户体验、服务中断、业务影响等层面的问题。
1. 延迟：服务请求所需时间。
2. 通讯量：监控当前系统的流量，用于衡量服务的容量需求。例如，在HTTP REST API中, 流量通常是每秒HTTP请求数；
3. 错误：监控当前系统所有发生的错误请求，衡量当前系统错误发生的速率。
4. 饱和度：衡量当前服务的饱和度。比如，“磁盘是否可能在4个小时候就满了”。

**在Prometheus 提供的许多exporter 中都支持上述metric**，只有想不到，没有不支持的。

## 监控的几个反模式

1. 事后监控，没有把监控作为系统的核心功能
2. 机械式监控，比如只监控cpu、内存等，程序出事了没报警。只监控http status=200，这样数据出错了也没有报警。
3. 不够准确的监控
4. 静态阈值，静态阈值几乎总是错误的，如果主机的CPU使用率超过80%就发出警报。这种检查 通常是不灵活的布尔逻辑或者一段时间内的静态阈值，它们通常会匹配特定的结果或范围，这种模式 没有考虑到大多数复杂系统的动态性。为了更好地监控，我们需要查看数据窗口，而不是静态的时间点。
5. 不频繁的监控
6. 缺少自动化或自服务

一个良好的监控系统 应该能提供 全局视角，从最高层（业务）依次（到os）展开。同时它应该是：内置于应用程序设计、开发和部署的生命周期中。

很多团队都是按部就班的搭建监控系统：一个常见的例子是监控每台主机上的 CPU、内存和磁盘，但不监控可以指示主机上应用程序是否正常运行的关键服务。如果应用程序在你 没有注意到的情况下发生故障，那么即使进行了监控，你也需要重新考虑正在监控的内容是否合理。**根据服务价值设计自上而下（业务逻辑 ==> 应用程序 ==> 操作系统）**的监控系统是一个很好的方式，这会帮助明确应用程 序中更有价值的部分，并优先监控这些内容，再从技术堆栈中依次向下推进。从业务逻辑和业务输出开始，向下到应用程序逻辑，最后到基础设施。这并不意味着你不需要收集基础设施或操作系统指标——它们在诊断和容量规划中很有帮助——但你不太可能使用这些来报告应用程序的价值。如果无法从业务指标开始，则可试着从靠近用户侧的地方开始监控。因为他们才是最终的客 户，他们的体验是推动业务发展的动力。PS：只要业务没事，底层os一定没事， 底层os没事，业务逻辑不一定没事，监控要尽量能够反应用户的体验。

## node-exporter

node-exporter 控制启用哪些[收集器](https://github.com/prometheus/node_exporter#collectors)，许多收集器默认都是启用的。它们的状态要么是启用要 么是禁用，你可以通过使用no-前缀来修改状态。

1. textfile收集器，它允许我们暴露自定义指标。这些自定义指标可能是批处理或cron作业等无法抓取的，可能是没有exporter的源，甚至可能是为主机提供上下文的静态指标。收集器通过扫描指定目录中的文件，提取所有格式为Prometheus指标的字符串，然后暴露它们以 便抓取。比如

    ```sh
    # 创建一个目录来保存指标定义文件
    $ mkdir -p /var/lib/node_exporter/textfile_collector
    # 指标在以.prom 结尾的文件内定义，并且使用Prometheus 特定文本格式
    echo 'metadata{role="docker_server",datacenter="NJ"} 1' > /var/lib/node_exporter/textfile_collector/metadata.prom
    ```
2. systemd 收集器， 记录systemd中的服务和系统状态。支持将特定服务列入白名单（比如docker），只收集以下服务的指标

除了通过本地配置来控制 Node Exporter在本地运行哪些收集器之外，Prometheus还提供了一种方式来限制收集器从服务器端实 际抓取的数据

```yaml
scrap_configs:
  - job_name : "node"
    static_configs:
    - targets: ['192.168.1.1:9100']
    params: 
      collect[]:
        - cpu
        - meminfo
        - diskstats
        ...
```

cpu 相关metric  node_cpu_seconds_total ，数据从`/proc/stat`中抽取，以计数的形式告诉我们每个cpu在每种模式下使用了多少秒，包含标签

1. cpu
2. instance
3. job
4. mode  包括idle/iowait/irp/nice/system/user 等

内存相关 metric (对应标签instance/job)，以node_memory 为前缀，均以字节为单位
1. node_memory_MemTotal_bytes  主机上的总内存
2. node_memory_MemFree_bytes   主机上的可用内存
3. node_memory_Buffers_bytes   缓冲缓存中的内存
4. node_memory_Cached_bytes    页面缓存中的内存

磁盘相关以 node_filesystem 为前缀，包含mountpoint 等标签
1. node_filesystem_size_bytes，被监控的每个文件系统挂载的大小

systemd 收集器的数据，比如node_systemd_unit_state， 包括标签name 和state， 示例：`node_systemd_unit_state(name="docker.service",state="active") 0`

对于每个instance 的抓取，Prometheus 会填充一些监控指标
1. up，示例`up{job="<job-name>",instance="instance-id"} 1` 即数据抓取成功返回。
2. scrap_duration_seconds， 抓取的持续时间
3. scrap_samples_scraped，目标暴露的样本数

元数据风格的 指标，许多现有的exporter 使用“元数据”模式来提供额外的状态信息，比如cadvisor_version_info，包含标签 cadvisorRevision/dockerVersion/instance/job/kernelVersion/osVersion

不只是node-exporter，mysql-exporter 等也有很多的收集器可选

## 监控Kubernetes

![](/public/upload/go/prometheus_k8s.png)

### node-exporter

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
  ...
  spec:
    tolerations:
    - key: node-role.kubernetes.io/master
      effect: NoSchedule
    hostNetwork: true
    hostPID: true
    hostIPC: true
    securityContext:
      runAsUser: 0
```

利用Kubernetes DaemonSet控制器在集群中的每个节点上自动node-exporter pod。
1. 启用systemd收 集器，并指定要监控的特定服务的正则表达式，而不是主机上的所有服务。
2. 使用toleration来确保 node-exporter pod也会被调度到Kubernetes主节点，而不仅是工作节点。
3. 以用户0或root运行pod(这允许访问systemd)，并 且还启用了hostNetwork、hostPID和hostIPC，以指定实例的网络、进程和IPC命名空间在容器中可用。

```yaml
containers:
- images: prom/node-exporter:latest
  name: node-exporter
  volumeMounts:
    - mountPath: /run/systemd/private
      name: systemd-socket
      readOnly: true
  args:
    - "--collector.systemd"
    - "--collector.systemd.unit-whitelist=(docker|ssh|rsyslog|kubelet).service"
  ports:
    - containerPort: 9100
      hostPort: 9100
      name: scrape
```

配置一个Prometheus scrape job，结合Kubernetes daemonset, 只需要定义一次，未来所有Kubernetes服务端点都将被自动发现 和监控。

### 业务Pod监控

大部分业务监控 非常依赖边车模式。

**在Pod 或Service 中定义注解，可以让Prometheus 自动发现当前metric endpoint 并抓取数据**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: tornado-db
  annotations:
    prometheus.io/scrape: 'true'  # 告诉Prometheus抓取这个服务
    prometheus.io/port: '9104'    # 告诉 Prometheus要抓取的端口，将被放入__address__标签中
```

## 可靠性与可扩展性

Prometheus 本身自带了监控自己的许多指标，比如收集的指标数量（`sum(count by(__name__)({__name__=\~"\.\+"}))`），进而估算Prometheus 需要的内存和磁盘空间。可以用来辅助做Prometheus 的运维决策

### 可靠性

**Prometheus推荐的容错解决方案**是并 行运行两个配置相同的Prometheus服务器，并且这两个服务器同时处于活动状态。该配置生成的重复警报可以交由上游 Alertmanager 使用其分组 (及抑制)功能进行处 理 。一个推荐的方法是尽可能使上游Alertmanager高度容错 ，而不是关注Prometheus服务器的容错能力。

Alertmanager包含由HashiCorp Memberlist库提供的集群功能。Memberlist是一个Go语言库，使
用基于gossip的协议来管理集群成员和成员故障检测。**Alertmanager集群本身负责与集群的其他活动成员共享所有收到的警报**，并处理数据去重(如果需要的话)。

### 可扩展性

单个Prometheus监控方案 毕竟性能有限，Prometheus支持集群联邦。这种分区的方式增加了Prometheus自身的可扩展性，常见分区两种方式：

1. 每一个Prometheus Server实例只负责采集当前数据中心中的一部分任务（Job）。例如可以将不同的监控任务（比如机器和应用程序的）分配到不同的Prometheus实例当中
2. 其二是水平扩展，将同一任务的不同实例的监控数据采集任务划分到不同的Prometheus实例。

如果需要对某些区域或功能进行整体视图查看，那么可以使用federat ion功能将时间序列提取到集中的Prometheus服务器。

### 存储扩展

Prometheus includes a local on-disk time series database, but also optionally integrates with remote storage systems.

```yaml
# prometheus.yml
remote_write:
- url: "http://localhost:1234/receive"
remote_read
- url: xx
```

可以这么理解， Prometheus 定义了一套协议规范，可以和Adapter (及其背后的remote storage) 进行数据交互

可以在Prometheus配置文件中指定Remote Write（远程写）的URL地址，一旦设置了该配置项，Prometheus将采集到的样本数据通过HTTP的形式发送给适配器（Adapter）。而用户则可以在适配器中对接外部任意的服务。外部服务可以是真正的存储系统，公有云的存储服务，也可以是消息队列等任意形式。

同样地，Promthues的Remote Read（远程读）也通过了一个适配器实现。在远程读的流程当中，当用户发起查询请求后，Promthues将向remote_read中配置的URL发起查询请求（matchers，time ranges），Adapter根据请求条件从第三方存储服务中获取响应的数据。同时将数据转换为Promthues的原始样本数据返回给Prometheus Server。当获取到样本数据后，Promthues在本地使用PromQL对样本数据进行二次处理。

## 其它

一些比较有意思的exporter
1. mtail，专门用于从应用程序日志中提取要导出到时间序列数据库中的metric。从无法导出自己内部状态的应用程序中解析日志数据。
2. Blackbox exporter，探针监控，exporter通过 HTTP、HTTPS、DNS、TCP和ICMP来探测端点，执行检查并将生成的指标返回给Prometheus

Pushgateway位于发送指标的应用程序和Prometheus服务器之间。Pushgateway接收指标，然后作为目标 被抓取，以将指标提供给Prometheus服务器。