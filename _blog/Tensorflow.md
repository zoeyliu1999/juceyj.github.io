---
title: 'Tensorflow Notes'
date: 2019-02-01 16:31:13
layout: archive
author_profile: true
permalink: /blog/tensorflow/
---

# Tensorflow

[Tensorflow](https://www.tensorflow.org/tutorials/) 已经毋庸置疑地成为了目前最火热的深度学习framework，相信其中也有不少Google的因素在里面。通过*CS231n*的Assignment我也正式入坑了TF，这篇博文就将专门用来记录我在使用tensorflow的过程当中遇到了一个Bug和比较厉害的API，因此这篇文章也将长期更新！

```python
import tensorflow as tf
import tensorflow.contrib.eager as tfe
```

# 0. Pipeline

- 根据需要定义session
- 定义计算图
- 每个variable都会定义自己的initializer，可能是随机初始化，也有可能是常量初始化
- 在run computational graph之前要先run initializer，可以是global的，也可以是指定var_list（根据var_name来判断）
- 然后再运行计算图

# 1. Tensorboard

Tensorboard是Tensorflow自带的一个可视化工具，你需要在Python中添加指定的指令就可以帮助你可视化计算图以及某些变量（e.g. loss & accuracy）的变化情况。所有的信息存储在一个`event`文件当中。



## Open Tensorboard

打开tensorboard的方式和jupyter notebook的方式差不多，你需要进入`event`文件的上上层目录，即如果`event`文件存储在**"/tensorboard/graphs/xx. event"**目录下，你就要先进入**tensorboard**，然后运行指令：

```python
tensorboard --logdir="./graphs"
```

即可。实际操作的时候可能会经常出现Bug，我至今没有搞清楚是为什么，因为经常相同的文件相同的指令，我多次打开tensorboard有的时候找得到event文件有的时候找不到，可能是因为TF的版本太低了吧 = = Anyway



## Summary

1. 记录计算图，在你的所有的计算图定义结束了之后加入指令：

   ```python
   writer = tf.summary.FileWriter('./graphs', tf.get_default_graph())
   ```

   即可。当然你可以更改后面的参数记录其他子图的定义。

2. 记录某个变量的训练过程有scalar、histogram等多种方式

   ```python
   tf.summary.scalar("loss", loss)
   tf.summary.histogram("histogram loss", accuracy)
   summary_op = tf.summary.merge_all()
   
   With tf.Session() as sess:
   	for index in range(100):
           summary = sess.run([summary_op])
           writer.add_summary(summary, global_step=index)
   ```

   在这里值得说明的是，有多个summary操作的话需要将summary操作merge之后添加到writer当中，否则是不会成功显示的。
   另外就是在这里的`global_step`这个参数很重要，是全局索引，在optimizer中也会用到。


# 2. tf.train.Saver()
参数存储。需要和checkpoint文件一起存储作为参数的map。当然你也可以指定需要存储的特定参数。

```python
# 存储过程
saver = tf.train.Saver() 
saver.save(sess, store path, global_step)

# 先restore checkpoint，里面有变量名的索引，然后再restore值文件
ckpt = tf.train.get_checkpoint_state(os.path.dirname('path')) 
saver.restore(sess, path)
```



# tf.cond(pred, fn1, fn2, name=None)

没有eager mode的情况下不能使用python if / for/ while，因为这时候tensor里面没有实际的值，这时候需要使用特定的分支API，其中fn1和fn2是函数，看下面的例子：

```python
def huber_loss(labels, predictions, delta=14.0): 
 	residual = tf.abs(labels - predictions) 
  	def f1(): return 0.5 * tf.square(residual) 
  	def f2(): return delta * residual - 0.5 * tf.square(delta) 
  	return tf.cond(residual < delta, f1, f2)
```



# 3. tf.data
## tf.data.Dataset() 

实验证明，使用这种方式比使用placeholder在速度上会有显著的提升，之后使用Iterator来访问，当然tf.data.Dataset()也可以设置`batch_size`。生成API的方法：

```python
# 注意这是实际上只有一个Input
dataset = tf.data.Dataset.from_tensor_slices((features, labels)) # 参数实际上是一个二元组
dataset = dataset.shuffle(10000) # if you want to shuffle your data
dataset = dataset.batch(batch_size)
```



## tf.Iterator()

```python
# 1. Iterates through the dataset exactly once. No need to initialization. 
iterator = dataset.make_one_shot_iterator() 
X, Y = iterator.get_next() # 获取一组数据的方法

# 2. Iterates through the dataset as many times as we want. Need to initialize with each epoch. 
iterator = dataset.make_initializable_iterator() 
sess.run(iterator.initializer)

# 3. 一个普适class
iterator = tf.data.Iterator.from_structure(train_data.output_types, train_data.output_shapes) 
# 为了应对训练集和测试集的情况，不指定某个特定dataset的迭代器，而是指定了一种结构，具体dataset的指向在初始化的时候完成，就是相同的结构给训练集和测试集同样分配一个迭代器 
train_init = iterator.make_initializer(train_data) # initializer for train_data 
test_init = iterator.make_initializer(test_data) # initializer for train_data
```



# 4. Eager Model

真的是，在我无比适应计算图之后告诉我eager model才是未来的趋势真的是令人心碎，Eager model使得tensor像numpy array一样易操作，支持各种分支和索引。但是缺点就是一旦开启无法关闭，这个还是很不走心的，尤其是在Eager model和底层TF API兼容性那么差的情况下。不过在这里很高兴的是：**Keras支持Eager model！**
- `tfe.Variable`
- `tfe.gradient_function(f)` 求给定函数f的输出对于输入的梯度
- `tfe.implicit_gradients(f)` 求给定函数f对于在计算过程中涉及到的tfe.Variable的梯度
- `tf.train.AdamOptimizer().apply_gradients(grad)` Eager model给我们平常使用的`tf.train.AdamOptimizer().minimize(loss)`也带来了影响，因为minimize的loss参数必须是一个函数，是一个tensorflow op，但是在eager model下我们计算得到的loss往往就是一个和numpy可以看成相同的tensor，所以原来的API无法继续使用，改为使用以grad为参数的新的API，值得注意的是apply_gradient就是minimize的第二步操作



# 5. API details

- Name Scope  VS Variable Scope
  - with tf.name_scope('two_layers') as scope:
    一方面是为了简化计算图，另一方面使得不同name scope中可以使用相同名字**（这里的名字Python Varaiable的命名）**。
    E.g. **GANs**中我们就可以在两个网络下使用相同的参数名字。
  - with tf.variable_scope('two_layers') as scope:
    Variable Scope的使用促进了参数的share。看下面的例子即可。注意reuse有两个前提：

  1. 重复使用的参数具有相同的名字，这里的名字指的是**计算图中的name**
  2. scope.reuse_variables()

```python
with tf.variable_scope('two_layers') as scope:
    logits1 = two_hidden_layers(x1)
    scope.reuse_variables()
    logits2 = two_hidden_layers(x2)
```

- Tensorflow Padding
  - Valid = no padding：使用valid的时候实际上就是没有padding，卷积核从头开始slide，如果最后一侧滑动没有到最后一个元素，但是已经不够了，就舍弃剩下的。
  - SAME = padding：same情况下如果stride=1的时候输入输出的长宽相同。但是值得注意的是tensorflow的padding和Caffe不同，tensorflow优先选择在activation map的右边填充0，而caffe优先在左边padding

- tf查看ckpt文件内部信息

```python
from tensorflow.python import pywrap_tensorflow
checkpoint_path = 'model.ckpt'
reader = pywrap_tensorflow.NewCheckpointReader(checkpoint_path) #tf.train.NewCheckpointReader
var_to_shape_map = reader.get_variable_to_shape_map()
for key in var_to_shape_map:
    print("tensor_name: ", key)
    #print(reader.get_tensor(key))
```

- tf.train.saver(max_to_keep=5)

  可以控制只保存5个模型即使你的epoch数量很大，如果要保存比如精度最高的几个模型就要自己手写判断

- tf不再直接占领整张卡

```python
gpu_options = tf.GPUOptions(allow_growth=True)
sess = tf.Session(config=tf.ConfigProto(gpu_options=gpu_options))
```

- 读取tensorboadrd event

```python
from tensorboard.backend.event_processing import event_accumulator
ea = event_accumulator.EventAccumulator(event_file)
ea.Reload()
print(ea.scalars.Keys())
curve = ea.scalars.Items('Train_AverageReturn')
step = [each.step for each in curve]
value = [each.value for each in curve]
```

