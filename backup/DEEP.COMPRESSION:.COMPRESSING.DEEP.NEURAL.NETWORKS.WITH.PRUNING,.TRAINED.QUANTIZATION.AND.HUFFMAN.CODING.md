# [DEEP COMPRESSION: COMPRESSING DEEP NEURAL NETWORKS WITH PRUNING, TRAINED QUANTIZATION AND HUFFMAN CODING](https://github.com/egolearner/paper-note/issues/9)

2016 ICLR
https://arxiv.org/abs/1510.00149

## 1. 介绍
深度学习部署到端上的两大挑战：模型大小（App大小）和耗电。移动端的耗电主要由访问内存决定。
![image](https://user-images.githubusercontent.com/45122959/66324497-28a7f300-e958-11e9-921e-77a2d41f3cb3.png)

为了减小模型，论文思路为先对网络剪枝，去掉冗余的连接，只保留最有信息的连接；然后对权重做量化，多个连接可以共享相同权重；最后用霍夫曼编码来利用有效权重的有偏分布。
剪枝和训练量化不会互相干扰，可以共同使用来达到惊人的高压缩率。

## 2. 网络剪枝
1. 先用正常的模型训练来学习连接信息
2. 对低权重的连接剪枝，去除权重小于阈值的连接。
3. 重新训练网络，学习剩余稀疏连接的最终权重。

剪枝对于AlexNet和VGG-16分别减少了9到13倍。
论文的主要目标是减少模型大小，因此用compressed sparse row(CSR)或CSC来存储稀疏模型权重，需要2a+n+1个数，其中a为非零元素个数，n为列数或行数。为了进一步减少大小，存储相邻下标的差，而非绝对值。卷积层和全连接层分别用8位和5位来存储下标差。如果下标差超过位数，则补0。

## 3. 训练量化和权重共享
![image](https://user-images.githubusercontent.com/45122959/66324531-3bbac300-e958-11e9-9366-58d123fadbbf.png)

将权重量化成n个桶，相同桶共享相同的权重值，对每个权重只需要存储到共享权重表中的小下标。模型训练时，所有的梯度按桶分组加在一起，乘以学习率，然后从上一迭代的矩心(centroid)中减去。

### 权重共享
论文使用k均值聚类来确定每层网络的权重，不同层不会共享权重。将n个原始权重W = {w<sub>1</sub>, w<sub>2</sub>, ..., w<sub>n</sub>}分到k个聚类C = {c<sub>1</sub>, c<sub>2</sub>, ..., c<sub>k</sub>}，要求
![image](https://user-images.githubusercontent.com/45122959/66324689-7de40480-e958-11e9-82cc-f48dd6fee17a.png)

 #8 在训练模型之前根据hash随机确定权重共享，不能反映模型和训练数据的特点，而本论文在训练模型后权重共享，共享权重近似于原始网络。

### 共享权重初始化
![image](https://user-images.githubusercontent.com/45122959/66324720-8e947a80-e958-11e9-8a98-dc357a5e54bd.png)

论文分析了3种初始化方式：
* forgy(随机)。从数据集随机选k个点。双峰分布有两个尖峰，forgy倾向于集中在两个尖峰附近。？？？
* 基于密度。沿y轴将权重CDF线性分段，然后找到与CDF的交点，落到x轴上。使两个尖峰附近更密，但总体比forgy分散。
* 线性。将原始权重[min, max]线性分段，不受权重分布影响，相比前两者最为分散。

实验表明第3种方式效果最好，论文的解释是大权重比小权重作用更大，前两种初始化方式有很少的矩心有大的权重绝对值，而线性初始化从min到max平均分布，有利于保留大的权重。

## 4. 霍夫曼编码
使用霍夫曼对下标差编码，进一步节省20-30%的存储。

## 结果
AlexNet
![image](https://user-images.githubusercontent.com/45122959/66324773-a835c200-e958-11e9-8d3d-00fac5dad29d.png)
