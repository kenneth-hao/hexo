title: 架构演进 at ArchSummit2016
date: 2016-12-07 21:03:48
tags: architecture 

---

## 架构演进 at ArchSummit2016 

### 架构之痛

在 2014-2016 两年间，网站点击数量由 80,000,000 提升到 1,600,000,000。
平台主要分为 2B、2C 两大块。

C 端项目主要用 PHP 实现。
B 端项目主要用 Java 实现，运行在 商业 的 Websphere 容器中。
前端用 F5 做硬件负载。
B 端存储主要用 Oracle。 C 端存储主要用 MySQL
B、C 端服务互相依赖严重，强耦合，很难维护。

#### 现有问题
- 可用性低：99%
- 性能差：单机 QPS<50
- 强耦合
- 扩展性差、数据一致性差、容错能力差
- 可维护性差：上线难、迭代难

> QPS - Query Per Second 每秒查询率QPS是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准，在因特网上，作为域名系统服务器的机器的性能经常用每秒查询率来衡量。对应fetches/sec，即每秒的响应请求数，也即是最大吞吐能力。 

从以上概念来看吞吐量和响应时间是衡量系统性能的重要指标，QPS虽然和吞吐量的计量单位不同，但应该是成正比的，任何一个指标都可以含量服务器的并行处理能力。当然Throughput更关心数据量，QPS更关心处理笔数。 

QPS提升带来什么？QPS提升说明单台服务器处理能力提升，如果QPS提升1倍，服务器资源减少1半，或者说服务器不变可以支撑2倍的请求量。 
如何提升QPS？ 
1）减少CPU的使用时间（哪些代码会消耗CPU：循环、字符串拼接\查找\替换、编码\解码、序列化\反序列化、压缩） 
2）增加CPU的数量 
3）减少同步锁 
（如果CPU不能被压到85%以上，并且此时的QPS已经达到了峰值，则说明另有瓶颈，接下去关注内存） 
RT提升带来什么？ 
响应速度提升说明单词请求的处理速度提升，用户感觉任务处理速度更快，系统反应速度更快。当然在处理能力不变的情况下，RT的提升必然会提升QPS。 
如何提升RT？ 
1）减少I/O的响应时间 
2）减少I/O的调用次数 
3）减少CPU使用时间（当然在I/O占大头的应用里，这方面优化效果肯定不明显） 