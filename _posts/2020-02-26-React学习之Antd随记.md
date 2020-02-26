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

