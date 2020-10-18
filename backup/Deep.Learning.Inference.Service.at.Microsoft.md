# [Deep Learning Inference Service at Microsoft](https://github.com/egolearner/paper-note/issues/4)

https://www.usenix.org/conference/opml19/presentation/soifer
微软在OPML19的论文。

## 引文
预测服务需求：因为DNN的计算复杂性，应用内嵌预测（近端部署）和现成的微服务不满足扩展性、性能和效率需求。
Deep Learning Inference Service (DLIS)支持了三百万qps，部署上万个模型，跨20个数据中心。

## 系统概述
总体划分为两类服务。
Model Master(MM)：满足模型需求和硬件资源限制条件下负责分配模型容器的单例编排器。
Model Server(MS)：数以千计的服务单元，同时做转发（将请求转发至指定模型的MS）和模型执行（低延时处理请求）。

## 智能模型部署(placement)
根据不同模型的需求部署到匹配的硬件。
**模型部署**
MM有所有服务器的硬件和可用资源信息，其中包括CPU指令集、核数、内存、GPU数等。MM通过validation测试知道模型的预估资源数。部署模型需要满足三个条件：满足模型的硬件需求、资源至少能分配一个实例出来、必须分布在指定个数的容灾域中。
**异构硬件管理**
DLIS使用MCM(machine configuration model)来配置和管理硬件，如定期执行去安装GPU驱动、重设GPU时钟速度、验证GPU健康度。

## 低延迟模型执行
DLIS支持系统级和模型级优化，本文侧重系统级优化。
**资源隔离和数据本地性**
本地化数据访问以利用不同的cache层，使用容器隔离资源保证模型不会相互干扰。此外，DLIS还强制设置处理器亲和性（保证模型关键数据在最近的处理器cache中）、NUMA亲和性（保证模型不会跨memory bank？）和内存限制（模型不会被swap到磁盘）。
**服务器到模型通信**
Linux模型使用UDP通信。
Windows模型使用共享内存队列通信。

## 高效路由
两大挑战：突发请求，长尾耗时。
解决思路为后备请求和跨服务器取消，先发给Server1，2ms后发给Server2，先处理请求的Server会通知另一Server取消处理。

## 结论
DLIS支持了上万个模型百万级qps的预测请求，使用不同的硬件低延迟支持了微软的生产级应用。

---

https://www.usenix.org/sites/default/files/conference/protected-files/opml19_slides_soifer.pdf
会议ppt