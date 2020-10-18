# [Accelerating Large Scale Deep Learning Inference through DeepCPU at Microsoft](https://github.com/egolearner/paper-note/issues/5)

https://www.usenix.org/conference/opml19/presentation/zhang-minjia

## 介绍
大规模DL预测的三大挑战
* 低延迟预测
* 超出单机容量时DL服务能有效水平扩展
* 受部署基础设施的影响，更倾向于用已有的商用硬件，即CPU

为此论文提出了SLT(Scenario, library, technique)的方法论。

## SLT
**Scenario** 
介绍主要的DL场景，如Deep feature, Web Q&A, Similarity ranking, Query rewriting, Query tagging

**Library**
对上面的场景分析，涉及的DL组件可以分为三类
* RNN族，如GRU/LSTM等
* 基础公共DL层，如矩阵乘法内核、池化层等
* 机器阅读理解和对话的DL层，如attention层

因此以上面的组件作为基础单元实现了DeepCPU，并包含定制优化。论文发现这些组件高度可复用，允许快速实现和低成本的支持新场景。

**Techniques**
* OP内优化 
    - Intel MKL结合定制的感知缓存的计算内核来实现更高效的矩阵运算，以及小的或者高瘦矩阵乘法（#2 也讲到高瘦矩阵是下一步DL预测优化的方向之一）。
    - 对常用激活函数的优化，如连续分数展开，并行，SIMD向量化
* OP间优化
OP融合
* 并行，调度与亲和性
Tensorflow针对通用的DAG，达不到最优的并行决策。论文针对计算量的特点通过分析模型结构进行全局优化，还将应用线程绑定到CPU物理核使DL计算NUMA-aware和socket-aware以避免上下文切换和跨socket通信的开销。

## DeepCPU使用
**定制优化**
使用DeepCPU库重新实现模型运行时，然后对线程设置等进行调优以达到最优性能。
优缺点：需要和模型开发者沟通，模型剧烈变化时有开发成本。

**框架集成**
把TF的运行时中常用和耗时的OP替换为DeepCPU的高性能实现。此外还和ONNX团队合作使用DeepCPU技术来赋能ONNX运行时。
优缺点：以框架用户为目标，只需要少量工作就能利用DeepCPU。

对于新的场景，优先采用框架集成的方式。

## 评估
使用DeepCPU实现5-20倍的耗时提升，同时最高实现100倍的呑吐提升。