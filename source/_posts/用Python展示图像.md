---
title: 用Python展示图像
date: 2019-05-27 14:38:18
tags:
---

目前完成了用cv2展示的部分。待续。

<!-- more -->

# 使用cv2

## cv2.waitKey

该函数控制着图像的展示时长。其有一个参数`delay`：cv2.waitKey([delay])。该函数工作的前提是，当前有至少一个活动的窗口，当有多个活动窗口时，它们中的任何一个中发生的事件都会传入该函数。下面以例子说明参数`delay`的各种情况，以及对应的函数功能。

### delay <= 0

图像会一直展示，直到发生一个键盘事件，即你按下了键盘上的一个键。

```python
import cv2

def imgshow():
	img = cv2.imread('test.jpg')
	cv2.imshow('0', img)
	cv2.waitKey(0)
    cv2.destroyAllWindows()
	print('Done!')
```

这个键盘事件发生后，cv2.waitKey(0)函数是会返回一个数值的，该数值为所按键位上面的字符对应的ASCII码。

```python
import cv2

def imgshow():
	img = cv2.imread('test.jpg')
	cv2.imshow('0', img)
	k = cv2.waitKey(0)
    cv2.destroyAllWindows()
    print(k)
	print('Done!')
```

例如按下键盘的`q`键之后，返回的值为`k=113`。利用该函数的键盘事件监听功能，可以实现对不同按键的不同响应：

```python
import cv2

def imgshow():
	img = cv2.imread('test.jpg')
    cv2.imshow('0', img)
    k = cv2.waitKey(1000)
    
    # wait for ESC to exit
    if k == 27: 
        cv2.destroyAllWindows()
    # wait for 's' key to save and exit
    elif k == ord('q'):
        cv2.imwrite('saveimg.jpg', img)
        cv2.destroyAllWindows()

	print('Done!')
```

### delay > 0

此时`delay`的值表示图像将会展示多长时间，单位为千分之一秒。

- 如果在此期间没有键盘事件发生，则cv2.waitKey返回`-1`；

- 若此期间有按键事件发生，则返回按键字符对应的ASCII码。

利用此监听键盘事件的功能，可以实现一些功能。

```python
import cv2

def imgshow():
    img = cv2.imread('test.jpg')
    
    cv2.imshow('0', img)
    k = cv2.waitKey(2000)
    cv2.destroyAllWindows()
    
    if k != -1:
        try:
            key = chr(k)
            print('key {} pressed! value is {}'.format(key, k))
        except ValueError:
            print('cant figure out which key pressed! value is {}'.format(k))        
    else:
        print('no key pressed, show image 2 second.')

    print('Done!')
```

再比如常用的按`q`退出：

```python
import cv2

def imgshow():
	img = cv2.imread('test.jpg')
    
    while True:
        cv2.imshow('0', img)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            cv2.destroyAllWindows()
            break

	print('Done!')
```

注意上面这个例子中，cv2.waitKey的参数任意，可以大于零、可以小于零、可以等于零。还有一点要弄清的是，关于`cv2.waitKey(1) & 0xFF`这个语句，`0xFF`是十六进制的255，其二进制的值如下：

> 00000000 00000000 00000000 11111111

一个数和`0xFF`经过`&`操作后，只留下该数的最后一个字节（byte），因为ASCII码在内存中占用一个字节（ASCII码是七位编码，共用8个bit位存储，最高位统一为0）。这一操作是防止出现bug。

显示视频也可以用上面那个函数，只需稍加改动：

```python
import cv2

cap = cv2.VideoCapture(0)

while True:
    ret, img = cap.read()
    cv2.imshow('frame', img)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        cv2.destroyAllWindows()
        break
        
cap.release()
```