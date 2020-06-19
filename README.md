## 翻译-ImageNet-Classification-with-Deep-Convolutional-Neural-Network

### Abstract概要
在ILSVRC-2010竞赛中，我们训练了一个大的卷积神经网络，该网络把包含120万张高分辨率图像的ImageNet数据集分成1000个类。在测试集上，我们的top-1 和top-5 error分数分别是37.5%和17.0%，这个成绩超过了之前表现最好的模型。该神经网络包含六千万的参数和65万个神经元，5个卷积层（有些卷积层后带有最大池化层max-pooling layer），以及三个全连接层，最后的输出层含有1000个神经元。为了加快训练速度，我们使用non-saturating 神经元和GPU。我们还引入了最新提出的dropout层，它对降低过拟合很有效。在ILSVRC-2012竞赛中，我们在该模型的基础上稍做了改变，最终使得top-5 test error rate达到了15.3%。

### 1 Introduction简介
目前的物体识别都会用的机器学习方法。为了改善性能，我们收集了更大的数据集，学习更强大的模型，探索更好的方法避免过拟合。直到最近，我们的数据集还是万级别的，例如NORB,Caltech-101,CIFAR-10/100。对于这种万级别的小型数据集对于简单的识别任务还是足够的，尤其是在用label-preserving变换来增强数据集的情况下。例如，MNIST数字识别任务，模型的性能(<0.3%)已经能达到人类水平。但是在现实环境中的物体有很多的变化variability，因此要学习识别他们需要用更大的训练数据集。实际上，这种小型数据集的缺陷早已经认识到了，只是直到最近才有可能收集这种带有标签的大型百万级数据集。最新产生的较大的数据集包括LabelMe和ImageNet，LabelMe包含几十万张全分割fully-segmented图像，ImageNet数据集包含1500万张标注好的高分辨率的图像，超过22000个类别。

要从近百万张图像中学习上千个分类，我们需要的模型必须拥有强大的学习能力。然而物体识别是一个很复杂的问题，甚至像ImageNet这样大的数据集仍然不能满足条件，因此在建立模型时，我们需要一些先验知识prior knowledge来弥补那些缺失的数据。卷积神经网络CNN就是这样一种模型，你可以改变它的深度depth和宽度breadth来改变网络的容量。这种网络对图像的性质做了一种strong and mostly correct assumptions，即像素的局部相关性以及统计上的稳定性stationarity of statistics and locality of pixel dependencies。和具有同样层数的标准反馈神经网络standard feedforward neural networks相比，卷积神经网络具有更少的参数和连接，这使得卷积神经网络训练起来更容易。

