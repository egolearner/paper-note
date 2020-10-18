# [Compressing Neural Networks with the Hashing Trick](https://github.com/egolearner/paper-note/issues/8)

2015 ICML
https://arxiv.org/pdf/1504.04788.pdf
这篇论文是 #7  的应用。

## 1. 介绍
之前的论文指出神经网络的权重存在惊人的大量冗余。
HashedNets的做法为，使用hash函数将网络连接随机分配到桶中，分配到i<sup>th</sup>桶的连接共享相同的权重w<sub>i</sub>。

## 4. HashedNets
### 随机权重共享
对于神经网络的层，定义内存上限K<sup>l</sup>: K<sup>l</sup> ≪ (n<sup>l</sup> + 1) × n<sup>l+1</sup>。之前的做法为减少节点数n<sup>l</sup>和n<sup>l+1</sup>，或者量化。论文的思路是随机权重共享，使用一个hash函数将虚拟权重表V<sub>(i,j)</sub>映射到真实权重向量w<sup>l</sup>的下标。
![image](https://user-images.githubusercontent.com/45122959/66256980-4fcebb00-e7c6-11e9-96b1-5d6ecdb061c6.png)


### 特征哈希 v.s. 权重共享
论文证明特征哈希与权重共享是等同的，因此可以用 #7 中sign factor来减少hashing冲突导致的偏差。除了hashing-trick的稀疏性外，论文还用ReLU作为激活函数，一方面有很好的泛化性能，另一方面会导致稀疏性。另外，HashedNets和模型架构正交，其他架构也可以使用。

### 训练

## 7. 结论
![image](https://user-images.githubusercontent.com/45122959/66256989-78ef4b80-e7c6-11e9-87f8-96971301b967.png)

通过将权重的数量虚拟扩展8倍，测试误差降低了50％，从3％到1.61%。论文认为不是正则化的原因，而是权重共享可能真的增强了神经网络的表达能力。