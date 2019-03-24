---
title: 创建与读取TFRecords文件
date: 2019-03-20 21:05:30
tags: TensorFlow
---

[参考链接](https://my.oschina.net/u/3800567/blog/1637798).用TensorFlow，将本地图片文件，制作为TFRecords文件的数据集，并读取。

<!-- more -->

# 遍历目录及文件

这个步骤，主要是把所有图片的路径读入三个列表中：`training_images`、`testing_images`、`validation_images`.对应三个集合：训练集、测试集、验证集。流程如下：

1. 确定所有子目录（`tf.gfile.Walk()`）
2. 找出所有图片文件（`tf.gfile.Glob()`）
3. 对每一个图片文件判断进入哪一个集合

## 确定所有子目录

在TensorFlow中有函数`tf.gfile.Walk()`函数可以用来遍历目录下的子目录以及文件。在此记录一下该函数的遍历次序问题，即每次遍历输出的结果是否一致。发现该函数和`os.walk()`函数的遍历方式是一样的，且每次遍历结果都相同。

每次调用遍历一个目录，返回的结果为一个三元组`(root, dirs, files)`：

- `root`：当前所遍历的目录
- `dirs`：所遍历目录下，所有的目录名
- `files`：所遍历目录下，所有的文件名

如下面这个文件树：

{% asset_img 1.jpg %}

默认会按照先根目录，后子目录的顺序遍历。如下：

```shell
>>> [x for x in os.walk('pycode')]
[('pycode', ['tmp', 'tutorial'], ['algo.py', 'log.txt', 'paper.py', 'StringConta
in.py', 'tf_test.py']), ('pycode\\tmp', [], ['new 1.py']), ('pycode\\tutorial',
[], ['fabs.py', 'reverse_string.py', 'run.py'])]
```

```shell
>>> [x for x in tf.gfile.Walk('pycode')]
[('pycode', ['tmp', 'tutorial'], ['algo.py', 'log.txt', 'paper.py', 'StringConta
in.py', 'tf_test.py']), ('pycode\\tmp', [], ['new 1.py']), ('pycode\\tutorial',
[], ['fabs.py', 'reverse_string.py', 'run.py'])]
```

当对应参数置为`False`时，将会按照先子目录，后根目录的顺序遍历。如下：

```shell
>>> [x for x in os.walk('pycode', topdown=False)]
[('pycode\\tmp', [], ['new 1.py']), ('pycode\\tutorial', [], ['fabs.py', 'revers
e_string.py', 'run.py']), ('pycode', ['tmp', 'tutorial'], ['algo.py', 'log.txt',
 'paper.py', 'StringContain.py', 'tf_test.py'])]
```

```shell
>>> [x for x in tf.gfile.Walk('pycode', in_order=False)]
[('pycode\\tmp', [], ['new 1.py']), ('pycode\\tutorial', [], ['fabs.py', 'revers
e_string.py', 'run.py']), ('pycode', ['tmp', 'tutorial'], ['algo.py', 'log.txt',
 'paper.py', 'StringContain.py', 'tf_test.py'])]
```

## 找出所有图片文件

上一步所找出的所有子目录，即为类别目录，类别目录下即为该类别的所有图片样本。通过一个文件类型匹配，可以避免目录中出现非图片文件而导致的异常。例如要求所有图片类型为`.jpg`：

```python
file_glob = './images/A/*.jpg'
image_list = tf.gfile.Glob(file_glob)
```

这一匹配函数`tf.gfile.Glob()`，给出的文件名称次序是固定的，每次调用都不变。这一顺序和`os.listdir()`给出的次序是一样的。对于类似`A001.jpg, A002.jpg, ...`这种命名方式，其次序并不按照数字的大小。如下：

```shell
>>> file_glob = '*.png'
>>> tf.gfile.Glob(file_glob)
['./A3.png', './A11.png', './A10.png', './A8.png', './A6.png', './A9.png', './A12.png', './A5.png', './A16.png', './A7.png', './A15.png', './A2.png', './A1.png', './A13.png', './A14.png', './A4.png', './A0.png']
```

```shell
>>> os.listdir('.')
['A3.png', 'A11.png', 'A10.png', 'A8.png', 'A6.png', 'A9.png', 'A12.png', 'A5.png', 'A16.png', 'A7.png', 'A15.png', 'A2.png', 'A1.png', 'A13.png', 'A14.png', 'A4.png', 'A0.png']
```

这一排序方式可能是由文件系统（file system）决定的。

## 将图片装入不同的集合

在获取了每一类的`image_list`之后。接下来就要对每一类样本，都按训练集、测试集和验证集这三个集合分开。之后再将每一类的这三个集合组合起来，形成最终的训练集、测试集和验证集。对每一个样本文件，需要一个函数，来决定这个样本将进入哪个集合。这里采用了网上的一种方式（目前还不知道原作者是谁，网络上很多源代码都是这么干的）：

```python
training_images = []
testing_images = []
validation_images = []
        
for file_name in file_list: 
	# e.g.
    # file_name: './images/0/0#1.jpg'
    # file_list: ['./images/0/0#1.jpg', './images/0/0#2.jpg', ...]
            
    hash_name = re.sub(r'_nohash_.*$', '', file_name)
    hash_name_hashed = hashlib.sha1(tf.compat.as_bytes(hash_name)).hexdigest()
            
    percentage_hash = ((int(hash_name_hashed, 16) % (MAX_NUM_IMAGES_PER_CLASS + 1)) * (100.0 / MAX_NUM_IMAGES_PER_CLASS))
            
    if percentage_hash < validation_percentage:
		validation_images.append(file_name)
	elif percentage_hash < (testing_percentage + validation_percentage):
		testing_images.append(file_name)
	else:
        training_images.append(file_name)        
```

里面通过哈希编码，只根据样本的文件名（含路径），来确定该文件是进入哪个集合。而且在文件名不变的情况下，同一个样本在每次操作都将进入同一个集合，不论是否加入了新的样本。因为哈希编码的结果只和文件名（含路径）有关。在对每一类样本都分开三个集合之后，下面就要将不同类之间的小集合组合起来。

```python
def analysis_image_lists(image_lists):
    '''
    Input: 
    - image_lists
    e.g.
    image_lists['A'] = {
            'training': ['./images/A/A001.jpg', ...],
            'testing': ['./images/A/A002.jpg', ...],
            'validation': ['./images/A/A005.jpg', ...],
        }
    
    Output: 
    - train_list
    - test_list
    - validation_list  
    e.g. ['./images/A/A002.jpg', ...], ..., ...
    '''  
    
    train_list = []
    test_list = []
    validation_list = []
    
    for _, lists_dict in image_lists.items():
        
        train_list.extend(lists_dict['training'])
        test_list.extend(lists_dict['testing'])
        validation_list.extend(lists_dict['validation'])
    
    random.shuffle(train_list)
    random.shuffle(test_list)
    random.shuffle(validation_list)
    
    return train_list, test_list, validation_list
```

这里是访问一个字典里的键值，我们知道python中的字典，内部是无序的。但是其实每次访问其中元素时，还是依照了一定的顺序，只不过不见得是放入时的顺序。这种访问次序，是由一个哈希算法决定的，这种哈希算法随着python版本的改变，以及操作平台的改变都会不同。

>Python dictionaries and sets are stored using some efficient hashing algorithm. The precise algorithm is an implementation detail (and may differ from one version of Python to another as well as one platform to another (CPython, vs. PyPy, vs Jython or IronPython, and so on).
>
>Iterating over the keys returns them in whatever order the underlying implementation chose to store them. Hashing provides for extremely efficient access to an item given its key. This typically has the effect of making them appear to be in a “random” order by human standards.
>
>[参考链接](https://www.quora.com/In-Python-what-decides-the-order-that-a-loop-on-a-dictionary-works-through-the-keys).

所以至此来说，制作成的`TFRecords`文件中样本的顺序，在每次制作时就应该认为是不固定的了。那么看来就没必要保持每次制作数据集时样本的次序一致。在上面的函数中，干脆加入了`random.shuffle()`，来让它乱的彻底一点。<span style="border-bottom:2px dashed yellow;">实际上，在从数据集中取样本用来训练模型的时候，也就是要尽量以一种均匀分布的概率来抽取样本。所以没必要在制作数据集的时候保持样本次序的稳定。这也是我一开始的一个误解。</span>以上作为一个例子，目的是说明在每次形成三个数据集的`image_list`的时候，文件的顺序很可能因为文件名的原因而发生改变。而我们现在对图片文件的命名规则为`class_index#image_index.extension`.例如：`0#1.jpg`代表第`0`类中的第`1`张图片。这种命名规则下文件的顺序始终是数字大小顺序，不过前面也说了，不用保持这种次序的稳定性。

至此就制作好了训练集、测试集、验证集三个图片路径列表。

# 读取TFRecords注意事项

这里主要关注函数`tf.data.Dataset.shuffle(buffer_size)`。针对这个函数，做两点说明。一个是关于参数`buffer_size`的取值；一个是该函数与`tf.data.Dataset.batch()`的先后次序问题。

## buffer_size的取值

先来看`tf.data.Dataset.shuffle(buffer_size)`的工作过程：

1. 在所有样本中按次序取前`buffer_size`个样本放入buffer中。

{% asset_img 2.jpg %}

2. 当读取数据时，每次从buffer中随机取出一个样本。

{% asset_img 3.jpg %}

3. 当buffer中取走数据后，会从所有样本中再按次序取下一个样本放入buffer。

{% asset_img 4.jpg %}

这一函数的作用就是避免一次性将整个数据集读入内存，但又能有随机性的取样本。读入内存的为`buffer_size`个样本，且在buffer中取样本为随机的。`buffer_size`和数据集样本总数的相对大小，将会影响取样本的过程对整个数据集而言的随机性。

- buffer_size >= 样本总数。那么此时整个数据集都会被放进buffer里，取样本时，对于整个数据集来说就是完全随机的。每个样本被选到的概率符合均匀分布。

- buffer_size == 1。这相当于依次取样本，并没有随机性。

以上是两种极端情况。那么其实还是没有回答应该如何取一个中间值，既避免因数据集过大而导致占用内存过多，又可以使样本的选取具有一定随机性。这种值的选取应该是要结合具体任务的。总之，要尽量保证一个`batch`的样本中，各类别样本数尽量平均。比如做猫和狗的图片分类，`batchsize = 32`，那么这其中大致应有16张猫的图片和16张狗的图片。[参考链接](https://user-gold-cdn.xitu.io/2018/8/28/16580f396628e48b).

## Dataset.shuffle与Dataset.batch的顺序

除了`tf.data.Dataset.shuffle(buffer_size)`还有一个函数叫`tf.data.Dataset.batch(batch_size)`。这两个都是对数据集的一种操作。结论是`shuffle`应该在`batch`前面。因为如果先执行`tf.data.Dataset.batch()`，会在原数据集中，依次将每`batch_size`个样本作为一个整体输出。那么如果在此之前数据集并没有很好的打乱的话，按顺序把样本组合起来的话，可能会有`batch`中 样本类别分布不均匀的情况。以[参考链接](https://stackoverflow.com/questions/50437234/tensorflow-dataset-shuffle-then-batch-or-batch-then-shuffle)中的例子为例：

```python
tf.enable_eager_execution()  # To simplify the example code.

# Batch before shuffle.
dataset = tf.data.Dataset.from_tensor_slices([0, 0, 0, 1, 1, 1, 2, 2, 2])
dataset = dataset.batch(3)
dataset = dataset.shuffle(9)

for elem in dataset:
  print(elem)

# Prints:
# tf.Tensor([1 1 1], shape=(3,), dtype=int32)
# tf.Tensor([2 2 2], shape=(3,), dtype=int32)
# tf.Tensor([0 0 0], shape=(3,), dtype=int32)

# Shuffle before batch.
dataset = tf.data.Dataset.from_tensor_slices([0, 0, 0, 1, 1, 1, 2, 2, 2])
dataset = dataset.shuffle(9)
dataset = dataset.batch(3)

for elem in dataset:
  print(elem)

# Prints:
# tf.Tensor([2 0 2], shape=(3,), dtype=int32)
# tf.Tensor([2 1 0], shape=(3,), dtype=int32)
# tf.Tensor([0 1 1], shape=(3,), dtype=int32)
```

