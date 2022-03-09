---
layout: post
title: "electron学习之lodop打印机"
date: 2020-09-03 15:56
categories: electron
tags: electron lodop
---

---

## 记录摸索 electron 与 lodop 的辛酸历程

#### 一、放弃在主进程调用 CLodopfuncs.js 服务吧

> 别挣扎了,当时考虑着是服务端渲染,服务端与客户端分离,打算将 lodop 服务调用写在主进程中,经历了三天的折磨,没搞出来
>
> 首先获取不到 doucment,------------------------绝路
>
> 其次将 doucment 从渲染进程传到主进程序列化是不可能的
>
> 从序列化转到使用插件 ,因为序列化 dom 树结果是空,electron 通讯不能传输 dom,报错 : Error: An object could not be cloned.
>
> 丧心病狂的使用插件 XMLserializer +parse5 将 dom 树解析成 XML 传输(终于成功了)
>
> 主进程用插件 DOMParser 解析 XML 成 text/html 类型数据
>
> 结果 lodop 服务里面 CLodopfuncs.js 文件有些运行不完整 ----------------> 个别几个参数出现 is not defined ,炸不炸裂
>
> 三天 electron 不熟悉 lodop 第一天就调起以为超简单,什么鬼心态
>
> 想想我现在写着几句话的有多大的怨

#### 二、老老实实在渲染进程调用

> 做好通信
>
> 就算在渲染进程能调用到主进程 js(我在开窗口的时候给了 nodeIntegrationTo 为 true,原生 html 可使用 node 引入方式),也得走通信,原因是有时候放在 global 的命名空间也可能获取不到(不晓得是什么原因,心态炸了没细查);
>
> 在壳这边写个小页面来获取 doucment(操作烦不烦都无所谓了),放了提示与确定按钮
>
> 大概这个样子

|              |                 |                |
| :----------: | :-------------: | :------------: |
| 正在尝试调起 | 外部打印机,点击 | 确定进行下一步 |
|              |                 |                |
|              |                 |      确定      |

> 调起这个界面的时候就发了个通信将需要打印的 html 经过渲染进程传输到主进程又传输到渲染进程然后保存在这个界面上
>
> 当时想着是在初始化的时候将 CLodopfuncs 加载一遍到时候用就行了,结果谁想得到获取的服务内容赋值给 window 上都会变得不一样(反正不知道哪里不一样代码对了下是一样的,大概想了下应该是这玩意必须实时创建)不能用
>
> 然后将加载服务获取到的 LODOP 拿来调用打印,真是不容易啊!终于成功了
>
> 剩下的就是读文档了[lodop 技术文档](https://raw.githubusercontent.com/Quietly-20201113/Quietly-20201113.github.io/master/screenshot/Lodop技术手册6.2.2.6.doc)

#### 三、lodop 真的是只能打印原生界面，样式可以内联，不内联的可以与内容一起拼接成字符串打印,不会的看[样例 10](http://www.c-lodop.com/demolist/PrintSample10.html)

#### 四、[lodop 官网](http://www.lodop.net)

#### 五、不附代码了
