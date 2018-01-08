---
layout:     post
title:      Python图像处理库PIL的Image模块介绍（一）
subtitle:   python学习之路
date:       2018-1-4
author:     feimo
header-img: img/post-bg-python.jpg
catalog: true
tags:
    - python
    - PIL
---
Image模块是PIL中最重要的模块，它有一个类叫做image，与模块名称相同。Image类有很多函数、方法及属性，接下来将依次对image类的属性、函数和方法进行介绍。
### Image类的属性
#### Format
- 定义：im.format ⇒ string or None
- 含义：源文件的文件格式。如果是由PIL创建的图像，则其文件格式为None。
- 例子：
```python
>>>from PIL import Image
>>> im= Image.open("D:\\Code\\Python\\test\\img\\test.jpg")
>>>im.format
'JPEG'
```
注：test.jpg是JPEG图像，所以其文件格式为JPEG。
```python
>>> im= Image.open("D:\\Code\\Python\\test\\img\\test.gif")
>>>im.format
'GIF'
```
注：test.gif为GIF文件，所以其文件格式为GIF。
#### Mode
- 定义：im.mode ⇒ string
- 含义：图像的模式。这个字符串表明图像所使用像素格式。该属性典型的取值为“1”，“L”，“RGB”或“CMYK”。对于图像模式的介绍，可以参考我的blog“Python图像处理库PIL的基本概念介绍”。
- 例子：
```python
>>>from PIL import Image
>>> im = Image.open("D:\\Code\\Python\\test\\img\\test.jpg")
>>> im.mode
'RGB'
>>> im = Image.open("D:\\Code\\Python\\test\\img\\test.gif")
>>> im.mode
'P'
```
#### Size
- 定义：im.size ⇒ (width, height)
- 含义：图像的尺寸，按照像素数计算。它的返回值为宽度和高度的二元组（width, height）。
- 例子：
```python
>>>from PIL import Image
>>> im= Image.open("D:\\Code\\Python\\test\\img\\test.jpg")
>>>im.size
(800, 450)
>>> im= Image.open("D:\\Code\\Python\\test\\img\\test.gif")
>>> im.size
```
#### Palette
- 定义：im.palette ⇒ palette or None
- 含义：颜色调色板表格。如果图像的模式是“P”，则返回ImagePalette类的实例；否则，将为None。
- 例子：

```python
>>> from PIL import Image
>>> im= Image.open("D:\\Code\\Python\\test\\img\\test.jpg")
>>> im.mode
'RGB'
>>>im.palette
>>> im= Image.open("D:\\Code\\Python\\test\\img\\test.gif")
>>> im.mode
'P'
>>>im.palette
<PIL.ImagePalette.ImagePaletteobject at 0x035E7AD0>
>>> pl= im.palette
```


#### Info
- 定义：im.info ⇒ dictionary
- 含义：存储图像相关数据的字典。文件句柄使用该字典传递从文件中读取的各种非图像信息。大多数方法在返回新的图像时都会忽略这个字典；因为字典中的键并非标准化的，对于一个方法，它不能知道自己的操作如何影响这个字典。如果用户需要这些信息，需要在方法open()返回时保存这个字典。
- 例子：

```python
>>>from PIL import Image
>>> im= Image.open("D:\\Code\\Python\\test\\img\\test.jpg")
>>>im.info
{'jfif_version':(1, 1), 'jfif': 257, 'jfif_unit': 1, 'jfif_density': (96, 96), 'dpi': (96, 96)}
>>> im= Image.open("D:\\Code\\Python\\test\\img\\test.gif")
>>>im.info
{'duration':100, 'version': 'GIF89a', 'extension': ('NETSCAPE2.0', 795L), 'background': 0,'loop': 0}
```