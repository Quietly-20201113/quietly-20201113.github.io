---
layout: post
title:  "flutter插件fluwx ios配置调起微信"
date:   2021-06-24 14:39
categories: Flutter
tags: Flutter IOS
---
* content
{:toc}
------

### 甲、安装[Fluwx](https://pub.flutter-io.cn/packages/fluwx)插件

### 乙、配置apple-app-site-association文件

> ```flutter
> {
>     "applinks": {
>         "apps": [],
>         "details": [
>             {
>                 "appID": "teamID.应用BundId",
>                 "paths": [ "/app/*" ]
>             }
>         ]
>     }
> }
> ```
>
> **teamID 与应用BundId参数查找**
>
> > 1、`https://developer.apple.com/`-->`account`
> >
> > 2、![示例图1](https://github.com/Quietly-20201113/PictureSpace/blob/main/flutter_fluwx/flutter-fluwx-ios-account.png?raw=true "示例图1")
> >
> > 3、点击`identifiers`
> >
> > 4、找到自己应用点击
> >
> > 5、找到**Edit your App ID Configuration**可以看到`App ID Prefix`与`Bundle ID`
> >
> > 6、将`App ID Prefix`赋值给`teamID`,括号内容不需要；将`Bundle ID`赋值给`应用BundId`
> >
> > 7、appID完整数据为`teamID.应用BundId`中间的点别忘记
> >
> > 8、`paths`内容一般需要给个地址防止网页内部到处调起打开app的内容

### 丙、配置Associated Domains（苹果网站）

**位置在乙项第1、2、3、4、5节点处**

**Capabilities的Tab页**

> 1、将`Associated Domains`复选框选中
>
> 2、点击`save`按钮保存

### 丁、XCode配置

1、![](https://github.com/Quietly-20201113/PictureSpace/blob/main/flutter_fluwx/flutter-fluwx-ios-xcode.png?raw=true)



2、将域名写入 Domains，格式如下`applinks:你的域名`，不需要https

3、Xcode `Info`-->`URL Types` 展开点击加号 

> `Identifier`写入 `weixin`
>
> `URL Schemes`写入`微信开放平台移动应用中绑定的appID`

4、Xcode `Info`--> `Custom IOS Target Properties`-->`LSApplicationQueriesSchemes`张开添加item项

> 1、添加`value`为`weixin`的项
>
> 2、添加`value`为`weixinULAPI`的项

### 戊、flutter项目启动配置

```flutter
    await fluwx.registerWxApi(
        appId: "微信开放平台移动应用的AppId",
        doOnAndroid: true,
        doOnIOS: true,
        universalLink: "配置Xcode的域名（https：//xxxx.com/）"
```

### 己、微信开放平台配置

> 1、进入开放平台添加应用
>
> 2、申请接口信息`分享到朋友圈`、`发送给朋友`、`微信登录`等接口信息
>
> 3、修改`开发信息`,`IOS平台`、`IPad平台`的`Universal Links`配置，与flutter配置中`universalLink`一致

