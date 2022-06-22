## 直接上结论

mp4视频在部分手机无法播放播放，加载失败或有声音没有画面的原因视频编码问题或者是声道数的问题，解决方案视频转码。

## **背景**

在工作中遇到mp4视频在 iPhone XR (IOS 14.1) 上无法播放，一直显示加载失败，但是安卓手机上却可以正常播放。还有的mp4视频安卓上有声音但是没有画面。

为了排除因素写了个简单的demo

```
<video src="无法播放视频.mp4" controls="controls">
```

视频在iPhone下依旧无法播放，代码的因素排除了，安卓手机上可以正常播放网络的问题也排除了，换其他iPhone测试依旧无法播放，手机的因素也排除了。

mp4视频无法播放的问题很可能出在视频本身。

查询 Apple 官网的[设备技术规格](https://support.apple.com/zh_CN/specs) 在 [iPhone 6 - 技术规格](https://https://support.apple.com/kb/SP705?viewlocale=zh_CN&locale=zh_CN)

![1712668a0f0c96baf541936ec7bb99f.png](https://b3logfile.com/file/2022/06/1712668a0f0c96baf541936ec7bb99f-090edaff.png)

明确的写到支持的视频格式：H.264 视频：最高支持 1080p、60 fps、High Profile level 4.2 编码

对照无法播放视频的编码信息找到了无法播放的真正原因视频编码问题，视频转码到High Profile level 4.2，问题解决了视频可以正常播放。（如何转码最后部分会介绍）

再次查看问题手机 [iPhone XR  - 技术规格](https://support.apple.com/kb/SP781?viewlocale=zh_CN&locale=zh_CN)

![2628a4d3517291652ea570b9aaa6fb0.png](https://b3logfile.com/file/2022/06/2628a4d3517291652ea570b9aaa6fb0-70a84aac.png)

支持的视频格式：HEVC、H.264、MPEG-4 Part 2 与 Motion JPEG

并没有明确写出支持的视频编码

[iPhone 4s - 技术规格](https://support.apple.com/kb/SP655?viewlocale=zh_CN&locale=zh_CN)

![f047cb5fd6c2a6f764649d45e67a9bd.png](https://b3logfile.com/file/2022/06/f047cb5fd6c2a6f764649d45e67a9bd-f0de3fc7.png)

支持的视频格式：H.264 视频，高达 1080p，每秒 30 帧，High Profile level 4.1

**iPhone 4s 只支持到High Profile level 4.1，iPhone 6 支持到 High Profile level 4.2。**

##### Profile和level是什么

Profile和level是视频的压缩级别，我也看过各大佬对Profile和level的专业解释我通俗的介绍下

H.264的Profile和level 可以理解为 gzip的level 越高的等级文件压缩越小，传输越快，但cpu消耗越多。

##### 这样说来Profile和level越高越好为啥不同手机支持的不一样而且不支持高些呢

压缩级别越高不仅在压缩时消耗cpu越多，在视频播放时也需要解压同样消耗cpu越多，手机不一样硬件也不一样所以支持的压缩级别也不一致

##### 该怎么选择Profile和level呢

我认为支持到iPhone 6，使用High Profile level 4.2

##### H.264，H.265是什么

H.264和H.265都是视频编码格式，可以理解为zip，rar这种不同的格式。

H.265是H.264的升级版它的压缩率更高，但是很少浏览器支持。

使用caniuse查询下[H.265兼容性](https://caniuse.com/?search=h265)通红一片，大部分浏览器都不兼容，自行百度h265为什么不普及吧

![](https://b3logfile.com/file/2022/06/5b2bcc87263b4e98b606ce28d180f894.png)

### 编码正确iPhone (IOS) 上还是无法播放

注意另外一个细节在 [iPhone 6 - 技术规格](https://support.apple.com/kb/SP705?viewlocale=zh_CN&locale=zh_CN)  在支持的视频格式中

其音频为 AAC-LC 格式、最高支持 160 Kbps、48kHz、立体声。某些高档的拍摄设备拍出来的是4声道而立体声指的是2声道。所以不仅需要正确声道数也不能超过2。

最后补充下畸形的分辨率也会导致视频无法播放

## 最后介绍下如何转码

使用ffmpeg转码，市面上绝大部分软件都是用ffmpeg包装的

使用ffmpeg把视频转码为H.264的High Profile level 4.2

```
ffmpeg -i 无法播放的视频文件.mp4 -vcodec h264 -profile:v high -level 4.2 转码后的视频.mp4
```

## 总结

本文重点 Profile和level 压缩级别，很多人认为压缩视频就是把视频的分辨率或码率调低，这种方式是有损压缩，提高压缩级别则是**无损压缩**。

很多压缩软件或视频转码软件是没有Profile和level选项的，主要原因也是怕压缩级别过高某些环境下视频无法播放。

转码或压缩后要不就是不对Profile和level修改要不就转码为Main Profile level 3.1，为啥是Main Profile level 3.1，查看 [iPhone 4  - 技术规格](https://support.apple.com/kb/SP587?viewlocale=zh_CN&locale=zh_CN)  它支持的最高也是这个档位，而我认为它是过时的 ，我们不应该在面向iPhone 4或以那时代的产物，还是应该提高压缩级别。

为此我做了一个[在线视频压缩](https://convert.dxcweb.com/)的网站 https://convert.dxcweb.com/ 提高压缩级别，默认无损压缩，压缩级别为High Profile level 4.2，相比Main Profile level 3.1在相同的分辨率和码率的情况下压缩率提高了很多。当然视频无法播放使用本软件压缩后自然能解决，本软件20M免费，大于20M按视频分钟计费。我也正在考虑是否开源，关注我的[GitHub](https://github.com/dxcweb)。
