---
layout: post
title:  "Flutter学习之bloc+rxDart搭建"
date:   2020-09-03 18:00
categories: flutter
tags: Flutter
---
* content
{:toc}
------

#### 一、简单的数据处理模式

> ```dart
> 引入InheritedWidget提升性能
> ```

#### 二、源代码

```dart
import 'package:flutter/material.dart';
//Type _typeOf<T>() => T;
abstract class BlocBase {

  bool  _isNoData = false;
  bool  get isNoData => _isNoData;

  int get size => 20;

  Future getData({dynamic labelId,dynamic data,int page});

  Future onRefresh({dynamic labelId,dynamic data});

  Future onLoadMore({dynamic labelId,dynamic data});

  void dispose();
}

class BlocProvider<T extends BlocBase> extends StatefulWidget {
  BlocProvider({
    Key key,
    @required this.child,
    @required this.bloc,
    this.userDispose: true,
  }) : super(key: key);

  final T bloc;
  final Widget child;
  final bool userDispose;

  @override
  _BlocProviderState<T> createState() => _BlocProviderState<T>();

  static T of<T extends BlocBase>(BuildContext context){
    ///ancestorInheritedElementForWidgetOfExactType即将废弃 12.1版本
//    final type = _typeOf<_BlocProviderInherited<T>>();
//    _BlocProviderInherited<T> provider = context.ancestorInheritedElementForWidgetOfExactType(type)?.widget;
    _BlocProviderInherited<T> provider =context.getElementForInheritedWidgetOfExactType<_BlocProviderInherited<T>>()?.widget as _BlocProviderInherited<T>;
    return provider?.bloc;
  }

//  static T of<T extends BlocBase>(BuildContext context) {
//    final type = _typeOf<BlocProvider<T>>();
//    BlocProvider<T> provider = context.ancestorWidgetOfExactType(type);
//    return provider.bloc;
//  }


}

class _BlocProviderState<T extends BlocBase> extends State<BlocProvider<T>> {

  @override
  void dispose() {
    if (widget.userDispose) widget.bloc.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return _BlocProviderInherited<T>(
      bloc: widget.bloc,
      child: widget.child,
    );
    ///widget.child;
  }
}

///引入InheritedWidget提升性能
class _BlocProviderInherited<T> extends InheritedWidget{
  _BlocProviderInherited({
    Key key,
    @required Widget child,
    @required this.bloc,
  }) : super(key: key, child: child);

  final T bloc;
  @override
  bool updateShouldNotify(_BlocProviderInherited oldWidget) => false;
}

```

#### 三、可自行更改BlocBase文件

> 根据需求自行更改BlocBase文件内容,当前源码适用于列表请求

#### 四、配合rxdart使用

 	##### 1.页面数据渲染状态定义

```dart
enum RequestTypeEnum{
  ///初始化页面
  InitPage,
  ///空页面
  EmptyPage,
 ///成功页面
  SuccessPage,
  ///错误页面
  ErrorPage,
  ///无权限
  NoPermissionPage,
  ///无登录
  NoLogIn
}

/// 列表请求返回状态以及数据,控制页面显示
/// @param
/// @return
/// @author 
/// created at 2020/8/4 14:16
///
class RequestDataSource{
  ///唯一值
  final Object type;
  ///数据列表
  final List<dynamic> mapList;
  ///数据请求返回类型
  RequestTypeEnum sourceType;
  ///是否有下一页
  bool isMore;
  ///页码
  int page;

  RequestDataSource(this.type, this.mapList, [this.page = 0,this.sourceType = RequestTypeEnum.InitPage,this.isMore = false]);
}
```

##### 2.配合rxDart使用

```dart
  BehaviorSubject<RequestDataSource> _rentDataSource = BehaviorSubject<RequestDataSource>();
  Sink<RequestDataSource> get _rentDataSourceSink => _rentDataSource.sink;
  Stream<RequestDataSource> get rentDataSourceStream => _rentDataSource.stream.asBroadcastStream();
```

##### 3.请求后装载数据

```
RequestDataSource _dataSource = new new RequestDataSource(RequestTypeEnum.InitPage, []);
_rentDataSourceSink.add(_dataSource);
```

##### 4.ui界面使用

```dart
StreamBuilder(
              stream: rentDataSourceStream,
              builder: (BuildContext context,
                  AsyncSnapshot<RequestDataSource> snapshot) {
                RequestDataSource _dataSource = snapshot?.data?? new RequestDataSource(RequestTypeEnum.InitPage, []);
                  ///根据RequestTypeEnum枚举状态加载不同页面
                  return Container();
              })
```

