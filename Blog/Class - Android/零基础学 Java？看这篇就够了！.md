# 零基础学 Java？看这篇就够了！

![icon](https://github.com/Mayan29/Blog/blob/master/Blog/Images/image001.png)

## 一、简介

### Java 语言概述

Java 是 SUN（Stanford University Network，斯坦福大学网络公司）1995 年推出的一门高级编程语言，是一种面向 Internet 的编程语言。

### Java 语言的三种技术架构

Java EE（Java Platform，Enterprise Edition，企业版）
Java 5.0 以前称为 J2EE。Java EE 是在 Java SE 的基础上构建的。简单点说 Java EE 是用来做网站的。

Java SE（Java Platform，Standard Edition，标准版）
Java 5.0 以前称为 J2SE。它允许开发和部署在桌面、服务器、嵌入式环境和实时环境中使用的 Java 应用程序。简单点说 Java SE 是用来做电脑上运行的软件的。

Java ME（Java Platform，Micro Edition，微型版）
Java 5.0 以前称为 J2ME。Java ME 为在移动设备和嵌入式设备上运行的应用程序提供环境。简单点说 Java ME 是用来做手机软件的。

### Java 语言特点

跨平台性。只需在操作系统上安装 Java 虚拟机（JVM，Java Virtual Machine）即可，由 JVM 来负责 Java 程序在该系统中的运行。

### Java 语言环境搭建

JRE（Java Runtime Environment，Java 运行环境）。包括 JVM 和 Java 程序所需的核心类库等。

JDK（Java Development Kit，Java 开发工具包）。其中包含 Java 的开发工具（编译工具、打包工具等）和 JRE，所以安装了 JDK 就不用再单独安装 JRE 了。下载地址：http://www.oracle.com

![icon](https://github.com/Mayan29/Blog/blob/master/Blog/Images/image002.png)

JRE 仅仅是运行工具，如需要开发工具则需搭建 JDK。JDK 开发完成的 Java 程序，交给 JRE 去运行。


## 二、编译流程

### Hello Java

编写如下代码保存为 Demo.java

```java
class Demo {
	public static void main(String[] args) {
		System.out.println("Hello Java");
	}
}
```

使用终端，在 Demo.java 文件当前目录，输入如下命令进行编译：

```
$ javac Demo.java 
``` 

编译成功后在 Demo.java 文件当前目录下生成 Demo.class 文件，输入如下命令运行：

```
$ java Demo
```

输出结果为：

```
Hello Java
```

### Java 名称规范

- 包名：多单词组成时，所有字母都小写  xxxyyyzzz

- 类名、接口名：多单词组成时，所有单词首字母大写  XxxYyyZyy

- 常量名：多单词组成时，每个单词用下划线连接，所有字母都大写  XXX_YYY_ZZZ

### Mac 使用 Sublime Text 3 搭建 Java 环境

Sublime Text 默认的 Java 处理方式仅仅是编译，Build 后会生成一个 .class 文件，而没有去运行，这时候就需要改配置文件达到编译 + 运行的目的。

##### 1. Tools —> Build System —> New Build System

![icon](https://github.com/Mayan29/Blog/blob/master/Blog/Images/image003.png)

##### 2. 添加如下代码：

```java
{
	"shell_cmd": "javac -encoding utf-8 $file_name && java $file_base_name",
	"file_regex": "^ *\\[javac\\] (,+):([0-9]+):() (.*)$",
	"selector": "source.java",
	"encoding": "utf-8"
}
```

##### 3. 直接保存，名字随意

![icon](https://github.com/Mayan29/Blog/blob/master/Blog/Images/image004.png)

##### 4. Tools —> Build System —> 选择上一步保存的配置文件

![icon](https://github.com/Mayan29/Blog/blob/master/Blog/Images/image005.png)

##### 5. cmd + B 编译运行，就有输出结果了

![icon](https://github.com/Mayan29/Blog/blob/master/Blog/Images/image006.png)









