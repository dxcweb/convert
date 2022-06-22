## 概述

mp4视频在部分手机上加载失败、无法播放或有声音没画面的原因是：视频编码或声道数有问题，解决方案：视频转码。

## 遇到的问题

1. mp4视频在 iPhone XR (IOS 14.1) 上无法播放，一直显示加载失败，但是在安卓手机上却可以正常播放；

2. 一些mp4视频在安卓手机上有声音但是没有画面。

## 问题定位
为了排除是代码的问题，写了个简单的demo：

```
<video src="无法播放的视频.mp4" controls="controls">
```

视频在iPhone下依旧无法播放，代码的问题被排除了，安卓手机上可以正常播放，网络的问题也排除了，换其他iPhone测试依旧无法播放，手机的问题也排除了。

mp4视频无法播放的问题很可能出在**视频本身**。

## 如何解决
### 查看官方文档
查询 Apple 官网的[设备技术规格](https://support.apple.com/zh_CN/specs) 在 [iPhone 6 - 技术规格](https://support.apple.com/kb/SP705?viewlocale=zh_CN&locale=zh_CN)

![iPhone 6 支持的视频格式](https://upload-images.jianshu.io/upload_images/6765590-d469f6cd582a7c9a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



明确写到支持的视频格式：H.264 视频：最高支持 1080p、60 fps、High Profile level 4.2 编码

对照无法播放视频的编码信息，找到了无法播放的真正原因为**视频编码问题**，把视频转码到High Profile level 4.2，视频可以正常播放。（如何转码最后部分会介绍）

进一步查看问题手机 [iPhone XR - 技术规格](https://support.apple.com/kb/SP781?viewlocale=zh_CN&locale=zh_CN)

![iPhone XR 支持的视频格式](https://upload-images.jianshu.io/upload_images/6765590-d8624e669377ae93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


支持的视频格式：HEVC、H.264、MPEG-4 Part 2 与 Motion JPEG

并没有明确写出支持的视频编码

[iPhone 4s - 技术规格](https://support.apple.com/kb/SP643?viewlocale=zh_CN&locale=zh_CN)

![iPhone 4s 支持的视频格式](https://upload-images.jianshu.io/upload_images/6765590-f986989483a34081.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


支持视频格式：H.264 视频最高可达 1080p，每秒 30 帧，High Profile level 4.1

### 文档结论
**iPhone 4s 只支持到High Profile level 4.1编码，iPhone 6 支持到 High Profile level 4.2编码。**

### 为什么视频编码正确，在iPhone (IOS) 上还是无法播放

#### 1. 声道的问题

注意另外一个细节 [iPhone 6 - 技术规格](https://support.apple.com/kb/SP705?viewlocale=zh_CN&locale=zh_CN)  

如文档中所述，在支持的视频格式中：其音频为 AAC-LC 格式、最高支持 160 Kbps、48kHz、立体声。

但是某些高档的拍摄设备拍出来的视频是4声道，而立体声是2声道，所以不仅需要正确的编码格式，声道数也不能超过2。

#### 2. 分辨率的问题

 畸形的分辨率也会导致视频无法播放。


## 技术说明

##### Profile和level是什么？

Profile和level是视频的压缩级别，我也看过各大佬对Profile和level的专业解释，简单来说：

H.264的Profile和level 可以理解为 gzip的level， 等级越高，文件压缩得越小，传输越快，但cpu消耗越多。

##### Profile和level越高越好吗？

压缩级别越高不仅在压缩时cpu的消耗越高，视频在播放时也需要消耗更多的cpu进行解压，各类型手机的硬件条件不一样，所以支持的压缩级别也不同

##### 该怎么选择Profile和level呢？

支持到iPhone 6及以上，使用High Profile level 4.2即可

##### H.264，H.265是什么

H.264和H.265都是视频编码格式，可以理解为zip，rar这种不同的格式。

H.265是H.264的升级版，它的压缩率更高，但是支持的浏览器较少。

使用caniuse查询[H.265兼容性](https://caniuse.com/?search=h265)通红一片，大部分浏览器都不兼容

![image.png](https://upload-images.jianshu.io/upload_images/6765590-6c3f92e2e1263e51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 最后介绍下如何转码

使用ffmpeg把视频转码为H.264的High Profile level 4.2

```
ffmpeg -i 无法播放的视频文件.mp4 -vcodec h264 -profile:v high -level 4.2 转码后的视频.mp4
```

## 总结

很多人认为压缩视频就是把视频的分辨率或码率调低，但这种方式其实是**有损压缩**，提高Profile和level压缩级别则是**无损压缩**。

很多压缩软件或视频转码软件是没有Profile和level选项的，主要原因也是考虑到视频的压缩级别过高，在某些环境下无法播放。

现在市场上流行的转码软件，在转码或压缩时：

1.有的不对Profile和level修改，直接进行有损压缩；

2.有的是直接转码为Main Profile level 3.1，是因为iPhone 4 支持的最高就是这个档位，具体文档 [iPhone 4  - 技术规格](https://support.apple.com/kb/SP587?viewlocale=zh_CN&locale=zh_CN) 

然而，这个档位的压缩率很低，压缩后的视频文件会很大，所以我们不应该再面向iPhone 4或那时代的产物，而是应该提高压缩级别。

<br>

为此我做了一个[在线视频压缩](https://convert.dxcweb.com/)的网站 https://convert.dxcweb.com/ 提高压缩级别，默认为无损压缩，压缩级别为High Profile level 4.2。

相比Main Profile level 3.1在相同的分辨率和码率的情况下压缩率提高了很多，实测压缩前23.32MB压缩后2.53MB。
 
**所有视频无法播放的问题，使用本软件压缩后都能解决！** 本软件20M免费，大于20M按视频时长计费。

我也正在考虑是否开源，关注我的[GitHub](https://github.com/dxcweb)。
