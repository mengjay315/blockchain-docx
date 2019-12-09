# java学习

## 开发工具

IDEA 2019.1.3 社区版，免费

通过JetBrains Toolbox安装

## Ubuntu 18.04安装Java JDK8三种方式

https://www.cnblogs.com/niuniuc/p/10736499.html

Java JDK在linux系统有两个版本，一个开源版本Openjdk，还有一个oracle官方版本jdk，oracle JDK既可以通过添加ppa源命令行安装，也可以去官网下载jdk压缩包安装。下面分别记录一下这三种安装方式的步骤。

安装openjdk
1、更新软件包列表：

sudo apt-get update
2、安装openjdk-8-jdk：

**sudo apt-get install openjdk-8-jdk**
3、查看java版本，看看是否安装成功：

java -version

**我采用的是第一种方式，使用openjdk安装**

![1563167496893](assets/1563167496893.png)