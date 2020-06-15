---
layout: post
title:  "Flutter-使用Sentry上报异常搭建"
date:   2020-06-15 16:35
categories: App
tags: Flutter
---
* content
{:toc}
------

**当前教程只负责搭建Flutter模块**

##### 搭建前提

一、需要搭建Sentry服务[Sentry搭建教程](https://www.jianshu.com/p/ca4ad23a2dd6)

二、熟悉Flutter异步异常同步异常

****

##### 入坑注意事项

一、main方法中将所有初始化操作都包含在runZoned中[1.17.0版本以上推荐使用runZonedGuarded]

> 不包含在里面会导致runZoned[runZonedGuarded]失效(现目前测试是此原因)监听不到异常

二、区分异步异常与同步异常

> 具有async以及请求异常为异步异常,try/catch为同步异常

三、教程中Map对象中有\n符号,是为了在服务页面(html页面自动解析\n符号)显示容易解读

##### 着手搭建

一、异常上报数据

​	1、错误标题

​	2、错误详细位置

​	3、设备信息

​	4、用户信息

二、用到插件

​	1、sentry: ">=3.0.0 <4.0.0"；以官网为准[Flutter官方推荐使用样例](https://flutter.dev/docs/cookbook/maintenance/error-reporting)

​	2、device_info；获取设备信息，版本以插件库为准

三、异常分类处理

​	1、RangeError ；dart异常

​	2、FlutterErrorDetails；页面渲染异常

​	3、MissingPluginException； 服务异常

​	4、DioError ；Dio请求异常

> 异常都为自定义可由error.runtimeType.toString()断定为什么异常

四、异常处理

###### 枚举定义

```dart
enum ErrorExceptionEnum{
  /// dart异常
  RangeError,
  ///页面渲染异常
  FlutterErrorDetails,
   ///服务定义异常
  MissingPluginException,
  ///dio请求异常
  DioError
}
```

###### 请求错误定义

```dart
///异常类型与枚举对应
Map<String ,ErrorExceptionEnum> errorMap = {
  'RangeError' : ErrorExceptionEnum.RangeError,
  'FlutterErrorDetails' : ErrorExceptionEnum.FlutterErrorDetails,
  'MissingPluginException' : ErrorExceptionEnum.MissingPluginException,
  'DioError' : ErrorExceptionEnum.DioError,
};

class ErrorParsing{
  static dynamic errorStatus(dynamic error){
      ///判断是否是请求异常
      ///将请求接口地址获取
    if(errorMap[error.runtimeType.toString()] == ErrorExceptionEnum.DioError){
      return {
        '\n msgData' :error?.response?.data??'',
        '\n 请求域名' :error?.request?.baseUrl ??'',
        '\n 请求类型' : error?.request?.method??'',
        '\n 请求地址' : error?.request?.path??'',
        '\n 错误error' : '$error \n'
      };
    }
    return error;
  }
}
```

###### 获取设备信息

1、Android信息处理

```dart
////自己翻译结果  
Map<String, dynamic> _readAndroidBuildData(AndroidDeviceInfo build) {
    return <String, dynamic>{
      ' \n 最新补丁日期':'${build.version.securityPatch}',
      ' \n 操作系统名' : '${build.version.baseOS??'安卓'}',
      ' \n sdkInt':'${build.version.sdkInt}',
      '\n Android版本': '${build.version.release} ',
      '\n 手机品牌 ': '${build.brand} ',
      '\n 手机详细版本': '${build.model} ',
      '\n 外观设计名 ': '${build.device} ',
      '\n 版本号': '${build.display} ',
      '\n 当前手机唯一标识': '${build.fingerprint} ',
      '\n 内核(单词简写)': '${build.hardware} ',
      '\n 主机名 ': '${build.host} ',
      '\n id': '${build.id} ',
      '\n supported32BitAbis': '${build.supported32BitAbis} ',
      '\n supported64BitAbis': '${build.supported64BitAbis} ',
      '\n supportedAbis': '${build.supportedAbis} ',
      '\n 是否真机': '${build.isPhysicalDevice} \n',
    };
  }

///官方插件提供样例device_info
//  Map<String, dynamic> _readAndroidBuildData(AndroidDeviceInfo build) {
//    return <String, dynamic>{
//      'version.securityPatch': build.version.securityPatch,
//      'version.sdkInt': build.version.sdkInt,
//      'version.release': build.version.release,
//      'version.previewSdkInt': build.version.previewSdkInt,
//      'version.incremental': build.version.incremental,
//      'version.codename': build.version.codename,
//      'version.baseOS': build.version.baseOS,
//      'board': build.board,
//      'bootloader': build.bootloader,
//      'brand': build.brand,
//      'device': build.device,
//      'display': build.display,
//      'fingerprint': build.fingerprint,
//      'hardware': build.hardware,
//      'host': build.host,
//      'id': build.id,
//      'manufacturer': build.manufacturer,
//      'model': build.model,
//      'product': build.product,
//      'supported32BitAbis': build.supported32BitAbis,
//      'supported64BitAbis': build.supported64BitAbis,
//      'supportedAbis': build.supportedAbis,
//      'tags': build.tags,
//      'type': build.type,
//      'isPhysicalDevice': build.isPhysicalDevice,
//      'androidId': build.androidId,
//      'systemFeatures': build.systemFeatures,
//    };
//  }

```

2、IOS设备信息获取

```dart
  Map<String, dynamic> _readIosDeviceInfo(IosDeviceInfo data) {
    return <String, dynamic>{
      '\n 设备名': '${data.name} ',
      '\n 操作系统名': '${data.systemName} ',
      '\n 系统版本': '${data.systemVersion} ',
      '\n 设备型号': '${data.model} ',
      ' \n 设备名(本地)': '${data.localizedModel}',
      '\n 当前设备唯一值': '${data.identifierForVendor} ',
      '\n 是否真机': '${data.isPhysicalDevice} ',
      '\n 版本号': '${data.utsname.version} ',
      '\n 硬件类型': '${data.utsname.machine} \n',
    };
  }
///官方插件提供样例device_info
//  Map<String, dynamic> _readIosDeviceInfo(IosDeviceInfo data) {
//    return <String, dynamic>{
//      'name': data.name,
//      'systemName': data.systemName,
//      'systemVersion': data.systemVersion,
//      'model': data.model,
//      'localizedModel': data.localizedModel,
//      'identifierForVendor': data.identifierForVendor,
//      'isPhysicalDevice': data.isPhysicalDevice,
//      'utsname.sysname:': data.utsname.sysname,
//      'utsname.nodename:': data.utsname.nodename,
//      'utsname.release:': data.utsname.release,
//      'utsname.version:': data.utsname.version,
//      'utsname.machine:': data.utsname.machine,
//    };
//  }
```

3、获取设备信息

```dart
 final DeviceInfoPlugin deviceInfoPlugin = DeviceInfoPlugin();
Future<Map<String, dynamic>> initPlatformState() async {
    Map<String, dynamic> deviceData;
    try {
      if (Platform.isAndroid) {
        deviceData = _readAndroidBuildData(await deviceInfoPlugin.androidInfo);
      } else if (Platform.isIOS) {
        deviceData = _readIosDeviceInfo(await deviceInfoPlugin.iosInfo);
      }
    } on PlatformException {
      deviceData = <String, dynamic>{
        'Error:': 'Failed to get platform version.'
      };
    }
   return deviceData;
  }
```

###### 异常数据上报(可上报到Sentry或者进行接口上报)

```dart
 static Future<Null> reportError(dynamic error, dynamic stackTrace) async {
    print("异常收集 error = ${error.toString()}");
    print("error.type = ${error.runtimeType.toString()}");
    initPlatformState().then((res){
      Map<String,dynamic> _errMap = {
        '\n 错误类型' :error.runtimeType.toString(),
        '\n error' : ErrorParsing.errorStatus(error),
        '\n 设备信息' :res,
        '\n 用户信息' :'${App.currentUser.toString()} \n '
      };
      ...Sentry上报或者接口上报
    }).catchError((err){
      Map<String,dynamic> _errMap = {
        '\n 错误类型' :error.runtimeType.toString(),
        '\n error' : ErrorParsing.errorStatus(error),
        '\n 设备信息' :'获取设备信息出错',
        '\n 用户信息' : '${App.currentUser.toString()} \n '
      };
      ...Sentry上报或者接口上报
    });
  }
```

Sentry上报

```dart
import 'package:sentry/sentry.dart'
final String _dsn = 'Sentry服务模块地址';
final RcSentry.SentryClient _sentry = new RcSentry.SentryClient(dsn: _dsn);
```

##### 错误页面上报

> 定义错误友好页面
>
> 在main -> 首页中定义接收错误ErrorWidget.builder = getErrorWidget;

```dart
///捕获页面异常，建议布局分块布局，以免出现异常异常模块以下都不会显示
Widget getErrorWidget(FlutterErrorDetails error)  {
  print(
      "异常日志开始+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++");
  print("异常日志=$error");
  print(
      "异常日志结束--------------------------------------------------------------------");
   ///传入错误自动上传
  reportError(error, null);
  return Scaffold(
    body: Center(
        child: Text('错误页面')),
  );
}
```

##### 同步异常上报

```dart
try{
    ...
}catch(err,stack){
    reportError(err, stack);
}
```

##### 异步异常上报

> 只需要在方法上添加async可自行获取异常





