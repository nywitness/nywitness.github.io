---
title: jenkins远程部署go程序
date: 2019-09-05 13:18:45
tags: jenkins
---

> 今年工作的重点是运维系统的自动化部署模块，目前开发完成，为了方便测试，需要用jenkins将go程序自动部署到测试服务器，本文重温一下jenkins配置的美妙旅程。

## 基础配置

### go配置

和配置jdk，maven或git一样，通常想在jenkins中使用go编译，需要在jenkins的global tools中配置go的安装路径。本文案例中由于使用了gox这个跨平台编译工具，不需要进行相关配置。

### windows server配置

因为需要将文件传输到目标服务器的目标文件夹，而目标服务器又是windows server，本身不具有linux系统自带的ssh连接功能，需要曲线救国。本文案例是通过powershell server实现windows的ssh链接功能。

```
Hostname: 远程机器ip
port: 端口号 默认22
Credentials: 远程机器登录账户和密码，在jenkins凭证中维护
```

Check connection返回成功即可。

## 目标

从A机器下载源码，打包编译出可执行文件，传输到B机器的指定路径下，kill正在运行的进程，重启该进程。

分解为jenkins任务的构建步骤：

1. 源码下载
2. gox编译打包
3. 文件传输
4. 重启程序

## 问题

### gox编译，找不到命令

用过jenkins的朋友一定很熟悉源码下载，这一步也不会出现什么问题。首先出现问题的是gox编译，由于gox工具是通过`go get`命令获取，实测中也发现，只有运行`go get`命令的路径能够识别gox指令，直接使用gox命令，自然是找不到。

将gox.exe的路径添加到系统环境变量Path中，jenkins尝试构建，结果依然无法识别gox命令，猜测是因为jenkins并没有从操作系统环境变量中查询可执行文件，所以依然找不到。

为了简便，索性在A机器输入gox.exe全路径：

```go
C:\Go\projects\bin\gox.exe -osarch "windows/amd64 linux/amd64 darwin/amd64"
```

问题解决。

### 参数化构建

jenkins参数化构建，是在构建时给参数赋值，并在构建过程中替换参数位置，案例中使用gox包编译不同平台的可执行文件，所以使用os参数表示操作系统，分别传入windows、linux等。

实际操作过程中，当使用`${os}`注入命令时，jenkins构建失败：

```
C:\jenkins-2.164.3\workspace\go-test>C:\Go\projects\bin\gox.exe -osarch "${os}/amd64" 
No valid platforms to build for. If you specified a value
for the 'os', 'arch', or 'osarch' flags, make sure you're
using a valid value.
```

可以看到，`${os}`被原样输出，并没有赋值，看样子猜错了使用方式。回去看参数化构建插件的说明，发现了问题所在：windows系统需要使用`%parameter%`方式注入，跟环境变量方式相同，改成`%os%`之后，构建成功。

### cmd编码

在jenkins构建任务中，如果配置了windows bat命令，执行时会打开cmd运行并将输出返回到jenkins日志中，由于windows cmd使用的是GBK编码，jenkins系统配置的是全局的UTF-8编码，导致jenkins回显的cmd日志乱码。

#### 解决方案1

win10更新之后，可以在区域设置中，将默认编码改为UTF-8，需要重启计算机使之生效，可以在cmd窗口输入`chcp`查看效果：

```powershell
C:\Users\nywitness>chcp
Active code page: 65001
```

#### 解决方案2

每次执行命令之前，先执行`chcp 65001`，将当次的cmd命令编码设置为65001。

> 方案1可能会导致一些用GBK编码开发的软件出现乱码问题，方案2虽然不彻底，但是讨巧，不会影响系统的其他地方。

### 后台启动exe

前面的问题解决之后，成功在远程机器启动了编译之后的go程序，但是程序日志直接输出到jenkins日志中，导致jenkins构建一直处于运行状态。

改成`start .\go-app.exe`，可以在后台启动程序，jenkins构建成功。