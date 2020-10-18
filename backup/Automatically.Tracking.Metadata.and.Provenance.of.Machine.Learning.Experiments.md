# [Automatically Tracking Metadata and Provenance of Machine Learning Experiments](https://github.com/egolearner/paper-note/issues/14)

amazon的论文
http://learningsys.org/nips17/assets/papers/paper_13.pdf

## 1 Introduction

问题：没有标准的方式来存储和管理机器学习实验，导致结果无法比较；不能简单存储模型产物，还需要存储血缘关系、特征变换方式等。

## 2 系统设计

### 架构

底层提供JVM和Python的rest client，上层提供针对ML库的支持自动抽取元数据的高层客户端，比如支持抽取spark中pipeline和DataFrame的结构、mxnet中的网络结构。
![image](https://user-images.githubusercontent.com/45122959/89712337-cc23c300-d9c2-11ea-814b-6209ac65f524.png)


### 数据模型

核心问题是通用性与schema可解释性的权衡。采用折衷方案，采用强制存储血缘信息和ML特定的属性的schema，但又提供足够的灵活性让用户存储任意的应用相关注解。

![image](https://user-images.githubusercontent.com/45122959/89712340-cf1eb380-d9c2-11ea-947c-f27322ff5f48.png)


使用声明式的方式，存储artifacts的metadata而非产出的代码（建议代码在git中，revision id作为metadata的一部分），仅存储实际输入数据或序列化模型参数的指针。

- The metadata for datasets is captured by the Dataset-Metadata entity, which stores name and version of the dataset, its column names and their respective types, as well as a URI pointer to the actual storage location of the dataset.
- Models are represented by the Model-Metadata entity which stores a link to the metadata of the input dataset to capture the lineage of the computation. We provide a name for the model, its hyperparameters and their types, as well as the name of the associated learning algorithm and its version. Furthermore, we represent the metadata of the data transformations describing the transformation of the input data by this model via a graph of Transform entities. Note that we do not explicitly represent the model’s parameters in order to ensure generality of our system; we just store a pointer to a serialized version of the parameters.
- Training-Run entity tracks live execution and captures statistics like the training loss over time and the computational environment
- Evaluation data is represented by the Evaluation entity, for which we allow users to store customizable scores and tags.

## 3 Automating Metadata Extraction

### spark

SparkML pipeline的架构允许我们自动跟踪schema变化和pipeline operator的参数。为了跟踪pipeline中的数据变换细节，我们抽取输入数据帧的schema，重放每个pipeline stage的transformSchema，记录结果的schema变化。将数据变换的DAG建模为数据库中Transformation graph。

### sk-learn

和spark类似，但sk-learn没有schema变换信息。我们得到数据变换的DAG和参数。

### mxnet

和前面相比，dl框架的元数据可以深入到算子级别。可以得到op的op, name, input, attr，将其转换为Transform的DAG。