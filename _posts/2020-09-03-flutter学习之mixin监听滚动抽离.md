---
layout: post
title:  "flutter学习之mixin抽离监听滚动"
date:   2020-09-03 15:56
categories: Flutter
tags: Flutter
---

------

**做记录使用大概没得啥博客价值,直接丢代码**

```dart
import 'package:common_utils/common_utils.dart';
import 'package:flutter/cupertino.dart';

/// 提出滚动组件ScrollController滚动计算出显示与隐藏
/// @param 
/// @return 
/// @author 
/// created at 2020/8/19 15:21
///
@optionalTypeArgs
mixin ScrollCalculatedAttributesClientMixin{

  ///滚动最大值做出反应
  @protected
  double get scrollOffsetMax;

  ///滚动为负数时是否停止一切操作
  @protected
  bool  get OffsetNegative;

  ///滚动旧数据
  double _oldScrollOffset = 0.0;
  double get oldScrollOffset => _oldScrollOffset;

  bool _isStay = true;

///
/// @param offset [实时滚动高度]
/// @param dynamicHeight [动态高度,需手动传入] | 可选参数
/// @return  true | false 根据返回值更新数据
/// @author 
/// created at 2020/8/19 11:43
///
  bool setScrollReaction(double offset,[double dynamicHeight]){
    double _scrollOffsetMax = scrollOffsetMax;
    bool _OffsetNegative = OffsetNegative;
//    print("滚动高度 = $offset");
//    print("历史高度 = $_oldScrollOffset");
    if(ObjectUtil.isEmpty(scrollOffsetMax)) _scrollOffsetMax = 20.0;
    if(ObjectUtil.isNotEmpty(dynamicHeight)) _scrollOffsetMax = dynamicHeight;
    if(ObjectUtil.isEmpty(OffsetNegative)) _OffsetNegative = true;
    ///判断 负数后停止操作 & 滚动高度为负数
    if(_OffsetNegative && offset.isNegative){
      _isStay = true;
      _oldScrollOffset = 0.0;
    }else{
      if(offset - _oldScrollOffset >= _scrollOffsetMax){
        _oldScrollOffset = offset;
        _isStay = false;
      }
      if(_oldScrollOffset - offset  >= _scrollOffsetMax){
        _isStay = true;
        _oldScrollOffset = offset;
      }
    }
    return _isStay;
  }

}


```

#### 注意事项

> 使用ScrollController控制器,放在需要滚动的组件上
>
> 在initState里面监听
>
> ```dart
> scrollController.addListener(() {
> //     _scrollController.position.pixels 获取当前滚动部件滚动的距离
> //     当滚动距离大于 单位一屏之后，显示回到顶部按钮
> //scrollController.position.pixels(滚动距离)
> });
> ```
>
> 如果滚动的最大距离(scrollOffsetMax)是变量的 则需要手动传入,常量就不需要了

## 记录原由 

首次发现mixin挺好用的写了个样例

之后逐渐发现使用mixin结合rxdart做数据请求也挺好用的(简单页面仅仅单次请求)