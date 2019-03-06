---
title: python中的异常处理
date: 2019-03-06 20:09:14
tags: python语言特性
---

主要包含`raise`,`assert`和`try...except`结构。

<!-- more -->

# 概述

本文总结python中的异常处理，包含以下内容：
![](C:/Users/Jairo/Blog/Ja1r0.github.io/source/_posts/intro.jpg)

- [Exceptions versus Syntax Errors]()
- [Raising an Exception]()
- [The AssertionError Exception]()
- [The try and except Block:Handling Exception]()
- [The else Clause]()
- [Cleaning Up After Using finally]()
- [Summing up]()

# Exceptions versus Syntax Errors

python脚本在运行的过程中，如果发生了错误就会终止运行，这种错误事件可能是：

- syntax error
- exception

那么这两种情况有什么区别呢？`syntax error`是当出现脚本书写上的语法错误时出现的，不涉及到计算结果或逻辑问题，有如下例子：

```python
print(0 / 0))
```

```
  File "<ipython-input-1-b786f13d5138>", line 1
    print(0 / 0))
                ^
SyntaxError: invalid syntax
```



箭头所指的就是`syntax error`出现的地方，此例子中的语法错误是多了一个括号。修正这个错误再次运行：

```python
print(0 / 0)
```

```
---------------------------------------------------------------------------

ZeroDivisionError                         Traceback (most recent call last)

<ipython-input-2-b6c088d31521> in <module>()
----> 1 print(0 / 0)
```

```
ZeroDivisionError: division by zero
```

此时所发生的就是`exception error`。在脚本中没有`syntax error`的前提下，才会触发`exception error`。提醒信息中会指出此`exception error`的具体类型。在此例中为`ZeroDivisionError`。`exception error`分为两类：

- [build-in exceptions](https://docs.python.org/3/library/exceptions.html)
- self-defined exceptions

# Raising an Exception

用户可以使用`raise`关键字，当满足所设定的条件时，主动触发异常。![](C:/Users/Jairo/Blog/Ja1r0.github.io/source/_posts/raise.jpg)
`raise`的使用方式如下：

```python
x = 10
if x > 5:
    raise Exception('x should not exceed 5.The value of x was: {}'.format(x))
```

```
---------------------------------------------------------------------------

Exception                                 Traceback (most recent call last)

<ipython-input-3-36d986f491c4> in <module>()
      1 x = 10
      2 if x > 5:
----> 3     raise Exception('x should not exceed 5.The value of x was: {}'.format(x))
```

```
Exception: x should not exceed 5.The value of x was: 10
```

程序会在满足`raise`执行的条件后，执行`raise`语句。抛出一个错误，并打印出事先写好的错误信息。

# The AssertionError Exception

在python脚本的编写中，除了被动的等待程序报错，可以用`assert`关键字去主动的去探测一些错误，返回更多的与错误有关的信息。[这里](https://dbader.org/blog/python-assert-tutorial)是一篇介绍`assert`的博客。`assert`关键字后面会跟一个条件判断，当此条件为真时，程序继续正常执行；当条件为假时，则会抛出一个`AssertionError`。也就是说，`assert`语句是要保证脚本运行至此时，满足括号中的条件，如果不满足的话，则触发异常，终止程序。![](C:/Users/Jairo/Blog/Ja1r0.github.io/source/_posts/assert.jpg)见如下例子：

```python
x = 6
assert (x == 5), "x should equal to 5."
```

```
---------------------------------------------------------------------------

AssertionError                            Traceback (most recent call last)

<ipython-input-4-5d942ae404fb> in <module>()
      1 x = 6
----> 2 assert (x == 5), "x should equal to 5."
```

```
AssertionError: x should equal to 5.
```

在这个例子中，程序在抛出一个`AssertionError`异常之后便终止了。那么如果还想再触发异常之后继续执行一些其他的指令呢？

# The try and exception Block:Handling Exceptions

在python中，`try`和`except`结构用来捕捉异常情况，并做一些对应的处理。`try`之后的语句，如果存在异常，则会执行`except`之后的语句。![](C:/Users/Jairo/Blog/Ja1r0.github.io/source/_posts/try_except.jpg)

```python
def value_check(x):
    assert (x == 5), 'x should equal to 5.'
    x *= 2
    print('Everything checks out.x = %d' % x)
```

上面这个函数中有一个`assert`语句，当`x != 5`时，则会抛出一个`AssertionError`异常。而`try`加`except`结构则可以在出现异常时，并不终止程序，进行一些其他操作。

```python
try:
    value_check(6)
except:
    print('The value_check() function threw error.')
```

```
The value_check() function threw error.
```

这个例子中，`value_check()`函数在执行的过程中，出现了异常。但程序并没有`crash`，而是继续运行预先设置的语句，提示这个函数并没有成功执行。但是在此例之中，并不能知道`value_check()`函数在执行中出现的异常具体是什么类型的。若想要知道关于异常的更多的信息可以这样做：

```python
try:
    value_check(6)
except AssertionError as error:
    print('The value_check() function threw error.')
    print(error)
```

```
The value_check() function threw error.
x should equal to 5.
```

打印出的第一条信息说明了`value_check()`函数在执行时抛出了异常。而第二条信息则代表该异常的类型为`AssertionError`异常，提示`value_check()`函数中，要求`x == 5`。下面再看一个例子，这个例子中出现的是`build-in`exception：

```python
try:
    with open('file.log') as file:
        read_data = file.read()
except:
    print('Could not open file.log')
```

```
Could not open file.log
```

因为并不存在`file.log`这个文件，所以就会执行`except`之后的语句。这里出现的异常是`FileNotFoundError`，这是一个`build-in`异常。更多的`build-in exception`见[Build-in Exceptions](https://docs.python.org/3/library/exceptions.html)。在官方文档中，是这样描述`FileNotFoundError`的：

> Raised when a file or directory is requested but doesn’t exist. Corresponds to errno ENOENT.

若想捕获这一异常的具体类型，并打印出异常中包含的信息，可以像之前一样：

```python
try:
    with open('file.log') as file:
        read_data = file.read()
except FileNotFoundError as fnf_error:
    print(fnf_error)
```

```
[Errno 2] No such file or directory: 'file.log'
```

可以同时检测多个类型的异常。需要注意的是，`try`之后的语句，在触发了第一个异常之后，便会终止。

<table><tr><td bgcolor=orange>**Warning**:若`except`之后接的是`Exception`，那么所有类型的异常都会触发这个代码块，也就没有体现出具体是哪一类异常。故这样做是不推荐的，更多的讨论见[这里](https://realpython.com/the-most-diabolical-python-antipattern/)。</td></tr></table>

```python
try:
    value_check(6)
    with open('file.log') as file:
        read_data = file.read()
except FileNotFoundError as fnf_error:
    print(fnf_error)
except AssertionError as ast_error:
    print('The value_check() function threw error.')
    print(ast_error)
```

```
The value_check() function threw error.
x should equal to 5.
```

在上面这段代码中，`try`之后的代码块当执行到`value_check()`函数时便抛出了异常，于是此代码块终止运行，并不会抛出之后的`FileNotFoundError`异常。那么当`value_check()`函数可以正常运行时，则是下面这种情况：

```python
try:
    value_check(5)
    with open('file.log') as file:
        read_data = file.read()
except FileNotFoundError as fnf_error:
    print(fnf_error)
except AssertionError as ast_error:
    print('The value_check() function threw error.')
    print(ast_error)
```

```
Everything checks out.x = 10
[Errno 2] No such file or directory: 'file.log'
```

小结一下：

- `try`之后的代码块，当抛出第一个异常时便终止运行。
- `except`之后的代码块，是用户设定的在该异常发生时的处理方式。
- 可以用`except`关键词检测多种类型的异常，并分别做不同的后续处理。
- 直接用`except`关键词而不管具体是什么类型的异常是不推荐的。

# The else Clause

还可以在`try`和`except`结构之后，再加`else`模块。`else`之后的内容在没有`try`代码块中没有异常抛出时，才会执行的。![](C:/Users/Jairo/Blog/Ja1r0.github.io/source/_posts/try_except_else.jpg)见如下例子：

```python
try:
    value_check(5)
except AssertionError as error:
    print(error)
else:
    print('Executing the else clause.')
```

```
Everything checks out.x = 10
Executing the else clause.
```

在`else`中当然也可以写入`try`加`except`结构：

```python
try:
    value_check(5)
except AssertionError as error:
    print(error)
else:
    try:
        with open('file.log') as file:
            read_data = file.read()
    except FileNotFoundError as fnf_error:
        print(fnf_error)
```

```
Everything checks out.x = 10
[Errno 2] No such file or directory: 'file.log'
```

# Cleaning Up After Using finally

在之上所介绍的`try`加`except`加`else`结构中，`except`和`else`都是有条件执行的。可以再加一个`finally`模块，来无条件的执行一些语句。![](C:/Users/Jairo/Blog/Ja1r0.github.io/source/_posts/try_except_else_finally.jpg)来看下面的例子：

```python
try:
    value_check(5)
except AssertionError as error:
    print(error)
else:
    try:
        with open('file.log') as file:
            read_data = file.read()
    except FileNotFoundError as fnf_error:
        print(fnf_error)
finally:
    print('Cleaning up, irrespective of any exceptions.')
```

```
Everything checks out.x = 10
[Errno 2] No such file or directory: 'file.log'
Cleaning up, irrespective of any exceptions.
```

不论`try`和`else`代码块中出现了什么异常，`finally`代码块都会被执行。

# 总结

在了解了`syntax error`和`exceptions`的区别之后，继续介绍了如何`raise`、`catch`和`handle`异常情况。

- `raise`可以在满足某个条件时主动抛出异常。
- `assert`为了保证满足某个条件，若不满足则抛出异常。
- `try`代码块会被执行直到出现异常时终止。
- `except`可以捕捉`try`中出现的特性类型的异常，并做相对应的处理。
- `else`代码块会在`try`中没有出现异常时被执行。
- `finally`代码块不论之前结构中是否出现异常，都会被执行。

# 参考

https://realpython.com/python-exceptions/