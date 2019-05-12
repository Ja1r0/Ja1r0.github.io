---
title: Python中的装饰器
date: 2019-05-11 23:20:13
tags: Python
---

从基本样式、原函数带有参数、装饰器带有参数三种情况来介绍Python中的装饰器。

<!-- more -->

装饰器的作用是为原有的函数或类增加一些额外的操作。在Python中函数可以作为一个参数传递给另一个函数，装饰器正是利用了这一语言特性。装饰器本身也是一个函数或类，将装饰器比作外套，在写好装饰器之后，可以给各种原有的函数穿上这一外套，在原有函数的基础上增加一些操作。这样一来便可以达到精简重复代码的作用。

# 基本样式

当希望给原有函数`foo()`附加一些其他操作，而又不想对函数`foo()`进行更改的时候，可以新定义一个函数完成这些附加操作，并将`foo`作为参数传递给该函数。如下所示：

```python
def foo():
    print('Hello world!')
    
def check_name(func):
    assert (func.__name__ == 'foo'), 'Incorrect function name!'
    func()
    
check_name(foo)
```

以上处理方式的弊端在于，原来的对`foo()`函数的执行语句需要更改为`check_name(foo)`。而利用装饰器，则可以不改变原函数的执行语句。以下为一个简单的装饰器：

```python
def check_name(func):
    def wrapper():
        assert (func.__name__ == 'foo'), 'Incorrect function name!'
        return func()
    return wrapper

def foo():
    print('Hello world!')
    
foo = check_name(foo)
foo()
```

在Python中，@符号是装饰器的语法糖。可以将`foo = check_name(foo)`语句省略。

```python
def check_name(func):
    def wrapper():
        assert (func.__name__ == 'foo'), 'Incorrect function name!'
        return func()
    return wrapper

@check_name
def foo():
    print('Hello world!')
    
foo()
```

# 原函数带有参数

当原函数带有参数时，例如有参数`date`，此时装饰器的写法如下：

```python
def check_name(func):
    def wrapper(date):
        assert (func.__name__ == 'foo'), 'Incorrect function name!'
        return func(date)
    return wrapper

@check_name
def foo(date):
    print('Hello world! This is {}.'.format(data))

foo(date)
```

这里对原函数进行装饰的原本过程为：

```python
foo = check_name(foo) # foo = wrapper
foo(date) # wrapper(date)
```

如果有不知道数目的多个参数，可以使用`*args`和`**kwargs`来解决。

```python
def check_name(func):
    def wrapper(*args, **kwargs):
        assert (func.__name__ == 'foo'), 'Incorrect function name!'
        return func(*args, **kwargs)
    return wrapper

@check_name
def foo(date, weather='fine', temperature=None):
    print('Hello world! This is {}. The weatheris {} with temperature {}.'.format(data, weather, temperature))

foo(date, weather='fine', temperature=None)
```

对原函数的装饰过程为：

```python
foo = check_name(foo) # foo = wrapper 
foo(date, weather='fine', temperature=None) # wrapper(date, weather='fine', temperature=None)
```

# 装饰器带有参数

当装饰器函数需要一些参数时，需要在原装饰器函数的基础上再包装一层函数，用来接收参数。

```python
def check_name(name):
    def decorator(func):
        def wrapper(date):
            assert (func.__name__ == name), 'Incorrect function name!'
        	return func(date)
        return wrapper
    return decorator

@check_name(name='foo')
def foo(date):
    print('Hello world! This is {}.'.format(data))

foo(date)   
```

Python可以发现`check_name(name)`对`decorator(func)`的封装，并把参数`name`传递到装饰器环境中。`@check_name(name='foo')`相当于`@decorator`，于是这一对原函数的装饰过程为：

```python
foo = decorator(foo) # wrapper
foo(date) # wrapper(date)
```

