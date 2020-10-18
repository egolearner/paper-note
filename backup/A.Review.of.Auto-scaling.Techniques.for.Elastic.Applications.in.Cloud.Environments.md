# [A Review of Auto-scaling Techniques for Elastic Applications in Cloud Environments](https://github.com/egolearner/paper-note/issues/10)

https://www.researchgate.net/publication/265611546_A_Review_of_Auto-scaling_Techniques_for_Elastic_Applications_in_Cloud_Environments

对弹性伸缩的综述
## Introduction
scaling类型
* 水平绅缩，又称scaling out/in，加减VM
* 竖直绅缩，又称scaling up/down，加减cpu、内存

弹性绅缩必须同时满足应用SLA（如rt)和资源SLA

## Auto-scaling Process: the MAPE loop
Auto-scaler面临的问题：
* 分配不足
* 过量分配
* 振荡：scaling过快，在上述两个状态来回切换。

MAPE循环
### Monitor
Auto-scaling的决策依赖于性能指标，取决于可用指标、采样粒度、指标获取成本。
>Hardware: CPU utilization per VM, disk access, network interface access, memory usage.
>GeneralOSProcess: CPU-timeperprocess,pagefaults,real memory (resident set).
>Load balancer: size of request queue length, session rate, number of current sessions, transmitted bytes, number of denied requests, number of errors.
>Application server: total thread count, active thread count, used memory, session count, processed requests, pend- ing requests, dropped requests, response time.
>Database: number of active threads, number of transactions in a particular state (write, commit, roll-back).
>Message queue: average number of jobs in the queue, job’s queuing time.

### Analyze
从监控系统的性能指标获取系统可用率的数据，是否预测未来的需求作为可选项，据此分为reactive和proactive两类。因为扩容需要一段时间才能生效，预测未来很重要，反应式系统在流量突增时无法应对，主动式系统有可能处理波动的需求并做提前扩容。

### Plan
根据分析的结果、目标SLA和其他云环境相关的因素（如价格，VM启动时间）做出决策。

### Execute
执行。

## A Classification of Auto-scaling Techniques
根据底层理论或技术分为5类
* 基于阈值的规则。容易部署，容易使用，然而有效性打问号。
* 强化学习。RL的问题是学习阶段长，收敛到最佳策略的时间可能长到不可行。
* 排队论。将每个VM视为请求队列，来评估不同的性能指标。主要限制是过于僵化，应用或者负载有变化时需要重新计算。
* 控制论。许多研究者认为最有潜力的技术，特别是加上资源预测后。
* 时间序列分析。预测精度取决于正确的技术和参数，能够主动auto-scaling的基础。

## Review of Auto-scaling Techniques
### 基于阈值的规则
```
if x1 > thrU1 and/or x2 > thrU2 and/or ...
    for durU seconds then
        n = n + s and
        do nothing for inU seconds
if x1 < thrL1 and/or x2 < thrL2 and/or ... 
    for durL seconds then
        n = n − s and
        do nothing for inL seconds
```
需要应用管理者设置合理的阈值。为了防止振荡，一般有冷静期。

RightScale’s auto-scaling algorithm在一般的反应式规则上增加了投票，每个VM投票是否扩缩容，RightScale建议操作后等15分钟的冷静期，因为一般新机器需要5-10分钟的运维时间。\
针对有些云服务商按小时收费，smart kill在计费小时结束前不会停止VM，既降低了成本，也因为防止持续创建和停止VM提高了性能。\
基于规则的两个主要问题是反应式的特性和设置正确的性能指标和相应阈值的困难性。阈值的有效性高度依赖负载变化，可能需要不断调优。为此有人提出了动态阈值的思路。

### 强化学习
![image](https://user-images.githubusercontent.com/45122959/74543293-18e09100-4f80-11ea-86f8-3f49645932bd.png)



作者的观点是RL有希望来解决通用的auto-scaling任务，但还没有足够的成熟。值得研究的方向包括对流量突增提供足够的适应性和应对连续状态空间与行为。

### 排队论
排队论可能不是设计通用auto-scaling系统的最优选择。排队论一般针对固定的架构，任何变化都需要重新计算，因此对弹性应用并不适用。另外，排队论是分析工具，需要加上其他组件才能构成完成的auto-scaler。

### 控制论

![image](https://user-images.githubusercontent.com/45122959/74543240-f8b0d200-4f7f-11ea-8dfa-0ccf3979c3bf.png)

反馈控制器的分类
* 固定收益控制器，如PID
* 自适应控制器，能够在线调整参数值，适用于缓慢变更的负载变化，不适用于陡增。
* 模型预测控制器(MPC)。主动式方式。

控制器是否适用于auto-scaling任务高度取决于控制器的类型和目标系统的动态性。易预测需求的系统可以用简单的反应式控制器，而adaptive和MPC更适合产生通用auto-scaling解决方案。

### 时间序列分析
时间序列分析能够预测弹性应用的未来需求，预先分配资源，主要的缺陷是预测精度。预测精度取决于目标应用的负载类型、请求突增度、选取的指标、时间窗、预测周期以及特定的技术。未来研究应该投入对特定的应用自动选取最优的预测技术。[automl? auto auto-scaling?]

## Conclusions and Future Work
反应式auto-scaling系统无法应对流量突增。因此应该将研究精力投向主动式的系统，预测未来的需求并分配足够的资源。另外，要研究如何降低分配VM的时间。\
作者认为应该利用时间序列分析技术的预测特性，加上控制器的自动化特性。之前的研究用了不同的方式来产生请求、有些用仿真有些用真实应用、不同的SLA定义、不同的执行平台等，作者认为需要构建通用的预测平台来产生良好定义的标准指标，用于对比和实现新的auto-scaling技术。