# [ModelHub: Lifecycle Management for Deep Learning[4]](https://github.com/egolearner/paper-note/issues/16)

17年ICDE论文，引用不多。

[https://www.cs.umd.edu/class/spring2016/cmsc396h/downloads/modelhub.pdf](https://www.cs.umd.edu/class/spring2016/cmsc396h/downloads/modelhub.pdf)

作者认为的挑战是

- 难以追踪开发的许多模型和/或理解其不同之处。
- 模型开发过程中很多耗时的重复步骤，如在不同的地方加一层，搜索不同的超参数。
- 相似的模型可能多次训练和运行，可能使用其他模型的权重做初始化。
- 深度模型的存储需求大，某些情况下只能保存少数几个快照。
- 不容易共享和重用模型。

![image](https://user-images.githubusercontent.com/45122959/93714307-43d53800-fb94-11ea-9e45-af3714bfae4b.png)

ModelHub由3部分构成：

1. a model versioning system (DLV) to store and query the models and their ver- sions
2. a model enumeration and hyper-parameter tuning domain specific language (DQL) to serve as an abstraction layer to help modelers focus on the creation of the models instead of repetitive steps in the lifecycle
3. a hosted deep learning model sharing system (ModelHub) to publish, discover and reuse models from others

### 2.2 DataModel

以层作为模型的基本单元。

VCS数据模型：

- 网络定义
- 权重集合
- 提取的元数据（如超参数、auc、loss等）
- 文件集合（如脚本、数据集）
- 可读的模型版本。

### 2.3 Query Facilities

- query

`dlv list [--model_name] [--commit_msg] [--last]`

- describe

`dlv desc [--model_name | --version] [--output]`

展示模型元数据，如network definition, learnable parameters, execution footprint (memory and runtime), activations of convolution networks, weight matrices, and evaluation results across iterations

- compare

`dlv diff [--model_names | --versions] [--output]`

主要是`desc`的结果side-by-side的对比。

- evaluate

`dlv eval [--model_name | --versions] [--config]`

使用不同的数据来test模型，或者修改超参数（修改超参不重新训练模型吗？？？）

![image](https://user-images.githubusercontent.com/45122959/93714309-49328280-fb94-11ea-8d62-d8e6e04917aa.png)


DQL支持查询模型，获得模型的一部分(slice)，修改模型，自动搜索超参。想法比较有意思，但感觉意义不大，前两个Query从学习用途来说有点用，但感觉不如直接看代码；Query3似乎不如直接改原始代码，多了DSL的学习成本；Query4似乎是用SQL封装了简单automl和自动评估保留模型。

## Take away

- 层作为展示模型结构的基本单元。引申一下，如果能够自动从代码和模型中产出以层为单元的模型结构图并进行可视化展示可能有助于分享和理解模型，类似tensorboard展示的信息过细，代码又不如模型结构图直观。
- modelhub，可以展示和查询模型的元数据。