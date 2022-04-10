---
layout: post
title: 2022-03-25-Python PIL 使用笔记
tags: Python
date:    2022-04-10
categories: [学习] 
---
这段时间在想做一个能够将GIF裁剪到指定尺寸，并在GIF上叠放（overlay）一个png图像的脚本，就捡起了很久都没有碰过的python试一下。
在经过查询之后，选择使用**PIL（ pillow）** python的一个第三方类库来实现。
PIL具有强大的图像处理功能，同时对GIF也有一定的处理能力
官方手册：[Pillow — Pillow (PIL Fork) 9.1.0 documentation](https://pillow.readthedocs.io/en/stable/)
## 安装
我使用的是pycharm开发，直接在项目的终端中用pip进行安装即可
	``` 
	pip install Pillow
	```
## PIL读取GIF文件
PIL中最主要的类是 **Image** 通过例化一个**Image**对象来实现对文件的各种操作
``` python
from PIL import Image

im = Image.open('1.gif')
```

## PIL处理GIF文件
在处理GIF的时候需要注意，如果直接对im对象进行处理，只会处理这个GIF的第一帧图像，也就是说直接对在使用PIL对GIF文件处理的时候，需要通过遍历将每一帧图像分别处理来达到处理整个GIF图像的目的

``` python
    try:
		im.seek(0)	#im对象指向GIF文件的第一帧图像
		#do something to im
        while 1:
            im.seek(im.tell() + 1)	#开始遍历
            # do something to im
    except EOFError:
		#当遍历到最后一帧后，会抛出EOP异常来终止遍历，可以在这里写一些log之类的
        pass  # end of sequence
```

## PIL对图像进行裁剪

``` pgsql
Image.crop(box = None)
```

调用整个方法可以依据传入的参量box对当前对象进行裁剪，返回一个Image对象
其中box为一个元组，格式为 **（left,upper,right,lower）**,以图像左上角为坐标原点，向右为X轴，向下为Y轴
 - left ：裁剪框左边框的X轴坐标
 - upper : 裁剪框上边框的Y轴坐标
 - right  : 裁剪框右边框的X轴坐标
 - lower ：裁剪框下边框的Y轴坐标
  通过这四个参数可以圈出一个裁剪框的区域，这个方法返回的就是这个框内的图像的对象
  **这里需要注意一下：这个裁剪框的大小是可以超出图片大小的，最后返回的对象会将超出部分用透明图层补充，例如图片高500px，你的upper设置400，lower设置600那么最后返回的图片对象在图片的最下方有高为100px的透明图层**
  
  ## PIL图像保存

``` irpf90
  Image.save(fp, format=None, **params)
```
对于保存GIF图像来说，需要先准备一个List,在通过遍历处理GIF文件的时候，每次处理完一帧图像，利用**List.append()** 方法将这一帧图像添加到List中
``` python

imagelist = []

    try:
		im.seek(0)	#im对象指向GIF文件的第一帧图像
		#do something to im
		imagelist.append(im)
        while 1:
            im.seek(im.tell() + 1)	#开始遍历
            # do something to im
			imagelist.append(im)
    except EOFError:
		#当遍历到最后一帧后，会抛出EOP异常来终止遍历，可以在这里写一些log之类的
        pass  # end of sequence
```
在调用save方法的时候主要有几个参数需要关注

 - fp  你保存图片的存储位置，字符串
 - params   通过键值对设置选项：
   save_all=True  （将所有帧保存为一个文件）
   append_images=cropped_images[1:]    （所有帧保存的那个LIst）
   duration=30        （每一帧的持续时间，30=0.03s）
   loop=0            （这个GIF循环多少次，0为无数次）

``` python
        cropped_images[0].save(
            'cropped.gif',
            save_all=True,
            append_images=cropped_images[1:],
            duration=30,
            loop=0,
        )
```
这里需要注意一下，如果说你最初传入的这个GIF图像的每一帧的持续时间不一样，那么保存时将“duration”设置为固定值是不明智的，这样会导致保存的GIF持续时间与源文件不一样
可以像保存每一帧图像那样再用一个List来保存每一帧的持续时间，最后保存的时候将这个list传递个"duration"即可
``` python

imagelist = []
myduration = []

    try:
		im.seek(0)	#im对象指向GIF文件的第一帧图像
		#do something to im
		myduration.append(im.info['duration'])
		imagelist.append(im)
        while 1:
            im.seek(im.tell() + 1)	#开始遍历
            # do something to im
			imagelist.append(im)
    except EOFError:
		#当遍历到最后一帧后，会抛出EOP异常来终止遍历，可以在这里写一些log之类的
        pass  # end of sequence
		
		cropped_images[0].save(
            'cropped.gif',
            save_all=True,
            append_images=cropped_images[1:],
            duration=myduration[0:],
            loop=0,
        )
```