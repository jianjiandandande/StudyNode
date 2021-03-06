# Adaboost

AdaBoost是英文`Adaptive Boosting`(自适应增强的缩写)，拿多个分类器来决定一个事，将几个弱的分类器集成在一起了。

## 基本步骤

* 初始化训练数据的权值分布。如果有N个样本，则每一个训练样本最开始时都被赋予相同的权重: 1/N。

* 训练弱分类器。具体训练过程中，如果某个样本点已经被准确地分类，那么在构造下一个训练集中，它的权重就被降低;相反，如果某个样本点没有
被准确地分类。那么它的权重就得至提高。然后，权重更新过的样本集被用于j 练下一个分类器，整个i 练过程如此迭代地进行下去。

* 将各个训练得到的弱分类器组合成强分类器各个弱分类器的训练过程结束后，加大分类误差率小的弱分类器的权重，使其在最终的分类函数中起着
较大的决定作用，而降低分类误差率天的弱分类器的权重，使其在最终的分类函数中起着较小的决定作用。换言之，误差率低的弱分类器在最终分类
器中占的权重较大，否则较小。

## 总结

AdaBoost其实就是通过将多个弱分类器(一些特征与权值的组合，然后对数据进行分类，如果有一些样本没有被准确分类，则根据第一次分类的结果，重新定义一组权重参数，然后对原始样本进行分类，以此类推)
最终给每个分类器分配一个权重参数(可以通过分类器的错误率计算出来)，将此权重参数与各个分类器累乘求和，得到最终的模型。
