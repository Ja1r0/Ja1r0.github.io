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

所以至此来说，制作成的`TFRecords`文件中样本的顺序，在每次制作时就应该认为是不固定的了。那么看来就没必要保持每次制作数据集时样本的次序一致。在上面的函数中，干脆加入了`random.shuffle()`，来让它乱的彻底一点。