# [Feature Hashing for Large Scale Multitask Learning](https://github.com/egolearner/paper-note/issues/7)

2009 ICML
https://arxiv.org/pdf/0902.2206.pdf
## 介绍
hashing-trick：将高维输入向量映射到低维特征空间。
> Φ: R<sup>d</sup> -> R^<sup>m</sup> where m << d

优点是保留稀疏性，且没有存储成本。

## Hash Functions
![image](https://user-images.githubusercontent.com/45122959/66256317-2fe7c900-e7bf-11e9-90ce-a89cbb235245.png)

sign hash函数`ξ: N -> {+1/-1}`的作用是消除碰撞导致的偏差，本质上是做了两次哈希。

## 应用
论文中主要以个性化邮件过滤器为例，垃圾邮件的标签数据不多，无法只对每个用户训练过滤器，而是每个用户的个性化过滤器加一个全局过滤器。
![image](https://user-images.githubusercontent.com/45122959/66256328-50178800-e7bf-11e9-91c4-87140bfe250f.png)
φ<sub>0</sub>为全局hash函数，φ<sub>u</sub>为个性化hash函数。
用户的个性化hash函数φ<sub>u</sub>为hash(concat(uid,word))
![image](https://user-images.githubusercontent.com/45122959/66256382-d46a0b00-e7bf-11e9-92fa-296041b483a1.png)

如上图所示，使用22位或更多的hash-table后，个性化的过滤器减少了30％的垃圾邮件。