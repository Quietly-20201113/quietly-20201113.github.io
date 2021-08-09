---
layout: post
title:  "原生MVVM配合JS模板引擎渲染搭建"
date:   2021-08-09 09:02
categories: MVVM
tags: MVVM
---
* content
{:toc}
------

## MVVM升级实现功能 

### show -控制显示隐藏

### model- 数据双向绑定

### {{}}-动态绑定

### class -动态替换class

### html -动态添加html片段

### text- 动态渲染text内容

### on- 声明事件

### computed -计算属性

### $watch-内容副作用监听

### js模板引擎



#### 引入文件

> ```html
> <script src="@/public/vendor/mvvm/compile.js"></script>
>   <script src="@/public/vendor/mvvm/mvvm.js"></script>
>   <script src="@/public/vendor/mvvm/observer.js"></script>
>   <script src="@/public/vendor/mvvm/watcher.js"></script>
> ```
>
> 


#### v-show

> ```html
> <div id="show-features">
> 	<p v-show="isShow">是否是男的</p>
> 	<button v-on:click="clickTrigger">点击隐藏/显示</button>
> </div>
> ```
>
> ```javascript
> let show_vm = new MVVM({
>  el: '#show-features',
>  	data : {///data内容声明
>  		isShow : false
> 		 },
>      methods : {///methods声明方法
> 			clickTrigger : (e) => {
>              this.isShow = !this.isShow;///更新后会自动更改html进行处理
>          }
>      }
> });
> ```

#### v-model(目前实现input输入,其他后期添加)

> ```html
> <div id="model-features">
>  <input type="text" v-model="name" />
>  <p>{{name}}</p>
> </div>
> ```
>
> ```javascript
> let model_vm = new MVVM({
>  el : "#model-features",
>  data : {
>      name : '',//可自定义默认值
>  }
> });
> ///可实现input输入与value值绑定和{{}}内容同步,叫做双向绑定
> ```

#### {{}}

> ```html
> <div id="parantheses-features">
>       <h2>{{json.name}}</h2>
>       <h2>{{json.tel}}</h2>
>       <h2>{{json.sex}}</h2>
>       <h2>{{json.address}}</h2>
>  <button v-on:click="clickTrigger"> 点击更新json数据 </button>
> </div>
> ```
>
> ```javascript
> let parantheses_vm = new MVVM({
> el : "#parantheses-features",
> data : {
>  json : {
>      name : '丁平',
>      tel : 'xxxxx',
>      sex : '男',
>      address : '昆明'
>  }
> },
> methods : {
>    ///更新json对象数据,界面同步更新
>    clickTrigger : () => {
>        let _json = this.json;
>        _json.name = "丁不平";
>        _json.address = "曲靖";
>         this.json = Object.assign({},this.json,_json);///对象合并才能触发数据劫持更新html
>    }
> }
> });
> ```
>

#### text

> 与{{}}功能类似
>
> ```html
> <div>
> <p v-text="textVariable"></p>
> </div>
> ```
>
> 

#### class

> ```html
> <div id="class-features">
> <div v-class="classVariable">用户名称显示</div>
> <button v-on:click="clickTrigger">点击切换class属性</button>
> </div>
> ```
>
> ```css
> .name-color{
> color: red;
> font-size: 20px;
> 
> }
> .name{
> 	font-weight: 600;
> }
> .name-color-small{
> color: black;
> font-size: 11px;
> }
> ```
>
> ```java
> let class_vm = new MVVM({
> el : "#class-features",
> data : {
> classVariable : 'name-color',
> 
> },
> methods : {
>   clickTrigger : (e) => {
>       this.classVariable = "name-color-small";
>   }
> }
> });
> ```

#### html 

> ```html
> <div id="html-features">
>  <h4>
>      v-html可进行html装卸,可赋值htmlDOM也可赋值html字符串
>      可以配合js模板渲染操作,详情请看 js模板选人模块
>  </h4>
>  <div v-html="htmlDom"></div>
> </div>
> ```
>
> ```javascript
> let html_vm = new MVVM({
> el : "#html-features",
> data : {
> 	htmlDom : '<span>初始化暂无数据</span>',
> },
> });
> ///初始化请求装载数据
> _().then( res => {
>  let _res = res;
>  _res = "<span>数据加载成功</span>";
>  ///赋值之后自动更新html页面
>  html_vm.htmlDom = _res;
> });
> ```

#### on

> 格式 : v-on:事件
>
> 支持 : 
>
> `click`、`dblclick`、`mouseout`(鼠标移动触发系列)等等等等,
>
> 详情请看[JavaScript事件参考](https://www.w3school.com.cn/jsref/jsref_events.asp)
>
> ```html
> <div id="on-features">
>  <button v-on:click='clickTrigger'>点击事件声明</button>
>  <button v-on:dblclick="dblclickTrigger">双击事件声明</button>
>  <button v-on:mouseout="mouseoutTrigger">鼠标移出元素事件声明</button>
> </div>
> ```
>
> ```javascript
> let on_vm = new MVVM({
>  el : "#on-features",
>  methods : {
>       clickTrigger : () => {console.log('鼠标单击打印日志');},
>       dblclickTrigger : () => {console.log('鼠标双击打印日志');},
>       mouseoutTrigger : () => {console.log('鼠标移开打印日志');},
>  }
> });
> ```
>
> 

#### computed 

> 计算属性
>
> 使用场景 :`当页面中有某些数据依赖其他数据进行变动的时候,就可使用`
>
> ```html
> <div id="computed-features">
>  <input v-model="text" />
>  <input v-model="num" />
>  <h4>输入值相乘</h4>
>  <p>{{sum}}</p>
> </div>
> ```
>
> ```javascript
> let computed_vm = MVVM({
>  el : "#computed-features",
>  data : {
>      text : 0,
>      num : 0
>  },
>  computed : {
>      sum : () => {
>          return this.text * this.num;
>      }
>  }
> });
> ```
>
> 

#### $watch

>副作用监听
>
>使用场景 : `一般用于异步或者开销较大的操作`
>
>```javascript
>let vm_app = new MVVM({
>el : "xxx",
>data : {
>   json : {
>       name : "用户",
>       sex : "男"
>   }
>},
>});
>///实现监听数据
>vm_app.$watch('json', (newVale,oldVale) => {
>       console.log(`watch new word ：${newVale}`);
>       console.log(`watch old word ：${oldVale}`);
>});
>```

#### js模板渲染

> 使用场景
>
> 复杂渲染,列表,异步操作
>
> 配合v-html使用
>
> ```html
> <head>
>   <script id="interpolationtmpl" type="text/x-dot-template">
>          <div>Hi {{=it.name}}!</div>
>          <div>{{=it.age || ''}}</div>
>  </script>
> </head>
> 
> <body>
>  <div id="js-features">
>      <div v-html="htmlDom"></div>
>  </div>
> </body>
> ```
>
> ```javascript
> import doT from "../../../public/vendor/template/doT.js";
> let js_vm = new MVVM({
>  el : "#js-features",
>  data : {
>      htmlDom : '初始化加载'
>  }
> });
> 
> ///初始化数据请求
> _.then(res => {
>     let dataInter = {"name":"Jake","age":31};
>  ///获取模板
>    let interText = doT.template($("#interpolationtmpl").text());
>     js_vm.htmlDom = interText(dataInter);///给实例装载数据自动渲染给v-html
> });
> 
> ```
>
> 使用命令文档[官方文档](http://olado.github.io/doT/)
>
> 详细例子[博客园文档](https://www.cnblogs.com/inkwhite/p/9099025.html)

#### 使用规范

1、MVVM实例外部只能进行data数据更新，不能调用实例内部函数

2、v-class只能使用一个变量，多个class名请定义在实例data之中，赋值时进行字符串拼接

```javascript
let on_vm = new MVVM({
    el : "#on-features",
    data : {
        className : 'name'
    }
    methods : {
         clickTrigger : () => {
    		this.className = 'name color';
         },
        
    }
});
```



3、js模板引擎（dot）必须配合v-html使用，自行dom操作属于重复操作，MVVM实例化中以及将dom处理好，只需调用

4、MVVM模式观念可自行看vue.js

5、一个html页面可实例化多个MVVM，MVVM之间只能进行数据读取与复制，尽量一个html最多只实例化一个MVVM

6、数据初始化或者插件初始化不能在MVVM中使用，MVVM只提供用户操作事件实例化，数据初始化请求可在外层操作给实例化复制

7、目前事件声明还不能传参,但是能接受到默认参数`event`

8、公共处理数据函数需要定义在实例之外，实例内部可调用外部函数

9、外部公共函数需要给实例化data复制可进行如下操作

```javascript
let on_vm = new MVVM({
    el : "#on-features",
    data : {
        name : '名字'
    }
    methods : {
         clickTrigger : () => {
             publicClick(this);
         },
        
    }
});

const publicClick = (vm) => {
    vm.name = "名字更新了";
}
```

