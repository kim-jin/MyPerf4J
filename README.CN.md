简体中文 | [English](./README.EN.md)

<h1 align="center">MyPerf4J</h1>

<div align="center">

一个针对高并发、低延迟应用设计的高性能 Java 性能监控和统计工具。

[![GitHub (pre-)release](https://img.shields.io/github/release/LinShunKang/MyPerf4J/all.svg)](https://github.com/LinShunKang/MyPerf4J) [![Build Status](https://travis-ci.com/LinShunKang/MyPerf4J.svg?branch=develop)](https://travis-ci.com/LinShunKang/MyPerf4J) [![Coverage Status](https://coveralls.io/repos/github/LinShunKang/MyPerf4J/badge.svg?branch=develop)](https://coveralls.io/github/LinShunKang/MyPerf4J?branch=develop) [![GitHub issues](https://img.shields.io/github/issues/LinShunKang/MyPerf4J.svg)](https://github.com/LinShunKang/MyPerf4J/issues) [![GitHub closed issues](https://img.shields.io/github/issues-closed/LinShunKang/MyPerf4J.svg)](https://github.com/LinShunKang/MyPerf4J/issues?q=is%3Aissue+is%3Aclosed) [![GitHub](https://img.shields.io/github/license/LinShunKang/MyPerf4J.svg)](./LICENSE)

</div>

## 特性
* 高性能: 单线程支持每秒 **1000 万次** 响应时间的记录，每次记录只花费 **73 纳秒**
* 无侵入: 采用 **JavaAgent** 方式，对应用程序完全无侵入，无需修改应用代码
* 低内存: 采用内存复用的方式，整个生命周期只产生极少的临时对象，不影响应用程序的 GC
* 高精度: 采用纳秒来计算响应时间
* 高实时: 支持秒级监控，最低 **1 秒**

## 使用场景
* 在**开发环境**中快速定位 Java 应用程序的性能瓶颈
* 在**生产环境**中长期监控 Java 应用程序的性能指标

## 文档
* [English Doc](https://github.com/LinShunKang/MyPerf4J/wiki/English-Doc)
* [中文文档](https://github.com/LinShunKang/MyPerf4J/wiki/Chinese-Doc)    
    
## 监控指标
MyPerf4J 为每个应用收集数十个监控指标，所有的监控指标都是实时采集和展现的。

下面是 MyPerf4J 目前支持的监控指标列表:
- **[Method Metrics](https://grafana.com/dashboards/7766)**<br/>
RPS，Count，Avg，Min，Max，StdDev，TP50, TP90, TP95, TP99, TP999, TP9999, TP99999, TP100
![Markdown](https://raw.githubusercontent.com/LinShunKang/Objects/master/MyPerf4J-InfluxDB-Method_Show_Operation.gif)

- **[JVM Metrics](https://grafana.com/dashboards/8787)**<br/>
Thread，Memory，ByteBuff，GC，Class
![Markdown](https://raw.githubusercontent.com/LinShunKang/Objects/master/MyPerf4J_JVM_Compressed.jpeg)

    > 想知道如何实现上述效果？请先按照[快速启动](https://github.com/LinShunKang/MyPerf4J#%E5%BF%AB%E9%80%9F%E5%90%AF%E5%8A%A8)的描述启动应用，再按照[这里](https://github.com/LinShunKang/MyPerf4J/wiki/InfluxDB_)的描述进行安装配置即可。

## 快速启动
MyPerf4J 采用 JavaAgent 配置方式，**透明化**接入应用，对应用代码完全**没有侵入**。

### 打包
* git clone git@github.com:LinShunKang/MyPerf4J.git
* mvn clean package
* 把 MyPerf4J-ASM-${MyPerf4J-version}.jar 重命名为 MyPerf4J-ASM.jar

> 如果你使用的是 JDK 7 或者更高版本可以尝试直接下载 [MyPerf4J-ASM.jar](https://github.com/LinShunKang/Objects/blob/master/MyPerf4J-ASM-2.1.0.jar?raw=true)

### 配置
在 JVM 启动参数里加上以下两个参数
> -javaagent:/your/path/to/MyPerf4J-ASM.jar

> -DMyPerf4JPropFile=/your/path/to/myPerf4J.properties

其中，`MyPerf4JPropFile`的配置如下:

 ```
#应用名称
AppName=MyPerf4JTest

#MetricsProcessor类型，0:以标准格式化结构输出到stdout.log 1:以标准格式化结构输出到磁盘  2:以InfluxDB LineProtocol格式输出到磁盘
MetricsProcessorType=1

#配置各个Metrics日志的文件路径，可不配置
MethodMetricsFile=/data/logs/MyPerf4J/method_metrics.log
ClassMetricsFile=/data/logs/MyPerf4J/class_metrics.log
GCMetricsFile=/data/logs/MyPerf4J/gc_metrics.log
MemMetricsFile=/data/logs/MyPerf4J/memory_metrics.log
BufPoolMetricsFile=/data/logs/MyPerf4J/buf_pool_metrics
ThreadMetricsFile=/data/logs/MyPerf4J/thread_metrics.log

#配置Record模式，可配置为accurate/rough
RecorderMode=accurate
    
#配置时间片，单位为ms，最小1s，最大600s
MilliTimeSlice=10000
    
#需要监控的package，可配置多个，用英文';'分隔
IncludePackages=cn.perf4j.demo
 ```
        
> 想了解更多的配置？请看[这里](https://github.com/LinShunKang/MyPerf4J/wiki/%E9%85%8D%E7%BD%AE)

### 运行
* 输出结果，输出到 /data/logs/MyPerf4J/method_metrics.log:

    ```
    MyPerf4J Method Metrics [2018-09-06 19:21:40, 2018-09-06 19:21:45]
    Method[4]                           RPS  Avg(ms)  Min(ms)  Max(ms)   StdDev     Count     TP50     TP90     TP95     TP99    TP999   TP9999  TP99999    TP100
    DemoServiceImpl.getId1(long)       1974     0.00        0        0     0.00      9870        0        0        0        0        0        0        0        0
    DemoServiceImpl.getId2(long)       2995     0.50        0        2     0.01     14975        0        1        2        2        2        2        2        2
    DemoServiceImplV2.getId1(long)      787     0.00        0        0     0.00      3938        0        0        0        0        0        0        0        0
    DemoServiceImplV2.getId3(long)     1575     0.50        0        1     0.01      7876        1        1        1        1        1        1        1        1
    ```

### 卸载
在 JVM 启动参数中去掉以下两个参数，重启即可卸载此工具。
> -javaagent:/your/path/to/MyPerf4J-ASM.jar

> -DMyPerf4JPropFile=/your/path/to/myPerf4J.properties

## 问题
如果您有任何问题、疑问或者建议，请您毫不犹豫的 [提交Issue](https://github.com/LinShunKang/MyPerf4J/issues/new/choose) 或者 [发送邮件](mailto:feedback.myperf4j@gmail.com) : )

## 参考项目
MyPerf4J 是受以下项目启发而来：
* [Perf4J](https://github.com/perf4j/perf4j)
* [TProfiler](https://github.com/alibaba/TProfiler)

## 更多信息
想更深入的了解 MyPerf4J ？请看[https://github.com/LinShunKang/MyPerf4J/wiki/Chinese-Doc](https://github.com/LinShunKang/MyPerf4J/wiki/Chinese-Doc)。