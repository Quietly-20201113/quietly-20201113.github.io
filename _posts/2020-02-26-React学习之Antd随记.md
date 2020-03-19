---
layout: post
title:  "React-ant-design使用记录"
date:   2020-02-26 20:56
categories: 前端ui
tags: React-antd
---
* content
{:toc}
------

**Tree使用**

1、动态加载Tree结构时，默认选中以及默认展开功能问题

> 动态数据默认选中能选中，但是默认展开功能会失效
>
> 解决方案 ： 

```react
/**
*@loop方法 官网中方法
*@this.props.selectedKeys 默认选中数据  string[]  
*@this.props.expandedKeys 默认展开数据 string[]
*/
render(){
    return(
        <div>
         	{
                this.props.treeData && this.props.treeData.length > 0 ? 
                <Tree className="draggable-tree"
             		 defaultSelectedKeys={this.props.selectedKeys}
            		 defaultExpandedKeys={this.props.expandedKeys}
           			 blockNode>
              {loop(this.props.treeData)}
          </Tree > : '暂无数据'
            }   
         <div>
    )
}
```

**dva使用详解**

> ​	需要在 config.js文件进行开启配置

```react
plugins :[
    ['umi-plugins-react',{
        dva : true
    }]
]
```

在umi中，约定在src/models文件夹中定义model进行数据处理交互

> 以代码为例，使用dva进行分层

1、page页

```react
import React from 'react'
import { connect } from 'dva'

//model层[namespace]唯一值，提供给page页面找到指定的model数据
const namespace = 'spanModel';

/**
* 第一种写法
* [connect中参数解析]
* @val 作用：将page层与model层进行连接，并将数据绑定到this.props
* @dispatch 作用：调用model层effects中方法
*/
@connect((val) => {
   return {
    spanText : val[namespace].spanText,//spanText是mnodel中定义的值
   }
},(dispatch) =>{
   return {
     OnChange : (val) =>{
      dispatch({
        type : `${namespace}/OnChange`,//调用namespace =spanModel的model中OnChange方法
        payload : val //传值
     });
     }
   }
})
class DvaPage extends React.Component{
    render(){
        return(
            <div>
                <span>{this.props.spanText}</span>
                <button onClick = {this.props.OnChange.bind('嘿嘿')}>点击更改值</button>
            </div>
            //.bind("嘿嘿"),自定义方法传值方式之一
        )
    }
}



/* 
* 第二种写法
* 适用于添加了hooks
*/
const dvaState = ()=>{
    return {
    spanText : val[namespace].spanText,
   }
}
const DvaPage =(props) =>{
   const  OnChange = (val) =>{
     const { dispatch } = props;
      dispatch({
        type : `${namespace}/OnChange`,
        payload : val
     });
   }
    return(
        <div>
                <span>{this.props.spanText}</span>
                <button onClick = {this.props.OnChange.bind('嘿嘿')}>点击更改值</button>
          </div>
    )
}
export default connect(dvaState)(DvaPage)

```

2、model页面

```react
export default {
    namespace: 'spanText',
    state: {
        spanText: '',//树结构数组
    },
    reducers: {
        /**
        * @odlstate 当前state
        * @res effects中put过来的值
        */
      setSpanText: (odlstate, res) => {
            if (res.data) { 
                //给state中的spanText更新值
                //也可以直接返回res.data，会自动复制（不推荐）
                return { ...odlstate, spanText: res.data}; 
            }
        }
    },
    effects: {
        /**
        * 写法一
        * @payload page页面传入的参数
        * @call 请求使用
        * @put 提交数据到reducers进行数据更新
        */
        OnChange({ payload }, { call, put }) {
            yield put({
                type: 'setSpanText',
                data: payload,
            }); // Lo
        },
        //写法二
        //带星号是属于异步进行数据操作：如请求操作
        *OnChange({ payload }, { call, put }){}

    }

}
```

**hooks用法**

> 还不熟练  简单写写以后完善

```react
import React ,{ useState } from 'react'
import { connect } from 'dva'
//不能适用类的写法
const namespace = 'spanModel';
const DvaPage =(props) =>{
    /**
    * @text 初始值
    * @setText 更改值得方法
    */
    const {text,setText} = useState('嘿嘿'); //可以规定值得类型 useState<string>('嘿嘿')
    return(
        <div>
               <span>{`显示这个文字${text}`}</span>
               <button onClick={() => setText('哈哈')}>更改值</button>
         </div>
    )
}
export default connect(null)(DvaPage)



```

**react-redux运用**

1、树结构详解（切勿多次定义属性名。以免树结构层次变深）

>reducers中返回的命名是state树最底下的名字

```react

//reducers.js
import * as Audit from './auditTypes';

//treeData 最终取值属性名
//treeReducer 可在合并时更改
const treeReducer = (state = [], action) => {
    switch(action.type) {
      case Audit.AUDIT_TREE:
        return {
          ...state, treeData: action.treeData
        }
      default:
        return state
    }
  }
  
  export default treeReducer;
```

```react
import { combineReducers } from 'redux';
import treeReducer from './auditReducers';
//treeReducer在此处还能更改树结构（合并处）
const auditReducers = combineReducers({
    treeReducer,
    //treeData : treeReducer 将treeReducer重命名为treeData
  })
  
  export default auditReducers;
```

> 合并后最上层属性为合并时定义的属性名；如：auditReducers
>
> 当取值时取值依次为：树结构.合并名.单个reducers名.定义名；
>
> 如：

```react
//业务.js(取值操作)
const mapStateToProps = (state, ownProps) => {
  return {
    treeData: state.auditReducers.treeReducer.treeData
  }
}
```

**react对函数式组件优化**

