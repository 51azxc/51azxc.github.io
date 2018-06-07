---
title: 译:使用Tensorflow建立一个卷积神经网络
date: 2018-05-29 15:43:44
tags: ["tensorflow","python"]
categories: ["python","tensorflow"]
---

> [A Guide to TF Layers: Building a Convolutional Neural Network](https://www.tensorflow.org/tutorials/layers)

Tensorflow的[`layers`模块](https://www.tensorflow.org/api_docs/python/tf/layers)提供了一些高等API用于更加容易的构建神经网络。它提供了方便创建密集（全连接）层/卷积层，添加激励函数，及使用正则化解决过拟合等一系列方法。在这个教程中，你将学习到如何使用`layers`创建一个用于识别MNIST手写数字集合的卷积神经网络。

<!-- more -->

![0123456789](https://tensorflow.google.cn/images/mnist_0-9.png)

**[MNIST数据集](http://yann.lecun.com/exdb/mnist/)包含了60,000个训练用例及10,000个测试用例，它们由内容为0-9个手写数字，大小为28*28像素的单色图片组成。**

----

### 准备开始
让我们首先开始创建一个Tensorflow程序的`cnn_mnist.py`文件，加入如下代码：
```python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function

# 引入基本组件
import numpy as np
import tensorflow as tf

tf.logging.set_verbosity(tf.logging.INFO)

# 我们的程序逻辑将写于此

if __name__ == "__main__":
  tf.app.run()

```

当完成这部分教程之后，你将会添加构建、训练、评估卷积神经网络的代码于上文件之中。完整的代码可以在[这里](https://www.github.com/tensorflow/tensorflow/blob/r1.8/tensorflow/examples/tutorials/layers/cnn_mnist.py)查看。

### 卷积神经网络简单介绍

卷积神经网络(CNNs)是目前用于图片分类任务中最先进的模型架构。卷积神经网络通过过滤图片的原始数据以提取出一系列更高级别的特征，并加以学习用于构建分类模型。他包含了三个部分：

* **卷积层**:将指定数量的卷积滤镜单元应用于图像中，然后在每个子区域中执行一组数学运算操作从而生成一个单一值放入到输出特征映射集合中。卷积层通常应用ReLU激活函数将非线性因素引入到模型中。
* **池化层**:为了减少处理时间，池化层会对由卷积层提取的图像数据进行[向下采样](https://en.wikipedia.org/wiki/Convolutional_neural_network#Pooling_layer)，这样可以减小数据的空间大小。常用的池化算法是最大池化，它将输出特征映射集合划分为若干个子区域（例如，2×2像素区块），提取这些区域的最大值并丢弃其他值。
* **密集（全连接）层**:对由上述两个模块输出的特征进行分类。在一个全连接层中，层中的每个节点都连接到上一层的每个节点。

通常来说，一个卷积神经网络由一叠执行特征提取的卷积模块构成。每个卷积模块都是由一个卷积层接一个池化层组成(这样可以更完整的获取图片特征)。最后一层卷积模块一般会接一个或多个用于执行分类操作的全连接层。CNN中的最终全连接层包含模型中每个目标分类的单个节点（模型可能预测的所有可能性的分类），使用[softmax](https://en.wikipedia.org/wiki/Softmax_function)激活函数为每个节点生成一个0-1的值(这些值最终相加总和为1)。这些值就是给定图像上区域落在每个目标分类的可能性的相对测量值。

> 如果想要更全面的了解卷积神经网络，可以参考斯坦福大学的[ Convolutional Neural Networks for Visual Recognition course materials](https://cs231n.github.io/convolutional-networks/).

----

### 构建分类MNIST数据的卷积神经网络

让我们按以下架构开始构建：
1. **卷积层 #1**：以5x5为单位面积大小提取深度为32的特征集合，并且添加ReLU激活函数。
2. **池化层 #1**：使用最大池化算法以2x2为单位面积，步数为2对卷积层#1得到的输出特征缩减一半。
3. **卷积层 #2**：以5x5为单位面积大小提取深度为64的特征集合，并且添加ReLU激活函数。
4. **池化层 #2**：继续池化层#1的操作，将输出特征再压缩一半。
5. **密集层 #1**：将输出特征组合成包含1024个神经元的全连接层，同时使用驱除40%的输出特征以防止过拟合。
6. **密集层 #2（逻辑层）**：包含10个神经元，每个神经元表示0-9这10个数字的目标分类。

对于构建上述三种不同类型的层，[`tf.layers`](https://www.tensorflow.org/api_docs/python/tf/layers)模块提供了下列便捷方法：

* `conv2d()`：构建一个二维的卷积层。参数包括输出特征维度(filters),卷积内核大小(kernel size),卷积填充方式(padding)及应用激活函数(activation function)。
* `max_pooling2d()`：使用最大池化算法构建一个二维池化层。参数包括单位池化大小(filter size)及步幅(stride)。
* `dense()`：构建一个密集层。参数包括神经元个数及激活函数。

上述所有的方法都接受一个张量(tensor)作为输入值然后输出一个经过变换的张量。这样每层之间就可以通过接收上一层返回的输出值作为输入值这样的方式很容易的组合在一起。

继续添加`cnn_model_fn`函数，此函数符合TensorFlow的Estimator API所期望的接口（后边有关于此部分的更多[内容](https://www.tensorflow.org/tutorials/layers#create_the_estimator)）。`cnn_mnist.py`使用从MNIST数据集中取出的特征数据，标签(labels)及[模型类型](https://www.tensorflow.org/api_docs/python/tf/estimator/ModeKeys)(`TRAIN`,`EVAL`,`PREDICT`)作为参数；函数过程为配置好卷积神经网络模型；返回一个需要在Estimator运行的模型，包含了预测模型，损失率及训练方式：
```python
def cnn_model_fn(features, labels, mode):
  """Model function for CNN."""
  # 输入层 Input Layer
  input_layer = tf.reshape(features["x"], [-1, 28, 28, 1])

  # 卷积层 Convolutional Layer #1
  conv1 = tf.layers.conv2d(
      inputs=input_layer,
      filters=32,
      kernel_size=[5, 5],
      padding="same",
      activation=tf.nn.relu)

  # 池化层 Pooling Layer #1
  pool1 = tf.layers.max_pooling2d(inputs=conv1, pool_size=[2, 2], strides=2)

  # 卷积层 #2 & 池化 #2
  conv2 = tf.layers.conv2d(
      inputs=pool1,
      filters=64,
      kernel_size=[5, 5],
      padding="same",
      activation=tf.nn.relu)
  pool2 = tf.layers.max_pooling2d(inputs=conv2, pool_size=[2, 2], strides=2)

  # 密集层 Dense Layer
  pool2_flat = tf.reshape(pool2, [-1, 7 * 7 * 64])
  dense = tf.layers.dense(inputs=pool2_flat, units=1024, activation=tf.nn.relu)
  dropout = tf.layers.dropout(
      inputs=dense, rate=0.4, training=mode == tf.estimator.ModeKeys.TRAIN)

  # 逻辑层 Logits Layer
  logits = tf.layers.dense(inputs=dropout, units=10)

  predictions = {
      # Generate predictions (for PREDICT and EVAL mode)
      "classes": tf.argmax(input=logits, axis=1),
      # Add `softmax_tensor` to the graph. It is used for PREDICT and by the
      # `logging_hook`.
      "probabilities": tf.nn.softmax(logits, name="softmax_tensor")
  }

  if mode == tf.estimator.ModeKeys.PREDICT:
    return tf.estimator.EstimatorSpec(mode=mode, predictions=predictions)

  # Calculate Loss (for both TRAIN and EVAL modes)
  loss = tf.losses.sparse_softmax_cross_entropy(labels=labels, logits=logits)

  # Configure the Training Op (for TRAIN mode)
  if mode == tf.estimator.ModeKeys.TRAIN:
    optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.001)
    train_op = optimizer.minimize(
        loss=loss,
        global_step=tf.train.get_global_step())
    return tf.estimator.EstimatorSpec(mode=mode, loss=loss, train_op=train_op)

  # Add evaluation metrics (for EVAL mode)
  eval_metric_ops = {
      "accuracy": tf.metrics.accuracy(
          labels=labels, predictions=predictions["classes"])}
  return tf.estimator.EstimatorSpec(
      mode=mode, loss=loss, eval_metric_ops=eval_metric_ops)
```

以下章节将深入解析上述代码部分，包括如何使用[`tf.layers`](https://www.tensorflow.org/api_docs/python/tf/layers?hl=zh-cn)创建卷积神经网络的每一层，以及如何计算损失率，配置训练方法及生成预测值。如果你已经有了CNN与[Tensorflow `Estimator`]()相关使用经验，并且可以直观的明白上述代码，你可以直接跳过这部分，跳至阅读[训练及评估CNN MNIST分类器](https://www.tensorflow.org/tutorials/layers#training_and_evaluating_the_cnn_mnist_classifier)。

#### 输入层(Input Layer)

在`layers`模块中的为二维图像创建卷积及池化层的方法预期的张量输入值默认形状(shape)为`[batch_size, image_height, image_width, channels]`。如果需要改变结构可以使用`data_format`参数。这些参数定于如下：

* `batch_size`: 用于参加训练的样本集个数。
* `image_height`: 图片高度。
* `image_width`: 图片宽度。
* `channels`: 图片颜色通道数。彩色图片为3(RGB), 黑白图片为1(Black)。
* `data_format`: string类型，默认值为`channels_last`，可选值为`channels_first`。前者对应输入形状为`(batch_size, ..., channels)`,后者为`(batch_size, channels, ...)`。

由于我们的MNIST数据集是由28*28像素的黑白图片组成，因此我们需要使用`reshape`函数将输入特征集合转换为为`[batch_size, 28, 28, 1]`这样的构造形状(shape):
```python
input_layer = tf.reshape(features["x"], [-1, 28, 28, 1])
```
这里使用了-1作为`batch_size`，意为根据输入值`features["x"]`动态计算，而其他的属性都指定了固定值。这样我们就可以根据对应情况进行调整`batch_size`。例如，我们提供5个批次的样例给我们的模型，那这样`features["x"]`就会包含了3,920个值(像素)，因此输入层的的形状`[5, 28, 28, 1]`。同理，如果我们需要输入100个样例，那就需要将值为78,400的`features["x"]`转换成`[100, 28, 28, 1]`。


#### 卷积层(Convolutional Layer) #1

我们使用`layers`模块中的`conv2d`方法创建第一个卷积层：
```python
conv1 = tf.layers.conv2d(
    inputs=input_layer,
    filters=32,
    kernel_size=[5, 5],
    padding="same",
    activation=tf.nn.relu)
```
第一个卷积层连接了输入层(`[bacth_size, 28, 28, 1]`),因此`inputs`接收的参数为`input_layer`。

> 如果指定参数`data_format=channels_first`，那`conv2d()`接收的输入层形状为`[batch_size, channels, image_height, image_width]`

`filter`参数定义为卷积过滤器的数量，也就是输出特征集合的深度(此处为32), `kernel_size`指定了需要卷积内核从图片抽离分析的单位面积大小`[height, width]`(此处为[5,5])。如果长宽一致，可以直接指定边长，也就是`kernel_size=5`。

`padding`参数(不区分大小写)指定了卷积层抽取特征的方式，默认值为`valid`。这里我们指定`same`,他会通过在抽取出来的特征体周边填充0这个值使得长宽会与输入层一致,即为28(如果不使用这种方式，最终得到的输出张量形状为24*24)。

`activation`参数指定需要应用在卷积输出值的激活函数。这里通过[`tf.nn.relu`](https://www.tensorflow.org/api_docs/python/tf/nn/relu)来指定激活函数为ReLU。

最终输入层`input_layer`经过`conv2d()`函数而生成的输出值形状(shape)为`[batch_size, 28, 28, 32]`:长宽与输入层的一致，而深度则为32。

#### 池化层(Pooling Layer) #1

接下来，我门需要将第一个池化层接入到上边创建的卷积层后边。我们使用`layer`模块里的`max_pooling2d()`方法构建一个步长为2，单位面积边长为2及执行最大池化算法的池化层：
```python
pool1 = tf.layers.max_pooling2d(inputs=conv1, pool_size=[2, 2], strides=2)
```
这里的`inputs`参数接收的张量(tensor)形状(shape)为`[batch_size, image_height, image_width, channels]`。这里我们的输入张量为`conv1`，就是第一个卷积层输出的数据(`[batch_size, 28, 28, 32]`)。

> `max_pooling2d()`与`conv2d()`一样拥有`data_format`参数，功能与`conv2d()`方法一致。

`pool_size`参数指定了最大池化的滤镜面积`[height, widht]`(这里为[2,2])。当然如果两个维度值一样，可以通过`pool_size=2`这样的方式设置。

`strides`指定了每次跨步的大小。这里设置了步长为2，意味着池化滤镜在提取过程中在高度及宽度上都间隔着2个像素(对于2x2的单位面积来说，刚好不重叠)。如果希望设置长宽为不同的步长，可以通过元组/列表的方式设置(例如，strides=[3,6])。

因为池化层的功能最终将输出张量的形状减小了一半，因此第一个卷积模块输出的张量(tensor)形状(shape)为`[batch_size, 14, 14, 32]`。

#### 卷积层(Convolutional Layer) #2 & 池化(Pooling Layer) #2

第二个卷积模块的构造与上边一样，都是使用了相同的函数来构建，只是这里的`conv2d()`指定了`filter`的数量为64，比上边多增了一倍:
```python
conv2 = tf.layers.conv2d(
    inputs=pool1,
    filters=64,
    kernel_size=[5, 5],
    padding="same",
    activation=tf.nn.relu)

pool2 = tf.layers.max_pooling2d(inputs=conv2, pool_size=[2, 2], strides=2)
```

`conv2d()`接收的`inputs`参数为上边池化层生成的张量`pool1`，`padding`指定为`same`意味着输出的长宽与输入值一致，因此`conv2`的形状(shape)为`[batch_size, 14, 14, 64]`。

`max_pooling2d()`继续将`conv2`的形状长宽进一步压缩一倍，因此`pool2`的形状(shape)为`[batch_size, 7, 7, 64]`。

#### 密集层(Dense Layer)

接下来，我们需要添加一个密集层(包含1024个神经元和ReLU激活函数)到CNN，对由经上述两个卷积模块提取的特征集合进行分类。在连接之前，需要通过`reshape`函数拍平(flatten)特征集合(pool2)，变成`[batch_size, features]`这样的二维形状(shape)：
```python
pool2_flat = tf.reshape(pool2, [-1, 7 * 7 * 64])
```
在`reshape()`方法中，-1表示`batch_size`维度将根据输入数据样例数动态计算。每一个样例都包含7(`pool2`的高度)X7(`pool2`的宽度)X64(`pool2`的通道数)个特征，因此最终输出张量`pool2_flat`的形状(shape)为`[batch_size, 3136]`。

现在我们就可以通过`layers`模块里的`dense()`方法创建一个密集层了：
```python
dense = tf.layers.dense(inputs=pool2_flat, units=1024, activation=tf.nn.relu)
```
`inputs`参数传入的是上边生成的`pool2_flat`特征集合，`units`表示指定1024个神经元，`activation`参数表示指定的激活函数为ReLU。

为了能够提升模型结果的准确性，我们还需要加入`dropout()`方法：
```python
dropout = tf.layers.dropout(
    inputs=dense, rate=0.4, training=mode == tf.estimator.ModeKeys.TRAIN)
```
`rate`参数指定`dropout`比例，这里指定0.4表示在训练中有40%的元素会被随机丢弃。
`training`参数表示当前模型是否运行在训练模式中；`dropout`只会在`training`为`True`的情况下执行。
最终经过了这一神经元层只会输出的张量形状(shape)为`[batch_size, 1024]`。

#### 逻辑层(Logits Layer)

在我们的神经网络中最后一层为逻辑层,将会返回预测的原始数据。我们创建一个包含10个神经元(分别表示0-9)和默认使用线性激活的密集层：
```python
logits = tf.layers.dense(inputs=dropout, units=10)
```
最终我们的CNN输出的张量形状(shape)为`[batch_size, 10]`。

#### 生成预测值

我们的模型最后一层返回的是预测的原始值。我们需要将这些值转换成两种不同的数据格式：

* **predicted class**：表示预测类别为0-9这10个数字。
* **probabilities**：表示每个样例为每个分类的概率。例如这个样本可能是0，1，2等等。

对于给定的样例，我们需要在逻辑张量中找到最高的数值(意味着概率最大)的那一行作为预测类别(就是哪个数字)。通过`tf.argmax`可以很容易的找到这个元素的索引：
```python
tf.argmax(input=logits, axis=1)
```

`input`参数意为希望从哪里(这里是`logits`)提取出最大值,而`axis`表示从哪个轴提取。这里设置为1，就是从`[batch_size, 10]`后边这个参数来获得最大值，而这个索引数字刚好就表示为预测类为0-9哪个数字。

我们可以通过应用`softmax`激活函数来得出准确率：
```python
tf.nn.softmax(logits, name="softmax_tensor")
```

> 这里指定来这个操作名为**softmax_tensor**，接下来我们可以通过这个名字来引用它(在["Set Up a Logging Hook"](https://www.tensorflow.org/tutorials/layersset_up_a_logging_hook)章节中我们将会通过它来记录softmax值)。

我们将我们的预测对象变成一个字典(dict)，传入到一个`EstimatorSpec`对象里并返回它:
```python
predictions = {
    "classes": tf.argmax(input=logits, axis=1),
    "probabilities": tf.nn.softmax(logits, name="softmax_tensor")
}
if mode == tf.estimator.ModeKeys.PREDICT:
  return tf.estimator.EstimatorSpec(mode=mode, predictions=predictions)
```

#### 计算损失率

对于培训与评估之间，我们需要定义一个[损失函数](https://en.wikipedia.org/wiki/Loss_function)来衡量预测值与目标值之间的匹配程度。对于类似MNIST这样的多分类问题，通常用[交叉熵](https://en.wikipedia.org/wiki/Cross_entropy)来衡量损失率。下面的代码用于计算模型运行于`TRAIN`与`EVAL`之间的交叉熵:
```python
onehot_labels = tf.one_hot(indices=tf.cast(labels, tf.int32), depth=10)
loss = tf.losses.softmax_cross_entropy(
    onehot_labels=onehot_labels, logits=logits)
```
我们的`labels`里的张量(tensor)包含的是我们的样例预测值列表，像`[1, 9, ...]`这样。为了能够计算
交叉熵，首先需要将`labels`转换成对应的[单热编码](https://www.quora.com/What-is-one-hot-encoding-and-when-is-it-used-in-data-science):
```
[[0, 1, 0, 0, 0, 0, 0, 0, 0, 0],
 [0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
 ...
```

我们使用[`tf.one_hot`](https://www.tensorflow.org/api_docs/python/tf/one_hot)来进行转换。`tf.one_hot()`有两个必备参数:

* `indices`：决定单热张量(one-hot tensor)中需要被指定为非零值的坐标。例如上述指定为1在内部数组中的索引。
* `depth`：决定单热张量的深度，在这个例子中也就是指定多少个数组。这里设置为10，就是指明0-9这10个数字分类。

接下来指定`onehot_labels`为我们的`labels`的单热张量(one-hot tensor):
```python
onehot_labels = tf.one_hot(indices=tf.cast(labels, tf.int32), depth=10)
```

因为我们的`labels`包括0-9个不同的数字，所以`indices`表示着`labels`值在张量中的位置，刚好与值对应，所以需要转换成整数类型。`depth`设置10表示有10个潜在的分类，代表着每个数字。

接下来，我们需要计算`onehoe_labels`与逻辑层(logits layer)生成（经过`softmax`激活）的预测值之间的交叉熵，我们可以使用`tf.losses.softmax_cross_entropy()`来计算，并且返回一个标量张量`loss`：
```python
loss = tf.losses.softmax_cross_entropy(
    onehot_labels=onehot_labels, logits=logits)
```

#### 配置训练方法

在上一节中，我们定义损失率为CNN最后一层输出的`softmax`值与`labels`之间的交叉熵。接下来我们需要配置我们的模型在训练中不断的优化损失率。我们使用学习率为0.001，优化算法为[随机梯度下降](https://en.wikipedia.org/wiki/Stochastic_gradient_descent)的优化器优化:
```python
if mode == tf.estimator.ModeKeys.TRAIN:
  optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.001)
  train_op = optimizer.minimize(
      loss=loss,
      global_step=tf.train.get_global_step())
  return tf.estimator.EstimatorSpec(mode=mode, loss=loss, train_op=train_op)
```

> 想要更多的了解Estimator模型函数配置培训操作，请参阅["Creating Estimators in tf.estimator."](https://www.tensorflow.org/get_started/custom_estimators)教程中的[Defining the training op for the model](https://www.tensorflow.org/get_started/custom_estimators#defining_the_training_op_for_the_model)部分。

#### 添加评估指标

我们在`EVAL`模式中定义一个字典(dict)用来给我们的模型训练添加准确率指标:
```python
eval_metric_ops = {
    "accuracy": tf.metrics.accuracy(
        labels=labels, predictions=predictions["classes"])}
return tf.estimator.EstimatorSpec(
    mode=mode, loss=loss, eval_metric_ops=eval_metric_ops)
```

----

### 训练及评估CNN MNIST分类器

我们已经编写了MNIST的CNN模型函数，现在我们要准备开始训练及评估它了。

#### 载入训练和测试数据

首先我们在`main()`方法中载入训练和测试数据:
```python
def main(unused_argv):
  # Load training and eval data
  mnist = tf.contrib.learn.datasets.load_dataset("mnist")
  train_data = mnist.train.images # Returns np.array
  train_labels = np.asarray(mnist.train.labels, dtype=np.int32)
  eval_data = mnist.test.images # Returns np.array
  eval_labels = np.asarray(mnist.test.labels, dtype=np.int32)
```

我们将训练特征数据(55,000个手绘数字图像中的原始像素值)及训练标签(labels,每个图像为0-9的对应值)分别存储为名为`train_data`及`train_labels`的[numpy数组](https://docs.scipy.org/doc/numpy/reference/generated/numpy.array.html)中。同样，我们将评估特征数据(10,000张图片)及评估标签(labels)存储为`eval_data`及`eval_labels`的numpy数组中。

> 如果无法加载MNIST数据，可以通过上述的[MNIST官网](http://yann.lecun.com/exdb/mnist/)先下载数据，放入到指定文件夹中(如`MNIST_data`)，然后通过`mnist = tf.contrib.learn.datasets.mnist.load_mnist(train_dir = "MNIST_data/")`这样的方式载入数据。

#### 创建Estimator

接下来，我们需要为模型创建一个`Estimator`(执行高层模型的训练、评估及推理的Tensorflow类):
```python
# Create the Estimator
mnist_classifier = tf.estimator.Estimator(
    model_fn=cnn_model_fn, model_dir="/tmp/mnist_convnet_model")
```

`model_fn`参数指定生成用于训练，评估和预测的模型函数。这里传入的`cnn_model_fn`就是上个段落定义的函数。`model_dir`参数保存模型数据(检查点)的目录(这里指定了一个临时文件路径`/tmp/mnist_convnet_model`,当然你可以随意更改一个你喜欢的路径)。

> 如果想要更深入的了解Tensorflow **Estimator** API,可以参考["Creating Estimators in tf.estimator."](https://www.tensorflow.org/get_started/custom_estimators)教程

#### 设置日志钩子

由于CNN需要花费一段时间进行训练，我们可以设置一些日志用来追踪训练进度。我们可以使用Tensroflow中的[`tf.train.SessionRunHook`](https://www.tensorflow.org/api_docs/python/tf/train/SessionRunHook)来创建一个[`tf.train.LoggingTensorHook`](https://www.tensorflow.org/api_docs/python/tf/train/LoggingTensorHook)。它将记录我们CNN中softmax层的概率值。继续添加以下代码到`main()`：
```python
# Set up logging for predictions
tensors_to_log = {"probabilities": "softmax_tensor"}
logging_hook = tf.train.LoggingTensorHook(
      tensors=tensors_to_log, every_n_iter=50)
```
我们定义一个`tensors_to_log`字典用来存储需要被记录的张量。字典里的每个键值对应着我们想要输出到日志中的标签，相应的标签是TensorFlow图形中张量的名称。这里我们可以通过`probabilities`来找到`softmax_tensor`,这个名字在之前的`cnn_model_fn`中已定义。

> 如果你没有指定具体名称，Tensorflow将会指定一个默认的名字。另外还有两个简单的方法可以发现操作：在[TensorBoard](https://www.tensorflow.org/programmers_guide/graph_viz)可视化你的神经元图,或者开启[TensorFlow Debugger (tfdbg)](https://www.tensorflow.org/programmers_guide/debugger)。

接下来，我们创建`LoggingTensorHook`，`tensors`参数传入`tensors_to_log`。我们设置`every_n_iter = 50`，它表示每隔50次训练之后记录概率值。

#### 训练模型

现在我们准备训练我们的模型，我们需要创建一个`train_input_fn`然后调用`mnist_classifier`的`train()`方法。添加以下内容到`main()`:
```python
# Train the model
train_input_fn = tf.estimator.inputs.numpy_input_fn(
    x={"x": train_data},
    y=train_labels,
    batch_size=100,
    num_epochs=None,
    shuffle=True)
mnist_classifier.train(
    input_fn=train_input_fn,
    steps=20000,
    hooks=[logging_hook])
```

在调用`numpy_input_fn`这个方法中，我们以字典的形式将训练数据传入到`x`中，将训练结果标签传入到`y`中。指定来`batch_size`为`100`(意味着每次将会训练100个样例)。`num_epochs=None`表示这个模型将一直训练下去。`shuffle=True`参数表示会冲刷训练数据。在调用`train`方法中，我们设置来`step=20000`(表示这个模型将会训练20,000次)，然后传入了`logging_hook`给`hooks`，让它会在训练过程中被触发。

#### 评估模型

一旦训练完成，我们希望通过MNIST test数据集来评估我们的模型生成的概率值的准确度。我们调用`evaluate`方法，而评估指标在`cnn_model_fn`中已定义(`eval_metric_ops`)。继续添加下列代码到`main()`方法中:
```python
# Evaluate the model and print results
eval_input_fn = tf.estimator.inputs.numpy_input_fn(
    x={"x": eval_data},
    y=eval_labels,
    num_epochs=1,
    shuffle=False)
eval_results = mnist_classifier.evaluate(input_fn=eval_input_fn)
print(eval_results)
```
我们设置了`num_epochs=1`，表示这次操作只迭代一次所有的数据。同时设置`shuffle=False`表示这次评估将顺行遍历数据。

#### 运行模型

我们编写了创建CNN模型的函数，`Estimator`，已经训练/评估的相关逻辑代码，现在是该看看结果如何了；运行`cnn_mnist.py`。

> 注意：训练CNNs是需要大量计算的，因此十分耗时。运行`cnn_mnist.py`所需的时间取决于你的处理器，如果在CPU上运行可能会超过1个小时。可以通过减少步数(指定**train()**中的**step**参数)来缩减训练时间，当然这会影响到准确度。

在训练过程中，你可以看到类似下列日志输出:
```
INFO:tensorflow:loss = 2.36026, step = 1
INFO:tensorflow:probabilities = [[ 0.07722801  0.08618255  0.09256398, ...]]
...
INFO:tensorflow:loss = 2.13119, step = 101
INFO:tensorflow:global_step/sec: 5.44132
...
INFO:tensorflow:Loss for final step: 0.553216.

INFO:tensorflow:Restored model from /tmp/mnist_convnet_model
INFO:tensorflow:Eval steps [0,inf) for training step 20000.
INFO:tensorflow:Input iterator is exhausted.
INFO:tensorflow:Saving evaluation summary for step 20000: accuracy = 0.9733, loss = 0.0902271
{'loss': 0.090227105, 'global_step': 20000, 'accuracy': 0.97329998}
```

在这里，我们的测试数据集已经达到了97.3％的准确率。

----

### 更多资源

如果想要更多的了解Tensorflow中的Estimators及CNNs，可以阅读以下资源：

* [Creating Estimators in tf.estimator](https://www.tensorflow.org/get_started/custom_estimators)提供一个关于TensorFlow Estimator API的入门介绍。里边包含了配置一个Estimator,编写生成模型函数，计算损失率，及定义训练操作。
* [Convolutional Neural Networks](https://www.tensorflow.org/tutorials/deep_cnn)如何使用Tensorflow*低层次*API而不是Estimator来构建一个MNIST CNN分类模型。