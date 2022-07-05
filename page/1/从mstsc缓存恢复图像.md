---
title: 从mstsc缓存恢复图像
date: 2022-07-05T13:56:26+08:00
lastmod: 2022-07-05T13:56:26+08:00
author: rebootORZ
avatar: /me/gl.jpg
#  authorlink: https://author.site
# cover: /img/cover.jpg
#  images:
#   - /img/cover.jpg
categories:
  - category1
tags:
  - 红队技术
nolastmod: true
draft: true
---

## 前置知识

### BMChache

a. BMChache全称RDP Bitmap Chache，即RDP（远程桌面协议）位图缓存。是Windows为了加速RDP连接时的显示，减少数据量的传输，改善RDP连接体验的一种缓存机制。

b. 这个技术是微软用来解决网络延迟带来的远程桌面卡顿问题的，该技术采用的是图块的形式，当进行RDP连接后，会将画面以位图的形式在本地进行存储，需要知道的是，一个画面并不是只有一张，而是分为多个，例如分成多个常见的64px*64px的位图，当用户进行操作时，只对发生改变的地方进行画面的更新，也就是说从目标那里重新获取这个位置的位图，从而降低了对带宽的消耗。

c. 这个存储是持久化存储，不会因为会话的结束而消失，所以这个技术其实在windows取证方面也是很有用的。

注：

​	a. 位图缓存选项可以由用户配置是否开启，可以打开远程桌面连接程序查看

​	![image-20210804165509044](/imgs/image-20210804165509044.png)

​	

​	b. 位图缓存只存在与远程连接的客户端系统中，而不是服务端系统中。



## BMC缓存分析

**路径**

高版本系统（例如win10）

```
%USERPROFILE%\AppData\Local\Microsoft\Terminal Server Client\Cache\
```

![image-20210804145947490](file:///Users/rebootorz/Documents/%E6%96%87%E7%AB%A0/2021/%E4%BB%8Emstsc%E7%BC%93%E5%AD%98%E6%81%A2%E5%A4%8D%E5%9B%BE%E5%83%8F/%E4%BB%8Emstsc%E7%BC%93%E5%AD%98%E6%81%A2%E5%A4%8D%E5%9B%BE%E5%83%8F/imgs/image-20210804145947490.png?lastModify=1628061651)

低版系统

```
%USERPROFILE%\Local Settings\Application Data\Microsoft\Terminal Server Client\Cache\
```





**文件头**

使用010Editor打开文件，前20个字节分别对应：

35 60 D1 10 F1 0B 41 BF —— 图像hash

40 00 —— 宽度

40 00 —— 高度

00 10 00 00 —— 大小（字节）

11 00 00 00 —— 是否压缩

![image-20210804153647798](/imgs/image-20210804153647798.png)

通过文件头知道高度、宽度都是0x40，所以大小就是64*64 = 8192 字节,图像的位深度:2\*4 = 8，说明每一个像素需要1个字节来存储，则这个区块的大小就是64\*64\*1=8192字节。后面的操作就是将内容提取出来后拼接BMP图像头导出成bmp图像即可。



## 工具分享

上面的操作中，这里需要对bmp图像的格式有了解 ，且如果是手工进行操作的话其实很麻烦，例如你可能会发现恢复后的图像有非常多，一个个找无非大海捞针，还需要自行进行拼接。。

![image-20210804172313760](/imgs/image-20210804172313760.png)



这里推荐三个老外的项目进行配合使用，可以省时省力，项目地址：

bmc-tools: https://github.com/ANSSI-FR/bmc-tools  

RdpCacheStitcher: https://github.com/BSI-Bund/RdpCacheStitcher

BMC Viewer：https://github.com/0xTowel/BMC-Viewer-Backup



**bmc-tools**

这个脚本的作用就是帮助我们从缓存中将bmp图像提取出来，但是这个提取的结果不方便查看，有时候脚本还会报错

![image-20210804220948001](/imgs/image-20210804220948001.png)

**RdpCacheStitcher**

利用AI技术将这些bmp图片进行拼凑，你可以理解为拼图。。然后就可以看到完整图像了。但实际用的时候并不方便，因为图片很多，而他并不是自动完成所有工作的，而是需要你断点击窗口上的"autoplace"，所以实际上他更像是一个拼图的平板，提供一个让你在上面拼图的地方而已。

首先是导入使用bmc tools提取出的bmp图像目录：
![image-20210804221233438](/imgs/image-20210804221233438.png)

选中任意一张图片后再点击上面任意一个方格，然后不断点击“Autoplace”进行图片的拼凑：

但是显然这样很慢。。

![image-20210804221314618](/imgs/image-20210804221314618.png)

**BMC Viewer**

对缓存目录下的.bmc文件进行解析还原出图像，和bmc-tools差不多，只不过用起来感觉这个更方便图片大一点直接能看清内容，脚本的话有时候会报错。

![image-20210804221611211](/imgs/image-20210804221611211.png)

从上图可以看出来，这个工具用起来比较简单方便，图像的展示虽然并没有进行拼接，但是基本上如果有敏感数据的话也能看到个大概，相对于前面的人工去还愿和前两个工具而言，还是比较好用的。

总的来说，这个利用手段其实还是比较鸡肋的，当然对于电子取证来说肯定还是很有用的，包括蓝军的溯源时也可以用来进行辅助分析，但是要用来红队工作中使用还是有点力不从心。