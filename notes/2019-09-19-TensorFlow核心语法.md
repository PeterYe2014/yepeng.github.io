---
## tf.keras

该API时用于构建和训练深度学习模型的API，tf.keras完全兼容Keras代码。
``` python
import tensorflow as tf
from tensorflow.keras import layers
```

### Sequential模型

tf.keras最简单建立模型的方法计算一层一层地堆叠模型，采用
tf.keras.Sequential 模型，构建模型如下：

``` python
## 构建模型
model = tf.keras.Sequential()
# 添加具有64个神经元地全连接层，激活函数为relu
model.add(layers.Dense(64, activation='relu'))
# 再添加一层
model.add(layers.Dense(64, activation='relu'))
# 十个输出地层
model.add(layers.Dense(10, activation='softmax'))
```
除了全连接层外，还有很多连接层，这些层都有很多相同地参数，

activation 层的激活函数

kernel_initializer/ bias_initializer 
创建层权重和偏差的初始化方案

kernel_regularizer / bias_regularizer
应用层权重和偏差的正则化方案

模型创建后，我们会对模型进行编译和训练，主要调用compile方法和fit方法完成

```python
model.compile(optimizer=tf.train.AdamOptimizer(0.001),              loss='categorical_crossentropy',              
        metrics=['accuracy'])
```
其中tf.model.compile方法有几个参数：

1. optimizer 优化器
2. 误差函数
3. 用于监控训练的参数

```
import numpy as np
data = np.random.random((1000, 32))
labels = np.random.random((1000, 10))
model.fit(data, labels, epochs=10, batch_size=32)
```
通过导入训练的数据，对于监督学习数据分为两个部分，一个是特征数据，一个是标签数据，通过model.fit函数来拟合训练数据

tf.keras.model.fit 有三个参数：

1. epochs: 以周期为单位来进行训练（分小数据批量迭代）
2. batch_size: 每次迭代的数据量
3. validation_data：
传递此参数（输入和标签元组）可以让该模型在每个周期结束时以推理模式显示所传递数据的损失和指标。

训练集数据被拟合后，模型已经训练出来了，我们需要在测试集进行评价和预测,通过evaluate 和 predict方法进行。
```
// 返回损失和准确度
model.evaluate(data, labels, batch_size=32)

// 预测数据的类别可能性数组
model.predict(data, batch_size=32) 

```

### tensorflow_hub 

通过tensorflow我们能够使用已经训练好的模型，减少我们的训练工作。


### 文件、数据以及工具

### tf.keras.utils.get_file
```python
# 下载一个文件，并且返回路径
tf.keras.utils.get_file(fname,  origin, ...)
""" 
参数：
    fname: 保存的文件名或者存储的绝对路径
    origin: 文件的url
 
 返回：
    下载文件存放的路径
"""

```


### 常用的Keras Layers


#### tf.keras.layers.Flatten层
将多维的数据变为1维的数据，常常用于在输入层处理图片数据
```python
model = keras.Sequential([
    keras.layers.Flatten(input_shape=(28,28))
])
# model.output_shape ==（None, 784）
```