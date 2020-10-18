# [Autopilot: workload autoscaling at Google[5]](https://github.com/egolearner/paper-note/issues/11)

Google在EuroSys 2020的论文。  
https://dl.acm.org/doi/abs/10.1145/3342195.3387524

# 摘要
解决的问题：用户手动指定cpu和内存资源会导致资源浪费，因为人工运维小心起见倾向于指定更大的资源限制。  
解决方案：Google使用Autopilot来自动配置资源，具体分为水平扩展和垂直扩展。最终Autopilot显著降低了资源浪费，减少了严重受OOM影响的任务数。  
为了推广Autopilot，采取的策略包括潜在推荐对可选加入的用户容易可见、自动迁移特定类型的任务、支持自定义推荐算法。

## 1 Introduction
需要autoscaling的原因：用户很难合理评估资源，因为涉及cpu、内存、实例数的不同组合。即使做了压测，得到当时合理的资源数，但资源需求也会变得过期，因为有下列因素存在：一些任务有时间周期、流量的变化、软硬件的升级等。

## 2 用Borg管理资源
要处理的工作量称为job，每个job有若干个task。一个task在单个机器上运行，每个机器上同时运行多个task。job分为serving和batch两类。不同job的OOM忍受度不同。Borg支持原地修改job的cpu和内存，如果机器的资源不足，Borg将终止低优先级的任务。

## 3 Autopilot自动化设置上界
### 3.1 架构
![image](https://user-images.githubusercontent.com/45122959/85213321-73a35280-b38f-11ea-945e-cb7205b49bd9.png)
Autopilot每个job分别考虑，没有跨job学习。有3个实例，其master负责选择推荐算法并将推荐结果传给Borgmaster。Autopilot从独立的监控系统订阅每个task的资源使用情况。

### 3.2 垂直（按task）自动扩展
#### 3.2.1 预处理：聚合监控
原始的监控是每秒一个样本的时间序列，聚合为5分钟粒度。
* 对于CPU，分为400个桶，记录落入每个桶的个数。
* 对于内存，只记录最大值，因为内存不足会导致OOM。

将每个task的直方图相加得到job的直方图。（疑问：job的直方图应该没有用？毕竟每个task的资源需求不同，按照task来统计才是合理的。还是说这里是task的多个实例相加？但没看到取平均）
#### 3.2.2 滑动窗口推荐
策略：需求增加时上界快速增加，但缓慢减少，防止对短时的工作量下降抖动响应过快。为了平滑load毛刺，增加指数下降的权重：  
w[τ] = 2<sup>-τ/t<sub>1/2</sub></sup>  
其中τ为样本age，t<sub>1/2</sub>为半衰期。Autopilot是给长运行任务优化的，分别使用12小时和48小时作为CPU和内存的半衰期。
![image](https://user-images.githubusercontent.com/45122959/85213329-946ba800-b38f-11ea-80c1-f53fad0de2bc.png)
针对CPU
* batch jobs：使用S<sub>avg</sub>
* serving jobs: 使用S<sub>p95</sub>或者S<sub>p90</sub>，取决于job的latency敏感度。

针对内存，Autopilot根据job的OOM容忍度使用不同的统计值。OOM容忍度对大多数大任务设置为low，对小任务设置为minimal，并且可以被用户覆盖。
* low: S<sub>p98</sub>
* minimal: S<sub>max</sub>
* 对于中等容忍度，使用S<sub>p60</sub>或者0.5S<sub>max</sub>

最后，加上10-15%的安全边界（上界越大则边界越小）。并且使用上一小时的最大推荐值以防止抖动。
#### 3.2.3 ML推荐
定义cost function，对每个job选取适当的参数来优化cost function，Autopilot可以自动化的对每个job优化滑窗推荐中固定的参数，如半衰期、安全边界、downscaling稳定时间。

ML推荐内部由大量的模型构成。对每个job，recommender定期选择最优表现的模型，被选中的模型负责设置上限。每个模型都是arg min类型的算法，优化cost——模型的区别在于arg min的每个元素的权重不同。Autopilot需要向job owner解释recommender设置的上界，有许多简单模型的好处是单个模型大致对应于推断出的job特点（例如稳定时间长的模型对应于使用率剧烈变化的job）。因此给定选中模型的权重，很容易解释决策。
> More formally, for a signal 𝑠, at time 𝑡 the ML recom- mender chooses from an ensemble of models {𝑚} a single model 𝑚 [𝑡 ] that is used to recommend the limits. A model is a parameterized arg min algorithm that computes a limit given historical signal values. A model 𝑚 is parameterized by a decay rate 𝑑𝑚 and a safety margin 𝑀𝑚 .

##### 单个模型
在时刻t，对每个bucket的上界值L，计算最近使用直方图s[t]的underrun和overrun损失，然后和历史值做指数平滑。
> The overrun cost 𝑜 (𝐿) [𝑡 ] counts the number of samples in buckets over the limit 𝐿 in the most recent histogram:
![image](https://user-images.githubusercontent.com/45122959/85213335-aa796880-b38f-11ea-876c-e3d3166ed181.png)
> the underrun cost 𝑢(𝐿)[𝑡] counts the number of samples in buckets below the limit 𝐿,
![image](https://user-images.githubusercontent.com/45122959/85213339-b402d080-b38f-11ea-8c2d-958be1e69b29.png)
> Then, a model picks a limit 𝐿𝑚′ [𝑡 ] that minimizes a weighted sum of overruns, underruns and a penalty Δ(𝐿, 𝐿𝑚′ [𝑡 − 1]) for a possible change of the limit:
![image](https://user-images.githubusercontent.com/45122959/85213340-bd8c3880-b38f-11ea-9ba2-86808d3476d0.png)
包含三部分的关键损失
* overrun 表示失去机会的损失。serving中请求被延迟，终端用户可能放弃使用。
* underrun 表示infra的损失。任务预留的资源越多，需要越多的电力、机器和人。
* 惩罚项Δ帮助避免过于频繁的修改上界。

> Finally, the limit is increased by the safety margin 𝑀𝑚 , i.e.,
![image](https://user-images.githubusercontent.com/45122959/85213343-c715a080-b38f-11ea-8910-7a37d9cfcb6e.png)

##### ensemble
单个模型的cost
> the ML recommender maintains for each model its (exponentially smoothed) cost 𝑐𝑚 which is a weighted sum of overruns, underruns and penalties for limit changes:
![image](https://user-images.githubusercontent.com/45122959/85213351-d399f900-b38f-11ea-936b-a736fc7f380c.png)

> the recommender picks the model that minimizes this cost, but with additional penalties for switching the limit and the model:
![image](https://user-images.githubusercontent.com/45122959/85213354-dac10700-b38f-11ea-878d-c2bbb18c428a.png)

> The ensemble has five hyperparameters: the weights in the cost functions defined above (𝑑, 𝑤𝑜, 𝑤𝑢, 𝑤Δ𝐿 and 𝑤Δ𝑚). 

在离线实验中优化这些超参数，在保存的代表性的job的trace样本中模拟Autopilot的行为，目标是产生大幅领先其他算法的配置。以迭代和半自动的方式进行：穷举权重的可行值，然后人工分析异常点，如果异常不可接受，在下次迭代聚合结果时人工增加相应job的权重。

### 3.3 水平自动扩展
垂直扩展确定单个task的最佳资源，水平扩展增加或减少实例。水平扩展支持两种方式指定：
1. 根据cpu利用率。用户指定平均CPU使用率的窗口时长（默认5分钟）、水平长度T（默认72小时）、统计值S(max或P<sub>95</sub>)、目标平均使用率r<sup>*</sup>。Autopilot计算最近T时间内的使用率统计值，原始实例数等于统计值/r<sup>*</sup>。
![image](https://user-images.githubusercontent.com/45122959/85213363-f2988b00-b38f-11ea-9fa2-7428a635336c.png)

2. 目标大小。用户指定函数f(t)，函数使用监控系统中的数据，如文件服务器使用文件大小来做伸缩。

水平扩展需要更多的定制化。原始的实例数再做后处理得到稳定的推荐，目标是防止实例数的突然变化。
* deferred downscaling。使用最近T<sub>d</sub>时间段的最大值。40%使用2天，35%使用3天。
* slow decay 避免同时停止过多task，每5分钟做停止。
* defer small changes 忽略过小的修改
* limiting growth 允许用户指定在初始化的任务比例上界，因此控制task增加的速度。

## 5 赢得用户信任：逐步推广的关键点
### 5.1 评估过程
1. 离线模拟评估
2. dry run，推荐结果写入日志，但不生效。分析统计聚合数据和异常点。
3. A/B test
4. 分批部署上线。
### 5.2 用户易于访问Autopilot上界
在dashboard中展示资源使用情况，以及Autopilot计算的资源上界（即使没有开启），帮助用户理解Autopilot的行为和赢得信任。
### 5.3 自动迁移
赢得足够信任后，对存量的小任务和新任务默认开启。用户收到提前通知，可以选择退出。
### 5.4 自定义recommender
自定义recommender可以让用户沉淀带有业务特点的算法、或者在Autopilot之前开发的auto scailing算法。

## 6 减少工程苦工
Autopilot减少了需要人工调整的情况和OOM。

## Take away
* autoscailing算法
* 评估Autopilot的方式、推广方式