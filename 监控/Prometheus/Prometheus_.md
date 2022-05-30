# Prometheus+Grafana+睿象云(监控告警系统)

## 背景

大数据主要解决的问题海量数据的

- 存储
- 传输
- 分析计算
  - 离线
  - 实时

​	不论是离线还是实时，都是在服务器上面来执行，执行的过程中对我们的服务器一些相关的指标来进行监控，为了保证公司的线上业务运行平稳运行，我们需要来关注各项业务的指标是否正常，比如说服务器、网络设备、硬件资源，数据库，包括应用程序本身。这个时候我们就需要借助第三方的监控工具Prometheus，监控的目标是当服务器的指标不符合我们的需求的时候，监控工具应该把这些数据收集起来，及时的做展现并报警通知相关的责任人，并且把异常信息记录下来。

- Prometheus:监控收集
- Grafana:展示信息
- 睿象云:报警



![image-20220530185306985](Prometheus_.assets/image-20220530185306985.png)

## 介绍

​	Prometheus 受启发于 Google 的 Brogmon 监控系统（相似的 Kubernetes 是从 Google 的 Brog 系统演变而来），从 2012 年开始由前 Google 工程师在 Soundcloud 以开源软件的 形式进行研发，并且于 2015 年早期对外发布早期版本。 

​	2016 年 5 月继 Kubernetes 之后成为第二个正式加入 CNCF 基金会的项目，同年 6 月 正式发布 1.0 版本。2017 年底发布了基于全新存储层的 2.0 版本，能更好地与容器平台、 云平台配合。 

​	Prometheus 作为新一代的云原生监控系统，目前已经有超过 650+位贡献者参与到 Prometheus 的研发工作上，并且超过 120+项的第三方集成。

## Prometheus 的特点

#### 1.易于管理

​	Prometheus 核心部分只有一个单独的`二进制文件`，不存在任何的第三方依赖(数据库， 缓存等等)。唯一需要的就是`本地磁盘`，因此不会有潜在级联故障的风险。

 	Prometheus 基于 `Pull 模型`的架构方式，可以在任何地方（本地电脑，开发环境，测试环境）搭建我们的监控系统。 

​	 对于一些复杂的情况，还可以使用 Prometheus `服务发现(Service Discovery)`的能力 动态管理监控目标。

#### 2.监控服务的内部运行状态

​	Pometheus 鼓励用户监控服务的内部状态，基于 Prometheus 丰富的 Client 库，用户可以轻松的在应用程序中添加对 Prometheus 的支持，从而让用户可以获取服务和应用 内部真正的运行状态。

![image-20220530190522854](Prometheus_.assets/image-20220530190522854.png)

#### 3.强大的数据模型

​	所有采集的监控数据均以指标(metric)的形式保存在`内置的时间序列数据库`当中 (TSDB)。所有的样本除了基本的指标名称以外，还包含一组用于描述该样本特征的标签。 如下所示：

```perl
http_request_status{code='200',content_path='/api/path',environment='produment'} =>
[value1@timestamp1,value2@timestamp2...]

#参数解释
http_request_status：指标名称(Metrics Name) 
{code='200',content_path='/api/path',environment='produment'}：表示维度的 标签，基于这些 Labels 我们可以方便地对监控数据进行聚合，过滤，裁剪。
[value1@timestamp1,value2@timestamp2...]：按照时间的先后顺序 存储的样本值。
```

​	每一条时间序列由指标名称(Metrics Name)以及一组标签(Labels)唯一标识。每条时 间序列按照时间的先后顺序存储一系列的样本值。 

#### 4 .强大的查询语言 PromQL

 Prometheus 内置了一个强大的数据查询语言 PromQL。 通过 PromQL 可以实现对 监控数据的查询、聚合。同时 PromQL 也被应用于数据可视化(如 Grafana)以及告警当中。 

通过 PromQL 可以轻松回答类似于以下问题： 

➢ 在过去一段时间中 95%应用延迟时间的分布范围？ 

➢ 预测在 4 小时后，磁盘空间占用大致会是什么情况？ 

➢ CPU 占用率前 5 位的服务有哪些？(过滤)

#### 5.高效

对于监控系统而言，大量的监控任务必然导致有大量的数据产生。而 Prometheus 可 以高效地处理这些数据，对于单一 Prometheus Server 实例而言它可以处理： 

➢ 数以百万的监控指标 

➢ 每秒处理数十万的数据点

#### 6 .可扩展 

​	可以在每个数据中心、每个团队运行独立的 Prometheus Sevrer。Prometheus 对于 联邦集群的支持，可以让多个 Prometheus 实例产生一个逻辑集群，当单实例 Prometheus  Server 处理的任务量过大时，通过使用功能分区(sharding)+联邦集群(federation)可以对 其进行扩展。

#### 7.易于集成

​	使用 Prometheus 可以快速搭建监控服务，并且可以非常方便地在应用程序中进行集 成。目前支持：Java，JMX，Python，Go，Ruby，.Net，Node.js 等等语言的客户端 SDK， 基于这些 SDK 可以快速让应用程序纳入到 Prometheus 的监控当中，或者开发自己的监控数据收集程序。

​	同时这些客户端收集的监控数据，不仅仅支持 Prometheus，还能支持 Graphite 这些 其他的监控工具。

​	同时 Prometheus 还支持与其他的监控系统进行集成：Graphite，Statsd，Collected， Scollector， muini， Nagios 等。 Prometheus 社区还提供了大量第三方实现的监控数 据采集支持：JMX，CloudWatch，EC2，MySQL，PostgresSQL，Haskell，Bash，SNMP， Consul，Haproxy，Mesos，Bind，CouchDB，Django，Memcached，RabbitMQ， Redis，RethinkDB，Rsyslog 等等。

#### 8.可视化

​	Prometheus Server 中自带的 Prometheus UI，可以方便地直接对数据进行查询，并 且支持直接以图形化的形式展示数据。同时 Prometheus 还提供了一个独立的基于  Ruby On Rails 的 Dashboard 解决方案 Promdash。

​	 最新的 Grafana 可视化工具也已经提供了完整的 Prometheus 支持，基于 Grafana 可 以创建更加精美的监控图标。

​	基于 Prometheus 提供的 API 还可以实现自己的监控可视化 UI。

#### 9.开放性

​	通常来说当我们需要监控一个应用程序时，一般需要该应用程序提供对相应监控系统协议的支持，因此应用程序会与所选择的监控系统进行绑定。为了减少这种绑定所带来的限制， 对于决策者而言要么你就直接在应用中集成该监控系统的支持，要么就在外部创建单独的服 务来适配不同的监控系统。

​	而对于 Prometheus 来说，使用 Prometheus 的 client library 的输出格式不止支持 Prometheus 的格式化数据，也可以输出支持其它监控系统的格式化数据，比如 Graphite。 因此你甚至可以在不使用 Prometheus 的情况下，采用 Prometheus 的 client library 来让 你的应用程序支持监控数据采集

## Prometheus的架构

![image-20220531003346931](Prometheus_.assets/image-20220531003346931.png)



**Prometheus 生态圈组件** 

➢ Prometheus Server：主服务器，负责收集和存储时间序列数据 

➢ client libraies：应用程序代码插桩，将监控指标嵌入到被监控应用程序中 

➢ Pushgateway：推送网关，为支持 short-lived 作业提供一个推送网关

➢ exporter：专门为一些应用开发的数据摄取组件—exporter，例如：HAProxy、StatsD、 Graphite 等等。 

➢ Alertmanager：专门用于处理 alert 的组件

**架构理解**

Prometheus 既然设计为一个维度存储模型，可以把它理解为一个 [联机分析处理（OLAP)](https://www.jianshu.com/p/83cd370536dd)系统。



1、存储计算层 

➢ Prometheus Server，里面包含了存储引擎和计算引擎。 

➢ Retrieval 组件为取数组件，它会主动从 Pushgateway 或者 Exporter 拉取指标数据。 

➢ Service discovery，可以动态发现要监控的目标。 

➢ TSDB，数据核心存储与查询。 

➢ HTTP server，对外提供 HTTP 服务。



2、采集层 

采集层分为两类，一类是生命周期较短的作业，还有一类是生命周期较长的作业。 

➢ 短作业：直接通过 API，在退出时间指标推送给 Pushgateway。(先把数据推送到网关，在从网关拉取数据) 

➢ 长作业：Retrieval 组件直接从 Job 或者 Exporter 拉取数据。



3、应用层

应用层主要分为两种，一种是 AlertManager，另一种是数据可视化。

➢ AlertManager 对接 Pagerduty，是一套付费的监控报警系统。可实现短信报警、5 分钟无人 ack 

​	打电话通知、仍然无人 ack，通知值班人员 Manager... 

​	Emial，发送邮件

​	....

​	`睿象云`



➢ 数据可视化 

​	Prometheus build-in WebUI 

​	`Grafana` 

​	其他基于 API 开发的客户端

## Prometheus安装

> 官网：https://prometheus.io/

> 下载地址：https://prometheus.io/download/

**安装 Prometheus Server**

​	Prometheus 基于 Golang 编写，编译后的软件包，不依赖于任何的第三方依赖。只需下载对应平台的二进制包，解压并且添加基本的配置即可正常启动 Prometheus Server。

```perl
1.上传安装包prometheus-2.29.1.linux-amd64.tar.gz至/usr/local/prometheus

2.解压
```

