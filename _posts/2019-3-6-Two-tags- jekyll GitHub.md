---
layout: post
title:  "Jekyll+GitMent搭建"
date:   2019-03-05 20:56
categories: jekyll
tags: jekyll
---
* content
{:toc}
------

 &emsp;&emsp;之前一直想着搭建一个知识库，至于博客之前是在**CSDN**上写，也没写多少，偶然发现**GitHub**上可以搭建一个博客，就当做自己的知识库搭建一个了。  
**这个是测试图片怎么玩（感觉md这个图片挺麻烦的,用cmd Markdown写的)，目前也打算将文章中的图片放在博客GitHub仓库中了，不想重新弄仓库，这几记录下获取GitHub中图片地址，有点尴尬，直接右键图片取出地址的**
![此处输入图片的描述][1]
&emsp;&emsp;总得来说搭建这个还是挺简单的，如果有人Clone了我的[博客主题](https://github.com/Quietly-20201113/Quietly-20201113.github.io.git)，Clone之后需要更改![][4]
**我在这里说一下需要更改到的地方：**  
1.这是图标![][2]，有ico格式和svg;  
2.更改**_config.yml**这个文件内容具体如下  

```js
  title: '还是夸张一点技术专栏'//这个是你自己的博客名  
  description: '一个专注于开发的普通技术民工。'//签名  
  keyword: 'C# VB Vue 小程序'//放的狠话  
  url: 'https://Quietly-20201113.github.io' // 自己的博客地址  
```
```javascript
//Author 配置博主信息  
author: '还是夸张一点'  
nickname: '还是夸张一点'  
bio: 'C# VB Vue 小程序'  
avatar: '/assets/img/profile.png'//这个是头像  
```
**评论功能就不需要弄了，我已经弄进去了**  
3.更改评论的配置在**post.html**文件中，找到以下代码  

```html

  <div id="gitmentContainer"></div>  
  <link rel="stylesheet" href="/assets/css/default.css"/>  
  <script src="/assets/js/gitment.browser.js"></script>  
  <script src="/assets/js/js-MD5.js"></script>  
  <script>  
  var gitment = new Gitment({  
      id: md5(window.location.pathname),  
      owner: 'Quietly-20201113',  
      repo: 'Quietly-20201113.github.io',  
      oauth: {  
          client_id: '1899bc3b6e1494ce68b5',  
          client_secret: 'b934e6fa3b90f3f129e5698915b9949d8f59b3d2',  
      },  
  });  
  gitment.render('gitmentContainer');  
  </script>  

```
`md5(window.location.pathname)`这句代码可以更改，是生成关于评论的唯一值，我用md5加密了，也不需要改，如果不写id的话会出现默认的id值长度超过50的问题，  
`owner: 'Quietly-20201113'`  
`repo: 'Quietly-20201113.github.io'`这两句就是用户名以及这个仓库的名
下面的是需要自己去手动更改的，去注册[注册新的OAuth应用程序](https://github.com/settings/applications/new)取出client_id、client_secret放进来下面附图![][3]  
4.自己更改图标，有一些的图标是**svg**格式的，自己网上在线转一下；  
5、首页的背景也可以更改，包括标签页，博客详细页都可以更改，需要自己调一下样式图片之类的；  
6、文件**README.md**里面是介绍自己博客背景以及预览之类的，自己改改，你也不希望别人看你GitHub介绍链接到我的博客吧； 

7、说一个评论的坑，初期搭建的时候可能同一个博客会出现多次初始化，导致之前的评论内容不见了，因为每次初始化相当于将当前博客的id给换了，所以会出现找不到的问题，第二个就是**GitMent**对IE内核有要求，win10自带的那个**Microsoft   Edge**不支持GitHub登录评论功能，出现的问题我在汉化评论的那里标注了，出现问题可以自己查一查。

8、本地跑**Jekyll**,这个比较麻烦我也不打算写给个链接，一般都能装对：[[jekyll本地环境搭建(Windows)](https://www.cnblogs.com/yevon/p/3308158.html)] <sub>链接原著为博客园叶子([yevon](https://home.cnblogs.com/u/yevon/))博客</sub>,安装成功之后将**_config.yml**文件中一下代码屏蔽：  

```js
plugins: [jekyll-paginate]
paginate: 6
port: 4001
```

在本地仓库运行：**jekyll serve**出现以下情况说明没问题：

```js
PS I:\MeGitHub\Quietly-20201113.github.io> jekyll serve
Configuration file: I:/MeGitHub/Quietly-20201113.github.io/_config.yml
       Deprecation: You appear to have pagination turned on, but you haven't included the `jekyll-paginate` gem. Ensure you have `plugins: [jekyll-paginate]` in your configuration file.
            Source: I:/MeGitHub/Quietly-20201113.github.io
       Destination: I:/MeGitHub/Quietly-20201113.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 1.427 seconds.
  Please add the following to your Gemfile to avoid polling for changes:
    gem 'wdm', '>= 0.1.0' if Gem.win_platform?
 Auto-regeneration: enabled for 'I:/MeGitHub/Quietly-20201113.github.io'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.

```

***



[1]:https://raw.githubusercontent.com/Quietly-20201113/Quietly-20201113.github.io/master/screenshot/1494404591.png
[2]:https://raw.githubusercontent.com/Quietly-20201113/Quietly-20201113.github.io/master/favicon.ico
[3]:https://raw.githubusercontent.com/Quietly-20201113/Quietly-20201113.github.io/master/screenshot/20190305202608.png
[4]:https://raw.githubusercontent.com/Quietly-20201113/Quietly-20201113.github.io/master/screenshot/20190305172642.png
