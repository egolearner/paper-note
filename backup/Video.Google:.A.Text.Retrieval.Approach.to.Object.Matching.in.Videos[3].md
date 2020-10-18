# [Video Google: A Text Retrieval Approach to Object Matching in Videos[3]](https://github.com/egolearner/paper-note/issues/17)

[http://www.robots.ox.ac.uk/~vgg/publications/papers/sivic03.pdf](http://www.robots.ox.ac.uk/~vgg/publications/papers/sivic03.pdf)

google ICCV2003关于视频搜索的论文，主要是将文本检索中创建索引的思想推广到了视频搜索中。

## 2 定义单词

对原始视频创建视角无关的描述，同时使用了Shape Adapted和Maximally Stable的方式来生成visual words（论文后面提到这两种方式是互补的，SA+MS效果更好），然后对这两种区域使用SIFT方法提取特征。对于出现不超过3帧的区域认为是噪声直接丢掉。

## 3 生成词表

对每个区域使用K-means来做量化（vector quantization），使用Mahalanobis distance来度量距离。

SA和MS独立计算，它们可以看作是描述同一场景的不同词汇。

## 4 visual indexing

文本检索中使用tf-idf来定义权重

$$t_i = (n_{id}/n_d)*log(N/N_i)$$

公式前半部分是词频，单词在文档中出现的频率；后半部分是逆文档频率，文档个数与出现单词的文档个数的商再取log。

## 6 object retrieval

出现最高的5%和最低的10%的visual word定义为stop list。

空间一致性（类似于文本检索中的单词顺序），针对匹配点/区域，检查附近是否有15个已匹配的点/区域，少于则rejected。

> The visual words learnt for Lola are used unchanged for the Groundhog Day retrieval.

不同电影可以使用相同的visual words，这是普适的还是特例？貌似是后者？

## take way/感想

- 图像相关的完全不懂。
- 新瓶装旧酒，关键要融会贯通。

## 参考资料

- demo网站 [http://www.robots.ox.ac.uk/~vgg/research/vgoogle/](http://www.robots.ox.ac.uk/~vgg/research/vgoogle/)
- 别人写的review [https://blog.csdn.net/mqfcu8/article/details/44907109](https://blog.csdn.net/mqfcu8/article/details/44907109) [https://blog.csdn.net/recognition/article/details/7901101](https://blog.csdn.net/recognition/article/details/7901101)
- 华盛顿大学的ppt [https://courses.cs.washington.edu/courses/cse455/10au/notes/Sivic.pdf](https://courses.cs.washington.edu/courses/cse455/10au/notes/Sivic.pdf)

---

test2