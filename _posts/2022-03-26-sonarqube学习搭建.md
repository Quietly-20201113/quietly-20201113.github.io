---
layout: post
title:  "sonarqube学习搭建"
date:   2022-03-26 10:02
categories: 前端学习
tags: 前端学习
---
* content
{:toc}
------

####  sonarqube学习搭建[官网地址](https://www.sonarqube.org/)

　<font color=FF0000> 禁止将项目代码上传至云端(官网自带的测试)，所有测试均在本地搭建内网本地测试（sonarqube被攻击了） </font>

> **SonarQube 简介**
>
> Sonar (SonarQube)是一个开源平台，用于管理源代码的质量。 Sonar 不只是一个质量数据报告工具，更是代码质量管理平台。 支持java, C#, C/C++, PL/SQL, Cobol, JavaScrip, Groovy 等等二十几种编程语言的代码质量管理与检测。 Sonar可以从以下七个维度检测代码质量，而作为开发人员至少需要处理前5种代码质量问题。
>
> 1、不遵循代码标准：sonar可以通过PMD,CheckStyle,Findbugs等等代码规则检测工具规范代码编写。
>
> 2、潜在的缺陷：sonar可以通过PMD,CheckStyle,Findbugs等等代码规则检测工具检 测出潜在的缺陷。
>
> 3、糟糕的复杂度分布：文件、类、方法等，如果复杂度过高将难以改变，这会使得开发人员难以理解它们, 且如果没有自动化的单元测试，对于程序中的任何组件的改变都将可能导致需要全面的回归测试。
>
> 4、重复：显然程序中包含大量复制粘贴的代码是质量低下的，sonar可以展示源码中重复严重的地方。
>
> 5、注释不足或者过多：没有注释将使代码可读性变差，特别是当不可避免地出现人员变动时，程序的可读性将大幅下降而过多的注释又会使得开发人员将精力过多地花费在阅读注释上，亦违背初衷。
>
> 6、缺乏单元测试：sonar可以很方便地统计并展示单元测试覆盖率。
>
> 7、糟糕的设计：通过sonar可以找出循环，展示包与包、类与类之间的相互依赖关系，可以检测自定义的架构规则 通过sonar可以管理第三方的jar包，可以利用LCOM4检测单个任务规则的应用情况， 检测耦合。

##### 1、官网下载sonarqube,随便解压到文件夹

> 目录结构 :
>
> bin** 此目录放置各操作系统（LInux、Windows、MacOS）用于启动 SonarQube 服务的工具、脚本；
>
> **conf** 此目录存放SonarQube相关配置文件；
>
> **data** 此目录包含嵌入式数据库(H2数据库引擎)的数据，建议只用于测试和演示；
>
> **elasticsearch** 此目录放置elasticsearch检索引擎相关内容；
>
> **extensions** 此目录存放SonarQube的插件、 扩展jar 包；
>
> **lib** 此目录存放SonarQube所依赖的 jar 包；
>
> **logs** 此目录存放SonarQube相关日志信息；
>
> **tmp**此目录包含服务器所需的临时数据，服务器启动时不要清理；
>
> **web** 此目录存放 SonarQube web 服务的静态资源。

##### 2、准备java环境-----安装jdk,配置环境变量

> 最新的sonarqube必须要 [java jdk 11](https://www.oracle.com/java/technologies/javase/jdk11-archive-downloads.html);

##### 3、测试启动(window为例)

> 访问地址默认 : `http://localhost:9000/`,
>
> 如需要更改或者端口冲突可进入`sonarqube\conf\sonar.properties`对`sonar.host.url`进行更改
>
> 启动服务 : `sonarqube\bin\windows-x86-64\StartSonar.bat`;**双击**

**流程图(typora可以展示)🌈

```flow
	开始=>start: 开始     
    结束=>end: 结束 
    下载=>operation: 下载sonarqube
    解压=>operation: 解压到非中文文件夹内
    jdk=>condition: 是否安装java jdk 11？
    下载jdk=>inputoutput: 下载jdk 11
    安装jdk=>operation: 安装jdk 11 并配置
    启动=>operation: 启动sonarqube

开始->下载->解压->jdk(yes)->启动->结束
jdk(no)->下载jdk->安装jdk->启动
```

##### 4、启动报错

> 遇到一个 : 
>
> jvm 启动失败
>
> 必须安装 jdk 11 ----------------------------------------------------------------------
>
> 有问题,卸载了重新安装

##### 5、使用

![61bfe986f3b3560001bb1481.png](https://github.com/Quietly-20201113/PictureSpace/blob/main/sonarqube/61bfe986f3b3560001bb1481.png?raw=true)


![61bfea8dfafd5900011f8e30.png](https://github.com/Quietly-20201113/PictureSpace/blob/main/sonarqube/61bfea8dfafd5900011f8e30.png?raw=true)


![61bfea9174c73a0001ed9b2b.png](https://github.com/Quietly-20201113/PictureSpace/blob/main/sonarqube/61bfea9174c73a0001ed9b2b.png?raw=true)

****

##### 6、执行

> 只需要到项目文件加下依次执行`Execute the Scanner`下的命令即可

****

##### 7、使用效果

![61bfeb23de155200018bb364.png](https://github.com/Quietly-20201113/PictureSpace/blob/main/sonarqube/61bfeb23de155200018bb364.png?raw=true)