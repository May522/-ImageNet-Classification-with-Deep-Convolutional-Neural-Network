## 翻译-ImageNet-Classification-with-Deep-Convolutional-Neural-Network

### Abstract概要
在ILSVRC-2010竞赛中，我们训练了一个大的卷积神经网络，该网络把包含120万张高分辨率图像的ImageNet数据集分成1000个类。在测试集上，我们的top-1 和top-5 error分数分别是37.5%和17.0%，这个成绩超过了之前表现最好的模型。该神经网络包含六千万的参数和65万个神经元，5个卷积层（有些卷积层后带有最大池化层max-pooling layer），以及三个全连接层，最后的输出层含有1000个神经元。为了加快训练速度，我们使用non-saturating 神经元和GPU。我们还引入了最新提出的dropout层，它对降低过拟合很有效。在ILSVRC-2012竞赛中，我们在该模型的基础上稍做了改变，最终使得top-5 test error rate达到了15.3%。

### 1 Introduction简介
目前的物体识别都会用的机器学习方法。为了改善性能，我们收集了更大的数据集，学习更强大的模型，探索更好的方法避免过拟合。直到最近，我们的数据集还是万级别的，例如NORB,Caltech-101,CIFAR-10/100。对于这种万级别的小型数据集对于简单的识别任务还是足够的，尤其是在用label-preserving变换来增强数据集的情况下。例如，MNIST数字识别任务，模型的性能(<0.3%)已经能达到人类水平。但是在现实环境中的物体有很多的变化variability，因此要学习识别他们需要用更大的训练数据集。实际上，这种小型数据集的缺陷早已经认识到了，只是直到最近才有可能收集这种带有标签的大型百万级数据集。最新产生的较大的数据集包括LabelMe和ImageNet，LabelMe包含几十万张全分割fully-segmented图像，ImageNet数据集包含1500万张标注好的高分辨率的图像，超过22000个类别。

要从近百万张图像中学习上千个分类，我们需要的模型必须拥有强大的学习能力。然而物体识别是一个很复杂的问题，甚至像ImageNet这样大的数据集仍然不能满足条件。因此在建立模型时，我们需要一些先验知识prior knowledge来弥补那些缺失的数据。卷积神经网络CNN就是这样一种模型，你可以通过改变它的深度depth和宽度breadth来改变网络的容量。这种网络对图像的性质做了一种strong and mostly correct assumptions，即像素的局部相关性以及统计上的稳定性stationarity of statistics and locality of pixel dependencies。和具有同样层数的标准反馈神经网络standard feedforward neural networks相比，卷积神经网络具有更少的参数和连接，这使得卷积神经网络训练起来更容易。

尽管卷积神经网络有很多吸引人的优点，但是要应用在大规模的高分辨率图像上的代价很高，让人望而却步。幸运的是，当前的GPU与高度优化的2D卷积功能两者结合起来，使得GPU强大到可以训练卷积神经网络CNN。并且新的的数据集包含足够多的带标签的图像，可以避免生成严重的过拟合。

接下来列出本篇文章所做的工作。ILSVRC-2010和ILSVRC-2012的竞赛用的数据集是ImageNet的子集，我们在该数据集上训练了一个目前最大的卷积神经网络。在所有基于该子数据集训练的网络中我们的网络表现是最好的。我们写了一个高度优化的可以应用在GPU上的2D卷积，还写了一些其他的运算操作都集成在训练卷积神经网络的过程中。这些应用我们都已经公开。我们的网络包含很多新颖的特征，可以提高网络性能，减少训练时间，这会在第三部分介绍。我们的网络比较大，即便是用120万带标签的图像来训练，仍然会产生严重的过拟合。因此我们使用几个有效的方法来防止过拟合，这会在第4部分介绍。最终，我们的网络包含5个卷积层和3个全连接层。这样的深度看起来似乎很重要，因为我们发现移除任何一个卷积层都会降低网络性能，其中每个卷积层包含的参数量占整个模型参数量的1%还不到。

网络的大小受当前GPU的内存大小限制，还受我们能够忍受的训练时间限制。我们的网络在两个GTX 580 3GB的GPU上训练了5到6天。所有我们的实验结果表明，如果有更快的GPU和更大的数据集，网络性能可以得到进一步改善。

### 2 The Dataset数据集
ImageNet数据集含有1500万张标注好的高分辨率图像，这些图像属于22000个类别。这些图像是从网络中收集的，然后用亚马逊的Mechanical Turk crowd-sourcing tool人工进行标注。从2010年起，每年都会举行ImageNet Large-Scale Visual Recognition Challenge(ILSVRC)竞赛。ILSVRC从ImageNet中选择了1000类图像，每类图像包含一千张左右。总的来说，ILSVRC数据集共包含大约120万张训练图像，5000张验证图像validation images，和150000张测试图像testing images。

ILSVRC-2010是所有ilsvrc中唯一一个测试集标签可以利用的一次竞赛。我们大部分的实验都是基于这一版本的数据集来进行的。我们也参加了ILSVRC-2012年的竞赛，该版本的数据集中测试集的标签是不可用的。在本文的第六部分会展示该版本数据集下我们的模型的表现结果。在比赛中，通常展示两种错误率，top-1,top-5。

ImageNet包含各种分辨率图像，可网络需要输入的图像是固定的维度。因此我们对图像进行下采样成一个固定的分辨率256 X 256。如果是一个矩形图像，我们首先把图像缩放成短边为256，然后从缩放后的图像中心裁剪出256 x 256的图像。我们把训练图像的每个像素减去所有训练图像的平均值，除此之外，没有对训练集做其他预处理。因此，我们是在RGB图像上训练网络的。

### 3 框架 Architecture











