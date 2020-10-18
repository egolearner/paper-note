# [TensorFlow Estimators: Managing Simplicity vs. Flexibility in High-Level Machine Learning Frameworks[5]](https://github.com/egolearner/paper-note/issues/6)

# 1 引文
对简洁性(Simplicity)的定义：
* 模型架构已知时，定义模型不需要基础性的新技术
* 实验模型特征是透明的

对健壮性(Robustness)的定义，既指软件开发过程，又指软件产品，对框架健壮性的定义为使用框架容易编写正确而高质量的软件，难以编写有问题或低效的软件。
Estimator的目标是给用户提供容易使用的工具，而不妨碍用户使用Tensorflow的通用功能。因此没有提供DSL，而是提供了通用代码的最佳实践的实现，用户不需要再写类似training loop的代码。作者也提到，Estimator的借口借鉴了Scikit-Learn。

# 2 总体设计
Estimator构建在Tensorflow之上，继承了一些设计模式：优先使用函数或闭包而非对象，经常使用callback，Layer函数是Tensor进Tensor出的。
Estimator的需求：
* 总体上简化模型构建，提供鼓励最佳实践和指导用户生产级实现的工具
* 实现最常用的机器学习模型架构
* 向下游框架和基础架构开发者提供接口。

# 3 组件
## 3.1 Layers
Layer：可重用的代码，可以简单如全连接网络的层，也可以复杂如完整的inception网络。
实现Layer的最佳实践：
* Layer包在variable_scope中，以保证在TensorBoard中恰当的分组。
* Layer的组成变量通过get_variable获取，以便可以在模型的不同部分重用或共享。
* 所有Layer假定第一维是batch，可以接收可变的batch输入。

特殊的Layer:
* loss 接收输入、标签、权重并返回标量loss的函数。
* Metrics 接收标签、预测结果、可选的权重并返回类似log-likelihood, accuracy, mean squared error的指标。可以在多个mini batch间聚合计算。Metrics返回两个Tensor: 
    * update_op 在每个mini batch运行，不返回值，仅仅更新中间变量，聚合mini batch的最新信息。
    * value_op 使用中间状态计算最终的指标并返回回去。

## 3.2 Estimator
Estimator接口包括4个函数：
* `train` 给定训练数据，训练模型
* `evaluate` 在测试集数据上计算评估指标
* `predict` 给定训练后的模型，用新数据做预测。
* `export_savedmodel` 导出SavedModel模型。

配置Estimator时向构造函数传递`model_fn`回调函数，上面的4个函数之一执行时，Estimator创建Tensorflow图，建立输入流水线，使用合适的参数调用`model_fn`以生成表示模型的图。Estimator包含必要的代码以运行training或evaluation loop，执行预测，或导出模型。
Estimator向用户隐藏了Graph或Session。构造函数接收`RunConfig`配置对象，以获取环境信息，如可用worker数，checkpoint保存频率。
为了保证封装性，Estimator每次函数调用都创建新的图，并可能从checkpoint中恢复。重建图开销很高，虽然可以缓存，但Estimator显式重建图，牺牲性能换取简洁，防止用户写出在循环中调用estimator函数的低效代码。

**以input_fn指定输入**
input_fn期望返回两个字典，一个包含输入Tensor，一个包含标签Tensor。将核心模型与输入处理解耦方便用户切换数据集。
**以model_fn指定模型**
使用一个回调函数返回训练、评估、预测用的OP，优点是鼓励模型开发者只写一次模型，防止写多个函数引入不一致。
**以Head指定输出**
Head API是模型最后的隐层之后的部分的抽象，以简化`model_fn`的编写。Head知道如何计算loss、相关评估指标、预测及预测的元数据。
**执行计算**
Estimator实现并控制training loop，自动将Variable赋给PS以简化分布式计算。
**以Hook做代码注入**
Hook允许用户定义Session创建、迭代开始或结束、训练结束时的自定义行为。

## 3.3 内置Estimator
内置Estimator继承自Estimator，仅覆盖构造函数，主要被限制在定义model_fn。内置Estimator的开发者成为Estimator的用户。

# 4 分布式执行
当前Estimator多副本训练的主要模式是between graph复制和异步训练。

# 5 案例研究和推广
在Google迅速推广。
![image](https://user-images.githubusercontent.com/45122959/64075805-8446e700-ccef-11e9-8e25-787aa072d84a.png)
