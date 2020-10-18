# [Data Management Challenges in Production Machine Learning[3]](https://github.com/egolearner/paper-note/issues/13)

https://thodrek.github.io/CS839_spring18/papers/p1723-polyzotis.pdf
SIGMOD 17的tutorial（什么是tutorial?）

## 1. 前言
目标是描述机器学习流水线上的数据管理问题，和数据库界的已有工作建立连接，概述待解决的问题。受众包括数据库研究者和从业人员，目的是让其明白生产机器学习流水线和数据管理交界处的问题，激发进一步的研究。

## 3. DATAISSUES IN PRODUCTIONMACHINE LEARNING
### 3.1 Understanding
基于数据特点自动推荐和生成从原始数据到特征的变换是有趣又没被探索的研究领域。\
已有的数据血缘管理技术能够跟踪机器学习的依赖，帮助我们理解复杂流水线中数据是怎么流动的。与此同时，机器学习有些新问题，确定去除遗留特征后的短期和长期影响、异构的基础设施下如何跟踪血缘。

### 3.2 Validation
校验包括
* 训练数据包括预期的特征
* 特征有预期的值
* 特征和预期的一样相关
* 预测数据不偏离于训练数据分布

预期特征及其取值特点可以用类似schema的东西来描述。但机器学习有下列问题：schema和数据可能会独立演化；训练数据的特征分布不能偏移太多（bounds of the drift)；当且仅当其他特征以某种方式做了归一化后才能对某些输入特征使用embedding；需要允许训练数据突然发生变化（如大选前搜索政治突增）。描述训练数据的schema既要有足够的表达力来允许或者不允许突然的变化，又能够发现一段时间内的数据分布偏移。

数据管理界的时间序列分析能够发现training-serving的数据分布的偏移，但time-travelling（使用serving后才有的特征做训练）问题需要别的解决方案。

带来更多复杂性的两个正交因素
* 数据的规模
* 校验可能要在数据分片上做，而非聚合后做，以发现特定数据源的问题（应该说的是有多个分片时其中一个分片数据有问题的场景）。

### 3.3 Cleaning
清理校验有错误的数据，三个子任务
* 理解错误发生的地方。需要比较明确的信息，如单纯说“特征X的分布变了”不如“Y取值[20,40)训练样本中的X有不同的分布”有意义。
* 理解错误的影响。影响小的可能就忽略了，问题在于如何确定影响？可以利用某些机器学习算法的特点，或者做ab实验（成本比较高，有可能利用DB参数优化的技术来有效的调度实验）。
* 修复错误。修复root cause，临时给数据打补丁（和数据库修复相关）

### 3.4 Enrichment
两种方式：增加新的特征；增加新的特征变换。一个关键问题是确定哪些新特征或变换能够有意义的丰富特征。另一个问题是是否可以用保护隐私的方式来评估敏感特征的作用。


---

https://www.quora.com/What-is-the-difference-between-tutorials-workshops-and-papers-at-a-conference-e-g-ICML

A tutorial seeks to teach a topic of interest by example and supply the information to complete a certain task. It is more interactive and specific than a book or lecture. Most of the good tutorials are aimed at providing in-depth information on ‘hot’ topics of the field.

A workshop is organized in a conference to promote emerging areas. The idea is that in future, these workshops may become a conference by themselves or by merging other similar workshops. Therefore, the papers presented in workshops are mostly work in progress or that shows preliminary results. Good workshops are mostly peer reviewed.

Full Papers in a conferences are completed research work with significant findings and outcomes. Good conference papers are mostly peer reviewed.

---

ppt
https://static.googleusercontent.com/media/research.google.com/zh-CN//pubs/archive/46178.pdf