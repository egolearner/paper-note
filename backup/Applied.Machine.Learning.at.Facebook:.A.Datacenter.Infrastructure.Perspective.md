# [Applied Machine Learning at Facebook: A Datacenter Infrastructure Perspective](https://github.com/egolearner/paper-note/issues/1)

https://research.fb.com/wp-content/uploads/2017/12/hpca-2018-facebook.pdf

![image](https://user-images.githubusercontent.com/45122959/48661487-82d46700-eaad-11e8-8d7a-e138b53ebdec.png)
论文主要是关于Facebook的机器学习软硬件架构。
## Machine learning at facebook
### Machine learning model
FB使用了各种各样的模型，其中
* LR和SVM可以高效的训练和预测
* GBDT可以提高准确率，但消耗额外的资源
* DNN精度最高，但也消耗最多的资源
    * MLP(Multi-Layer Perceptrons) 结构化的输入特征，常用于排序
    * CNN 空间处理器，常用于图像处理
    * RNN/LSTM 顺序处理器，常用于语言处理
### ML-as-a-Service Inside Facebook
FB Learner包含三个组件
* FBLearner Feature Store。用于训练和预测的特征中心，可以跨团队分享和发现特征。
* FBLearner Flow。pipeline管理系统，负责执行工作流，在工作流中描述训练和评估模型的步骤及资源需求。另见[这里](https://code.fb.com/core-data/introducing-fblearner-flow-facebook-s-ai-backbone/)。
    > Workflows are built out of discrete units, or operators, each of which have inputs and outputs. The connections between the operators are automatically inferred by tracing the flow of data from one operator to the next and Flow handles the scheduling and resource management to execute the workflow. Flow also has tooling for experiment management and a simple user interface which keeps track of all of the artifacts and metrics generated by each workflow execution or experiment. The user interface makes it simple to compare and manage these experiments.
* FBLearner Predictor，FB的预测引擎，既可以是多租户的服务，又可以是集成到其他后端服务的库。
### Deep Learning Frameworks
Caffe2侧重生产，PyTorch侧重研究，然而最新的[PyTorch 1.0](https://developers.facebook.com/blog/post/2018/05/02/announcing-pytorch-1.0-for-research-production/)已经以Caffe2为后端了，同时适用于生产和研究。

## RESOURCE IMPLICATIONS OF MACHINE LEARNING
### Resource Implications of Offline Training
**Compute Type and Locality**
GPU训练更快，然而CPU更加丰富，而且夜间的使用率极低。
GPU更多用于离线训练，而非在线预测，原因是GPU架构为throughput而非latency优化。
**Scaling Considerations and Distributed Training**
传统上模型训练是单机的，采用多GPU卡数据并行的策略。随着数据量的增大，分布式训练成为必须。
> the updates need to be shared with the other replicas using techniques that provide trade-offs on synchronization (every replica sees the same state), consistency (every
replica generates correct updates), and performance (which
scales sub-linearly),

如果模型异常的大，就需要模型并行。
> the model layers are grouped and distributed to optimize for throughput with activations pipelined between machines

许多情况下，DNN模型的预测一般是单机运行的，机器间分割模型会引入大量的通信。
### Resource Implications of Online Inference
广告排序模型中，sparse embedding层是内存密集的，跑在一个单独的服务上。
随着移动设备算力的增强，部分模型可以直接在移动端做预测。
不同产品的latency需求不同，有些可以返回默认的结果，后续预测结束后再进行修正。但对Feeds流和广告场景，latency是必须保证的。

## MACHINE LEARNING AT DATACENTER SCALE
### Getting Data to the Models
分为data workload和training workload，其中前者复杂，ad-hoc，业务相关，变化很快；后者比较规律，稳定，高度优化，倾向有干净的环境。据此分为reader和trainer，reader负责读取数据，处理和压缩后发送给trainer；后者负责快速和高效的执行训练。
通过优化压缩，调度算法，数据和算力的布局来降低网络传输。
### Leveraging Scale
利用全球数据中心的机器来弹性使用。
### Disaster Recovery
最开始GPU只在一个数据中心，后来多数据中心建设。

## FUTURE DIRECTIONS IN CO-DESIGN: HARDWARE, SOFTWARE, AND ALGORITHMS
同步SGD需要all-reduce操作，特点是当采用递归doubling或halfing的策略时（什么意思？）带宽需求随递归深度指数递减。据此可以采用分层的设计，底层的节点高带宽，上层的节点低带宽。
异步SGD通过PS来同步，节点发布更新到PS，PS聚合并分发回去。
一种混合的架构是，超级节点内部采用异步更新，因为本地节点的带宽高延迟低，跨超级节点采用同步更新。

## Take away
FB的机器学习实践，数据与算力的合理分布，使用闲置的CPU用于训练，关于SGD混合架构的讨论。

---

补一个相关的ppt
ML at Facebook: An Infrastructure View
https://www.matroid.com/scaledml/2018/yangqing.pdf