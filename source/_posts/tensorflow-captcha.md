---
title: 使用Tensorflow识别图片验证码
date: 2018-10-16 18:46:18
tags: ["TensorFlow", "CNN"]
categories: ["Python","TensorFlow"]
---

验证码是我们在网络浏览中经常能见到的元素之一。它主要是用来区分用户是计算机还是人的公共全自动程序。之前使用jmeter来测试登录页面的功能时就被这个验证码给挡住了，最终手动输入了事。由于这个验证码是比较简单的，由字母与数字组成，之前看过TensorFlow的一篇教程，是用卷积神经网络(CNN)来识别手写数字的，后来想想，不如使用TensroFlow来搭建一个卷积神经网络模型来识别一下验证码。

<!-- more -->

按照TensorFlow官网的说法，大多数tf程序的结构都似如下：
1. 导入和解析数据集。
2. 选择模型类型。
3. 训练模型。
4. 评估模型的效果。
5. 使用经过训练的模型进行预测。

不过在此之前，先需要导入相关的依赖:
```python
import tensorflow as tf
tf.enable_eager_execution()

import random
import shutil
import os
import math

from captcha.image import ImageCaptcha
```
在这里开启了Eager Execution。这个特性要求TensorFlow 1.8以上，因此如果tf版本不够，需要升级一下才行。
最后一个则是用来生成验证码的第三方包captcha。如果没有需要通过pip来安装
```
pip install captcha
```
----

### 导入和解析数据集

数据对于机器学习来说十分重要，好的数据集对机器学习的帮助十分大。这里若要成功识别出图片中的验证码则需要大量的图片数据做训练。

#### 创建数据集
首先需要使用captcha包来生成验证码:
```python
folder_path = 'images'
numbers = [str(i) for i in range(10)]
alphabet = [chr(i) for i in range(ord('a'), ord('z')+1)]

captcha_set = numbers + alphabet
captcha_size = len(captcha_set)
captcha_length = 6

def generate_image_code(path=folder_path, char_list=captcha_set, size=captcha_length):
  """生成图片验证码"""
  text = [random.choice(char_list) for _ in range(size)]
  code = ''.join(text)
  imageCaptcha = ImageCaptcha()
  filepath = os.path.join(path, code + '.jpg')
  imageCaptcha.write(code, filepath)
  return filepath
```
我们这里生成的验证码是数字+小写字母形式的验证码，一般的验证码都是4或者6位，这里使用6位的。通过随机取出字母数字然后通过captcha生成对应的验证码图片并且写入到指定文件夹中。文件名则是正确的验证码。
接下来通过这个方法生成一堆验证码:
```python
if not tf.gfile.Exists(folder_path):
  tf.gfile.MkDir(folder_path)
  for _ in range(20000):
    generate_image_code(path, captcha_set, captcha_length)
```
这里生成了20000组验证码图片，对比起mnist的那数据规模来说算是小的了。这里的数据规模会影响到最终生成模型的准确率。一般来说越多越随机是越好，不过数据变多就会导致训练的过程变的更加缓慢。

#### 拆分数据集
在生成了一堆所需要的图片数据之后，我们接下来需要做的就是将它们拆分成**训练集**与**测试集**两个子集。训练集是用于训练模型的子集，而测试集则是用于测试训练后模型的子集。
```python
def generate_features_and_labels(path=folder_path):
  """生成训练/测试特征标签"""
  image_data = [os.path.join(root, f) for root, _, files in os.walk(path) for f in files]

  # 打乱数据
  random.shuffle(image_data)
  split_num = math.floor(len(image_data)*0.8)
  train_data = image_data[:split_num]
  train_labels = [list(os.path.splitext(os.path.split(x)[1])[0]) for x in train_data]
  train_labels = [list(map(lambda x:captcha_set.index(x), label)) for label in train_labels]
  test_data = image_data[split_num+1:]
  test_labels = [os.path.splitext(os.path.split(x)[1])[0] for x in test_data]
  test_labels = [list(map(lambda x:captcha_set.index(x), label)) for label in test_labels]
  return (train_data, train_labels), (test_data, test_labels)
```
我们所有的图片都放在一个统一的文件夹下，因此我们需要用`os.walk()`方法来获取它们。`os.walk()`方法可迭代有三个变量:`root`表示根目录，这里为images；`dirs`表示根目录下的子目录，这里没有，因此可以缺省；`files`则是表示每个子目录下的文件名列表。我们这里需要将文件名与根目录给拼接起来组成一个相对路径，让后续的输入函数能够获取到对应的文件。这里需要注意一下的是Python的这种语法如果是子列表应该要写在父列表的后边。

获取到所有的文件相对路径列表之后我们先要将它们随机排列一下。因为上边获取到的列表是默认按文件名排序的，如果数据分布不随机后续我们需要拆分数据的话就会导致后边测试集中很大一部分是训练集中没有的(例如训练集中包含的是0-x开头的验证码，而测试集中包含了大多数y-z开头的验证码)。

接下来按80%的比例将原有数据集拆分为训练集及测试集。接下来的的步骤都是从文件名中提取正确的验证码，然后将其转换成对应的数组下标。
最后将得到对应的训练/测试数据集：
```python
(train_features, train_labels), (test_features, test_labels) = generate_features_and_labels()
```

#### 解析数据集

我们希望得到的训练/测试数据集都包含了**特征**及**标签**。特征类似于题目，记录样本的特点；标签类似于答案，记录着特征对应的需要预测的值。
上边我们获得的训练/测试数据集中的数据还比较原始，并不能被tf所理解运行。因此，我们需要通过解析函数来将对它们做一次转换工作:
```python
image_height = 60
image_width = 160

def parse_image(feature, label=None):
  image_string = tf.read_file(feature)
  # 解析图片
  image_decoded = tf.image.decode_jpeg(image_string)
  # 将图片大小转换成对应的大小
  image_resized = tf.image.resize_images(image_decoded, [image_height, image_width])
  # 将图片由彩色转换成黑白
  image_gray = tf.image.rgb_to_grayscale(image_resized)
  # 将图片的值由(0, 255)压缩到(0, 1)
  features = tf.divide(image_gray, 255)
  if label is not None:
    # 将标签转换成独热编码
    labels = tf.one_hot(label, depth=captcha_size)
    return features, labels
  else:
    return features
```
这里将通过图片文件的相对路径获取到图片信息，然后转换成对应的张量(tensors)。由于之前生成的图片都是60*160的，因此这里将按照这样的大小来定义张量的形状(shape)。
如果传入了标签，还需要对标签数据转换成独热编码(one-hot encoding)。独热编码是一种稀疏向量，其中一个元素设为1，其他元素均为0。例如一组向量为`[1,3,5]`,转换成独热编码的形式则为
```
[1,3,5] => [[0, 1, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 1, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 1, 0, 0, 0, 0]]
```
独热编码常用于表示拥有有限个可能值的字符串或标识符。我们的标签值为验证码对应着在`captcha_set`中的数组下标，可以转换成`captcha_length x captcha_size`(6 x 36)大小的独热编码。

#### 生成输入函数

输入函数可以将输入数据转换成`tf.data.Dataset`。此乃最终可用于训练的数据格式。
```python
def input_fn(features, labels=None, num_epochs=None, shuffle=True, batch_size=1):
  if labels is None:
    inputs = features
  else:
    inputs = (features, labels)
  dataset = tf.data.Dataset.from_tensor_slices(inputs)
  # 随机打乱数据
  if shuffle:
    dataset = dataset.shuffle(buffer_size=len(features))
  # 使用5个进程来格式化数据
  dataset = dataset.map(parse_image, num_parallel_calls=5)
  # 重复读取数据
  dataset = dataset.repeat(num_epochs)
  # 取样本数量
  dataset = dataset.batch(batch_size)
  return dataset
```
`tf.data.Dataset`通过`from_tensor_slices`方法加载了输入数据，然后通过调用上边的解析函数将原始数据转换成对应的张量。`shuffle`用于将数据随机化。`num_epochs`参数用于将数据集重复抓取，如果为`None`则会无限抓取。`batch_size`表示一个批次中的样本数，是一个超参数。

通过了以上几个步骤，我们的数据集就准备好了。接下来就应该是搭建模型了。

----

### 搭建模型

在这里我们使用**Keras**来搭建神经网络模型。`tf.keras`是一个高级的API，它具有使用友好、模块化及可组合、易于扩展等优点。`tf.keras`用于构建模型有两种方式，一种是`functional API`,一种是`subclassing`。这里使用的是后一种：
```python
class CnnModel(tf.keras.Model):
  def __init__(self):
    super(CnnModel, self).__init__()
    self.conv1 = tf.keras.layers.Conv2D(32, [5,5], padding='same', activation=tf.nn.relu, 
      input_shape=(image_height, image_width, 1))
    self.pool1 = tf.keras.layers.MaxPooling2D([2,2], 2)
    self.conv2 = tf.keras.layers.Conv2D(64, [5,5], padding='same', activation=tf.nn.relu)
    self.pool2 = tf.keras.layers.MaxPooling2D([3,2], [3,2])
    self.conv3 = tf.keras.layers.Conv2D(128, [5,5], padding='same', activation=tf.nn.relu)
    self.pool3 = tf.keras.layers.MaxPooling2D([2,2], 2)
    self.pool_flat = tf.keras.layers.Flatten()
    self.dense1 = tf.keras.layers.Dense(2048, activation="relu")
    self.dropout = tf.keras.layers.Dropout(0.4)
    self.dense2 = tf.keras.layers.Dense(captcha_length * captcha_size)
    self.logit = tf.keras.layers.Reshape((captcha_length, captcha_size))
  
  def call(self, x, training=True):
    x = self.conv1(x)       # 60 x 160 x 32
    x = self.pool1(x)       # 30 x 80 x 32
    x = self.conv2(x)       # 30 x 80 x 64
    x = self.pool2(x)       # 10 x 40 x 64
    x = self.conv3(x)       # 10 x 40 x 128
    x = self.pool3(x)       # 5 x 20 x 128
    x = self.pool_flat(x)   # 12800
    x = self.dense1(x)      # 2048
    if training:
      x = self.dropout(x, training=training)
    x = self.dense2(x)
    x = self.logit(x)
    return x
```
这里建立了一个类`CnnModel`继承于`tf.keras.Model`,按照要求在`__init__`方法中定义了所有的神经元层。`call`方法则是将它们组合成了一个神经网络模型。

这个模型包含了3个卷积-池化层，通过这三个组合层之后将特征不断的提取出来然后经过一个`tf.keras.layers.Flatten`展平之后传入到第一个密集层，如果是训练模式则加入了一个`dropout`层以防止[*过拟合*](https://developers.google.com/machine-learning/crash-course/glossary#overfitting)。接下来传入到第二个密集层，最终经过`Reshape`变成了输出层。

----

### 训练模型

数据与模型都准备好了，我们就应该开始训练模型了。通过训练模型，模型将会在数据特征中找出规律，从而更加了解数据，在不断的优化过程后，模型就能根据数据预测出标签了。

首先需要定义一个梯度函数：
```
def grad(model, inputs, targets):
  with tf.GradientTape() as tape:
    logits = model(inputs, training=True)
    loss_value = tf.losses.sigmoid_cross_entropy(multi_class_labels=targets, logits=logits)
  return tape.gradient(loss_value, model.variables), loss_value, logits
```
梯度函数通过损失(`loss`)与[`tf.GradientTape`](https://www.tensorflow.org/api_docs/python/tf/GradientTape)来记录计算[*梯度*](https://developers.google.com/machine-learning/crash-course/glossary#gradient)(用于优化模型)的操作。
[*损失*](https://developers.google.com/machine-learning/crash-course/glossary#loss)一种衡量指标，用于衡量模型的预测偏离其标签的程度。或者更悲观地说是衡量模型有多差。在这里使用了[`tf.losses.sigmoid_cross_entropy`](https://www.tensorflow.org/api_docs/python/tf/losses/sigmoid_cross_entropy)来计算模型输出的值与真实标签之间的损失率。它常用于计算每个类是**独立但不互斥**的分类任务中的概率误差。由于验证码中的字母数字是可能有重复的，因此它在这里比较适合用作计算损失率。

> 之前官方教程用于识别mnist手写数字中使用的损失函数是`softmax_cross_entropy`，它是计算**独立且互斥**的分类任务的。

接下来则是开始循环训练模型:
```python
model = CnnModel()
train_dataset = input_fn(train_features, train_labels, batch_size=32, num_epochs=1)
optimizer = tf.train.AdamOptimizer(learning_rate=0.001)
epochs = 51

for epoch in range(epochs):
  train_loss_avg = tf.contrib.eager.metrics.Mean()
  train_accuracy = tf.contrib.eager.metrics.Accuracy()
  for features, labels in train_dataset:
    grads, loss, logits = grad(model, features, labels)
    optimizer.apply_gradients(zip(grads, model.variables),
      global_step=tf.train.get_or_create_global_step())
    train_loss_avg(loss)
    train_accuracy(tf.argmax(input=labels, axis=2), tf.argmax(input=logits, axis=2))
  if epoch % 10 == 0:
    print("Epoch {:03d}: Loss: {:.3f}, Accuracy: {:.3%}"
      .format(epoch, train_loss_avg.result(), train_accuracy.result()))
```
训练数据集通过上边的输入函数生成，这里指定[*批次大小*](https://developers.google.com/machine-learning/glossary/#batch_size)为32；然后指定优化器为[`tf.train.AdamOptimizer`](https://www.tensorflow.org/api_docs/python/tf/train/AdamOptimizer),指定的[*学习速率*](https://developers.google.com/machine-learning/crash-course/glossary#learning_rate)为0.001；epochs为循环次数。它们三个都是[超参数](https://developers.google.com/machine-learning/crash-course/glossary#hyperparameter)，通过不同的组合可以让模型优化到最好。

在循环训练中，程序会从训练集中遍历每个样本，提取出样本的特征及标签；然后通过梯度函数得到预测结果与损失率及梯度；接着优化器会根据梯度指标更新模型的变量。中途程序将记录模型的损失率及准确率，以便查看。
```
Epoch 000: Loss: 0.136, Accuracy: 2.870%
Epoch 010: Loss: 0.011, Accuracy: 97.896%
Epoch 020: Loss: 0.005, Accuracy: 99.329%
Epoch 030: Loss: 0.004, Accuracy: 99.643%
Epoch 040: Loss: 0.003, Accuracy: 99.759%
Epoch 050: Loss: 0.002, Accuracy: 99.825%
```
可以看到一开始模型对这些数据束手无策，全靠瞎猜，准确率低的可怜。经过不断的循环训练之后，准确率将变的越来越高，损失率也变的越来越低。这一步耗时十分长，而且十分吃CPU，因此有个好的CPU或者GPU那花费的时间将大大减少。

----

### 评估模型

正如人学习完了课程需要用考试来检验一下，模型也要在训练完毕之后进行一场“考试”，也就是评估。
```python
test_dataset = input_fn(test_features, test_labels, num_epochs=1, shuffle=False)
test_accuracy = tf.contrib.eager.metrics.Accuracy()
for features, labels in test_dataset:
  logits = model(features, training=False)
  logits = tf.argmax(input=logits, axis=2)
  labels = tf.argmax(input=labels, axis=2)
  test_accuracy(labels, logits)
print("Test Accuracy: {:.2%}".format(test_accuracy.result()))
```
我们通过输入函数生成了测试数据集，然后迭代测试数据集，提取样本中的特征让模型进行预测，然后通过与样本标签之前的对比而得出测试集的准确率。
在这里需要注意的是模型进行评估时就不需要进行过拟合操作了，因此需要设置`training=False`。我们的标签及模型输出值都是独热编码，因此需要通过`tf.argmax`来得到他们的数组下标，类似独热编码的反向操作。
```
Test Accuracy: 96.80%
```
准确率还不算低，只能说CNN不愧是对目前对图像识别最先进的模型。如果对模型感到满意，可以调用`model.save_weights()`方法来保存模型，这样下次可以通过`model.load_weights()`方法来载入训练完毕的模型。

> 由于这里使用的是`subclassing`的方式，因此`tf.keras.models.Model`的某些内置方法如`model.save()`、`model.summary()`是不能使用的。

----

### 使用模型预测数据

最后应该让模型派上用场了，让它对一些图片验证码进行预测一下。预测的图片验证码可以通过上边的方法生成，也可以直接从测试数据集中随机提取，只要能通过输入函数生成数据集就行:
```python
# 随机生成10份预测数据
random_index = [random.randrange(0, len(test_features)) for _ in range(10)]
predict_features = [test_features[i] for i in random_index]
predict_labels = [test_labels[i] for i in random_index]

predict_dataset = input_fn(predict_features, num_epochs=1, shuffle=False)
if predict_labels is not None:
  for feature, label in zip(predict_dataset, predict_labels):
    logits = model(feature, training=False)
    predict = tf.reshape(tf.argmax(input=logits, axis=2), [-1]).numpy().tolist()
    predict = "".join([captcha_set[i] for i in predict])
    result = "".join([captcha_set[i] for i in label])
    print("Predictive value:", predict, "Actual value:", result)
  else:
  for feature in predict_dataset:
    logits = model(feature, training=False)
    predict = tf.reshape(tf.argmax(input=logits, axis=2), [-1]).numpy().tolist()
    predict = "".join([captcha_set[i] for i in predict])
    print("Predictive value: ",predict)
```
由于预测的数据可能是无标签的，因此这里就无须将标签传入到输入函数中了。通过模型我们可以得到对应的数据，然后通过`tf.argmax`将独热编码转换成张量，然后再通过`tf.reshape`方法将张量展平成标量，最后转换成数组。大概过程如下:
```
[[0, 1, 0, 0, 0, 0, 0, 0, 0, 0], tf.argmax => [[1,3,5]] reshape => [1,3,5]
 [0, 0, 0, 1, 0, 0, 0, 0, 0, 0],
 [0, 0, 0, 0, 0, 1, 0, 0, 0, 0]]
```
得到的自然是图片中的验证码对应着在`captcha_set`中的数组下标，然后通过在`captcha_set`集合中查找，最终得到了对应的验证码。
```
Predictive value: 2gz6yt Actual value: 2gz6yt
Predictive value: 2mza8c Actual value: 2mza8c
Predictive value: 3elyvi Actual value: 3eiyvi
Predictive value: 3krvqx Actual value: 3krvqx
Predictive value: 9gsbvr Actual value: 9gsbvr
Predictive value: eedyaz Actual value: eedyaz
Predictive value: pe3rpe Actual value: pe3rpe
Predictive value: pyfs6p Actual value: pyfs6p
Predictive value: q0wvnx Actual value: q0wvnx
Predictive value: xl29u6 Actual value: xl29u6
```
这样看来算是比较出色的完成了识别任务了。感觉现在简单的验证码也拦不住机器了。
最后可以使用`shutil`里的方法删除掉生成的图片做一些收尾工作:
```python
shutil.rmtree(folder_path, ignore_errors=True)
```