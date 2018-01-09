---
layout:     post
title:      Python图像处理库PIL的Image模块介绍（二）
subtitle:   python学习之路
date:       2018-1-4
author:     feimo
header-img: img/post-bg-python.jpg
catalog: true
tags:
    - python
    - PIL
---
### Image类的函数
#### New
- 定义：Image.new(mode,size) ⇒ image
   Image.new(mode, size, color) ⇒ image
- 含义：使用给定的变量mode和size生成新的图像。Size是给定的宽/高二元组，这是按照像素数来计算的。对于单通道图像，变量color只给定一个值；对于多通道图像，变量color给定一个元组（每个通道对应一个值）。在版本1.1.4及其之后，用户也可以用颜色的名称，比如给变量color赋值为“red”。如果没有对变量color赋值，图像内容将会被全部赋值为0（图像即为黑色）。如果变量color是空，图像将不会被初始化，即图像的内容全为0。这对向该图像复制或绘制某些内容是有用的。
- 例子：
```python
>>>from PIL import Image
>>> im= Image.open("test.jpg")
>>>im.format
'JPEG'
```
图像im为128x128大小的红色图像。
```python
>>> im= Image.new("RGB", (128, 128))
>>>im.show()
```
图像im为128x128大小的黑色图像，因为变量color不赋值的话，图像内容被设置为0，即黑色。
#### Open
- 定义：Image.open(file) ⇒ image
  Image.open(file, mode) ⇒ image
- 含义：打开并确认给定的图像文件。这个是一个懒操作；该函数只会读文件头，而真实的图像数据直到试图处理该数据才会从文件读取（调用load()方法将强行加载图像数据）。如果变量mode被设置，那必须是“r”。
用户可以使用一个字符串（表示文件名称的字符串）或者文件对象作为变量file的值。文件对象必须实现read()，seek()和tell()方法，并且以二进制模式打开。
- 例子：
```python
>>> from PIL import Image
>>> im= Image.open("test.jpg")
>>>im.show()
>>> im= Image.open("test.jpg", "r")
>>>im.show()
```
#### Blend
- 定义：Image.blend(image1,image2, alpha) ⇒ image
- 含义：使用给定的两张图像及透明度变量alpha，插值出一张新的图像。这两张图像必须有一样的尺寸和模式。
合成公式为：out = image1 *(1.0 - alpha) + image2 * alpha
如果变量alpha为0.0，将返回第一张图像的拷贝。如果变量alpha为1.0，将返回第二张图像的拷贝。对变量alpha的值没有限制。
- 例子：
```python
>>>from PIL import Image
>>> im01 =Image.open("test01.jpg")
>>> im02 =Image.open("test02.jpg")
>>> im =Image.blend(im01, im02, 0.3)
>>> im.show()
```
test01.jpg和test02.jpg两张图像size都为1024x768，mode为“RGB”。它们按照第一张70%的透明度，第二张30%的透明度，合成为一张。
![](https://i.imgur.com/CnSIFGF.png)
#### Composite
- 定义：Image.composite(image1,image2, mask) ⇒ image
- 含义：使用给定的两张图像及mask图像作为透明度，插值出一张新的图像。变量mask图像的模式可以为“1”，“L”或者“RGBA”。所有图像必须有相同的尺寸。
- 例子：
```python
>>>from PIL import Image
>>> im01 =Image.open("test03.jpg")
>>> im02 =Image.open("test04.jpg")
>>>r,g,b = im01.split()
>>> g.mode
'L'
>>> g.size
(1024, 768)
>>> im= Image.composite(im03, im04, g)
>>>im.show()
```
![](https://i.imgur.com/BMJ50b9.png)
#### Eval
- 定义：Image.eval(image,function) ⇒ image
- 含义：使用变量function对应的函数（该函数应该有一个参数）处理变量image所代表图像中的每一个像素点。如果变量image所代表图像有多个通道，那变量function对应的函数作用于每一个通道。注意：变量function对每个像素只处理一次，所以不能使用随机组件和其他生成器。
- 例子：
```python
>>>from PIL import Image
>>>im01 = Image.open("test01.jpg")
>>> def fun(x):
         return x * 0.5
>>>im_eval = Image.eval(im01, fun)
>>>im_eval.show()
>>>im01.show()
```
图像im01如下图：
![](https://i.imgur.com/IiJaedi.png)
图像im_eval如下图：
![](https://i.imgur.com/grK9hf5.png)
图像im_eval与im01比较，其像素值均为im01的一半，则其亮度自然也会比im01暗一些。
#### Frombuffer
- 定义：Image.frombuffer(mode,size, data) ⇒ image
  Image.frombuffer(mode, size,data, decoder, parameters) ⇒ image

- 含义：（New in PIL 1.1.4）使用标准的“raw”解码器，从字符串或者buffer对象中的像素数据产生一个图像存储。对于一些模式，这个图像存储与原始的buffer（这意味着对原始buffer对象的改变体现在图像本身）共享内存。并非所有的模式都可以共享内存；支持的模式有“L”，“RGBX”，“RGBA”和“CMYK”。对于其他模式，这个函数与fromstring()函数一致。
注意：版本1.1.6及其以下，这个函数的默认情况与函数fromstring()不同。这有可能在将来的版本中改变，所以为了最大的可移植性，当使用“raw”解码器时，推荐用户写出所有的参数，如下所示：
im =Image.frombuffer(mode, size, data, "raw", mode, 0, 1)
函数Image.frombuffer(mode,size, data, decoder, parameters)与函数fromstring()的调用一致。
#### Fromstring
- 定义：Image.fromstring(mode,size, data) ⇒ image
  Image.fromstring(mode, size,data, decoder, parameters) ⇒ image
- 含义：函数Image.fromstring(mode,size, data)，使用标准的“raw”解码器，从字符串中的像素数据产生一个图像存储。
函数Image.fromstring(mode,size, data, decoder, parameters)也一样，但是允许用户使用PIL支持的任何像素解码器。更多信息可以参考：Writing YourOwn File Decoder.
注意：这个函数只对像素数据进行解码，而不是整个图像。如果用户的字符串包含整个图像，可以将该字符串包裹在StringIO对象中，使用函数open()来加载。
####  Merge
- 定义：Image.merge(mode,bands) ⇒ image
- 含义：使用一些单通道图像，创建一个新的图像。变量bands为一个图像的元组或者列表，每个通道的模式由变量mode描述。所有通道必须有相同的尺寸。
变量mode与变量bands的关系：
len(ImageMode.getmode(mode).bands)= len(bands)
- 例子：
```python
>>>from PIL import Image
>>> im01 =Image.open("test01.jpg")
>>> im02 =Image.open("test02.jpg")
>>>r1,g1,b1 = im01.split()
>>>r2,g2,b2 = im02.split()
>>>r1.mode
'L'
>>>r1.size
(1024, 768)
>>>g1.mode
'L'
>>>g1.size
(1024, 768)
>>>r2.mode
'L'
>>>g2.size
(1024, 768)
>>>imgs=[r1,g2,b2]
>>>len(ImageMode.getmode("RGB").bands)
3
>>>len(imgs)
3
>>>im_merge = Image.merge("RGB", imgs)
>>>im_merge.show()
```
图像im_merge如下所示：
![](https://i.imgur.com/Q5FEU96.png)