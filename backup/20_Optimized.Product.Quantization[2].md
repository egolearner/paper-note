# [Optimized Product Quantization[2]](https://github.com/egolearner/paper-note/issues/20)

[http://ieeexplore.ieee.org/abstract/document/6678503/](http://ieeexplore.ieee.org/abstract/document/6678503/)

2013年CVPR论文，对[PQ](19) 的优化。

PQ的论文实验中提到，对于SIFT/GIST数据集structured分组结果要优于natural和random分组。这篇论文主要是讲在没有先验知识的前提下如何对向量空间做线性变换以产生最优的PQ结果。

论文将K-Means、Product Quantization、Iterative Quantization统一分析，将不同算法的特定配置定义为优化common目标函数的限制条件，而不是将配置定义到目标函数之中。定义Quantization distortion为目标函数。

![image](https://user-images.githubusercontent.com/45122959/99188857-21792880-2799-11eb-9b6b-b8a7358aa03e.png)

### 算法

论文使用正交矩阵R来表示空间分解（即维度的重排序或者变换）。目标函数有两个自由度，PQ的子codebook和空间分解R。

算法1，不需要有任何数据分布假设。

![image](https://user-images.githubusercontent.com/45122959/99188861-2807a000-2799-11eb-8b38-ea0cef0124cc.png)

算法2 假设数据服从高斯分布。第一，如果数据服从高斯分布保证了更简单的算法和最优的结果；第二，提供了初始化算法一的方式；第三，提供了对ANN的两个常用标准的新的理论解释。

1. 使用PCA对齐数据，将特征值(eigenvalue)按照方差由大到小排序。
2. 准备M个空桶
3. 线性选取最大的特征值，分配到有最小的积（？product)的桶中，除非桶满了。

每个桶的特征值提供了组成每个子空间的主成分(principal components)。

> Summary of the parametric solution. Our parametric solu- tion first computes the D x D covariance matrix S of the data and uses Eigenvalue Allocation to generate R. The data are then transformed by R. The PQ algorithm is then performed on the transformed data.

### 感想

看不太懂。