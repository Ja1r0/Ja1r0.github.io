---
title: 创建与读取TFRecords文件
date: 2019-03-20 21:05:30
tags: TensorFlow
---

[参考链接](https://my.oschina.net/u/3800567/blog/1637798).用TensorFlow，将本地图片文件，制作为TFRecords文件的数据集，并读取。

<!-- more -->

# 遍历目录及文件的函数

在TensorFlow中有函数`tf.gfile.Walk()`函数可以用来遍历目录下的子目录以及文件。在此记录一下该函数的遍历次序问题，因为涉及到每次制作数据集时，数据集中图片样本的次序是否保持不变。发现该函数和`os.walk()`函数的遍历方式是一样的，且每次遍历结果都相同。

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

