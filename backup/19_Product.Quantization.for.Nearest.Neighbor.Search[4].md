# [Product Quantization for Nearest Neighbor Search[4]](https://github.com/egolearner/paper-note/issues/19)

PAMI 2011

[https://lear.inrialpes.fr/pubs/2011/JDS11/jegou_searching_with_quantization.pdf](https://lear.inrialpes.fr/pubs/2011/JDS11/jegou_searching_with_quantization.pdf)

论文主要有两个贡献

1. 提出了product quantization的方法，从对向量做量化改为向量分段后做量化，解决了前者需要特别大的码表及内存需求高的问题。类比来说，1000=10*10*10，从存储1000个centroid变为了存储3组各10个centroid。
2. 使用倒排索引。

## 1. Introduction

为了降低量化噪音，centroid个数需要足够大，比如2^64。带来的问题是：需要的样本大，算法复杂，内存需求高。

## 2. Product Quantization

- 将向量分为m个不同的子向量，每个子向量使用不同的量化器分别做量化。如原始向量为1024维，分为8组后，每个子向量为128维。
- 每个量化器有一个码表C，整体向量的码表相当于$C = C_1 \times ... \times C_m$，但实际只需要存储每个子量化器的码表。
- 每个子向量量化为变为1个整数，如果每组子向量有256个中心点，需要log2(256)=8位存储。因此空间需求从1024个float变为8个8bit整数。

![image](https://user-images.githubusercontent.com/45122959/97783755-b7775580-1bd4-11eb-9f2a-823ab898f82a.png)

论文中指出，位数固定下，使用带多个中心点的少量的子量化器比带少量中心点的多个子量化器效果要好。给了一个经验数据是，256个中心点，分为8组。

## 3. 检索

两种距离计算方法 。

![image](https://user-images.githubusercontent.com/45122959/97783763-be9e6380-1bd4-11eb-983a-ec3bf800dfaf.png)


对称距离(SDC)：字典和检索向量都做量化。
![image](https://user-images.githubusercontent.com/45122959/97783770-ce1dac80-1bd4-11eb-930d-ccee0c9eaa3a.png)

其中，预先计算好每组子量化器中各个中心点的距离，计算两个量化的子向量的距离时只需要查表。

非对称距离(ADC)：只有字典做量化。精度高于SDC，所以论文推荐用ADC。

![image](https://user-images.githubusercontent.com/45122959/97783774-d4ac2400-1bd4-11eb-8719-47d65aa9b7a2.png)

其中，在线先计算好检索向量到每个中心点的距离，即$d(u_j(x), c_{j,i})^2$，j为组索引，i为中心点的索引，后续只需要查表。

## 4 非穷举检索

![image](https://user-images.githubusercontent.com/45122959/97783780-e097e600-1bd4-11eb-917f-cffc765c9285.png)

先计算粗粒度量化器，然后计算原始值与量化值的残差

$$r(y) = y - q_c(y)$$

然后对残差做product quantization。相比于直接对原始的向量做乘积量化，误差更小。直观的解释是，残差相比原始的向量分布更加集中，因此误差更小。误差更小应该有两个原因：
* [这里](http://yongyuan.name/blog/vector-ann-search.html)讲到的多阶段失量量化思想，centroid向量+残差向量更接近原始的向量。
* 减中心点之前，数据分布在不同的中心点附近。减去中心点后，数据都分布在坐标原点附近，再做量化的话误差会更小。

索引算法

![image](https://user-images.githubusercontent.com/45122959/97783790-eb527b00-1bd4-11eb-990c-c82d7871a6c0.png)

检索算法

检索向量x与其最近的邻居可能量化到不同的中心点，因此需要对w>1组索引到的向量做计算。

![image](https://user-images.githubusercontent.com/45122959/97783794-f0172f00-1bd4-11eb-8718-fea75998a952.png)


---

参考资料
* http://yongyuan.name/blog/vector-ann-search.html
* http://vividfree.github.io/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/2017/08/05/understanding-product-quantization

---

PQ的核心在于将高维向量空间分解为子空间的笛卡尔积，然后分别对子空间做量化。