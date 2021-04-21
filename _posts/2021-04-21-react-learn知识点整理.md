---
layout: post
title:  "react-learn知识点学习"
date:   2021-04-21 20:30
categories: react

tags: react 面试
---

------



## React ##

###### 1、什么时候使用状态管理器?





###### 2、render函数中return如果没有使用()会有什么问题？

[答案借鉴地址](https://github.com/haizlin/fe-interview/issues/952)

> 我们在使用JSX语法书写react代码时，babel会将JSX语法编译成js，同时会在每行自动添加**分号**（；），如果`return`后换行了，那么就会变成 `return；` 一般情况下会报错：Nothing was returned from render. This usually means a return statement is missing. Or, to render nothing, return null.译文 : 渲染没有返回任何内容。这通常意味着缺少return语句。或者，为了不渲染，返回null
>
> 举两个正确的书写例子：
>
> ``` react
> const Nav = () => {
>   return (
>     <nav className="c_navbar">
>       { some jsx magic here }
>     </nav>
>   )
> }
> 
> const Nav = () => {
>  return <nav className="c_navbar">
>     { some jsx magic here }
>   </nav>
> }
> ```
>
> 错误的写法：
>
> ```react
> const Nav = () => {
>   return
>     <nav className="c_navbar">
>       { some jsx magic here }
>     </nav>
> }
> ```



###### 3、componentWillUpdate可以直接修改state的值吗？

[答案借鉴地址](https://github.com/haizlin/fe-interview/issues/951)

> 首先你能这么做，但是不建议，因为会造成无限渲染
>
> 并不是说任何情况下不能在此生命周期方法中使用this.setState()，你可以设置合理的条件，保证它不会在每次渲染的时候执行this.setState()，而只是在需要的情况下执行【这很重要】
>
> 不止componentWillUpdate方法，比如componentWillReceiveProps也可以这样设置条件

###### 4、说说你对React的渲染原理的理解

[答案借鉴地址](https://github.com/haizlin/fe-interview/issues/950)

>从数据方面 : 单向数据流。React是一个MVVM框架，简单来说是在MVC的模式下在前端部分拆分出数据层和视图层。单向数据流指的是只能由数据层的变化去影响视图层的变化，而不能反过来（除非双向绑定）;数据驱动视图。我们无需关注页面的DOM，只需要关注数据即可.
>
>从渲染方面 : 首先编写的jsx会被转换执行返回node对象，node对象就可以理解为是虚拟dom，渲染时都会将老的虚拟dom和新的虚拟dom来进行比对，比对的过程中就涉及到了diff算法，只比较同一层的节点，节点tag不同，就不再比较，以新的节点为准，节点相同就比较key，key不同也会以新节点为准；之后就将最终的虚拟dom，再渲染在浏览器中；
>
>通过props state与一些高阶组件执行更新

###### 5、什么渲染劫持？

[答案借鉴地址](https://github.com/haizlin/fe-interview/issues/949)

> 组合渲染和条件渲染都是渲染劫持的一种，通过反向继承，不仅可以实现以上两点，还可以增强由原组件render函数产生的React元素。
>
> 实际的操作中 通过 操作 state、props 都可以实现渲染劫持
>
> **条件渲染**
>
> ```react
> const HOC = (WrappedComponent) =>
>   class extends WrappedComponent {
>     render() {
>       if (this.props.isRender) {
>         return super.render();
>       } else {
>         return <div>暂无数据</div>;
>       }
>     }
>   }
> ```
>
> **组合渲染**
>
> ```react
> // 例子来源于《深入React技术栈》
> function HigherOrderComponent(WrappedComponent) {
>   return class extends WrappedComponent {
>     render() {
>       const tree = super.render();
>       const newProps = {};
>       if (tree && tree.type === 'input') {
>         newProps.value = 'something here';
>       }
>       const props = {
>         ...tree.props,
>         ...newProps,
>       };
>       const newTree = React.cloneElement(tree, props, tree.props.children);
>       return newTree;
>     }
>   };
> }
> 
> ```




###### 6、React Intl是什么原理？

>实现原理和react-redux的实现原理类似，最外层包一个Provider，利用getChildContext，将intlConfigPropTypes存起来，在FormattedMessage、FormattedNumber等组件或者调用injectIntl生成的高阶组件中使用，来完成国际化的。
>
>使用过程也非常简单，和react-redux的使用方法非常类似。

###### 7、说说Context有哪些属性？

[答案借鉴地址](https://github.com/haizlin/fe-interview/issues/945)

> context属于一种解决组件间层级过多传递数据的问题，避免了层层嵌套的通过props传递的形式，同时对于不需要使用到redux时，是一种解决方案，关于组件的复用性变差的问题，我觉得是可以通过高阶组件和context配合来解决的，因为react-redux使用的就是这样的形式；
>
> 主要的形式：createContext(value)：创建一个context实例；其中的参数为当前数据的默认值，只有没在Provider中指定value时，才会生效；
> Context.Provider：生产者，数据提供方；通过value属性来定义需要被传递的数据
> Context.Consumer：消费者，数据获取方；根据是函数组件还是class组件，有不同的使用形式；class组件可以指定contextType来确定要使用哪一个context对象的值，函数组件需要使用回调函数的形式来获取context的值；需要显示的指定context对象；

###### 8、为什么React并不推荐我们优先考虑使用Context？

[答案借鉴地址](https://github.com/haizlin/fe-interview/issues/943)

> 1、Context目前还处于实验阶段，可能会在后面的发行版本中有很大的变化，事实上这种情况已经发生了，所以为了避免给今后升级带来大的影响和麻烦，不建议在app中使用context。
>
> 2、尽管不建议在app中使用context，但是独有组件而言，由于影响范围小于app，如果可以做到高内聚，不破坏组件树之间的依赖关系，可以考虑使用context
> 3、对于组件之间的数据通信或者状态管理，有效使用props或者state解决，然后再考虑使用第三方的成熟库进行解决，以上的方法都不是最佳的方案的时候，在考虑context。
> 4、context的更新需要通过setState()触发，但是这并不是很可靠的，Context支持跨组件的访问，但是如果中间的子组件通过一些方法不影响更新，比如 shouldComponentUpdate() 返回false 那么不能保证Context的更新一定可以使用Context的子组件，因此，Context的可靠性需要关注。

###### 9、childContextTypes是什么？它有什么用？

> childContextTypes用来定义context数据类型，该api从16.3开始已被废弃
>
> ```react
> class MessageList extends React.Component {
>   getChildContext() {
>     return {color: "purple"};
>   }
> 
>   render() {
>     return <div>MessageList</div>;
>   }
> }
> 
> MessageList.childContextTypes = {
>   color: PropTypes.string
> };
> ```
>
> hooks有此用法
>
> ```react
> import PropTypes from 'prop-types';
> const MessageList = ({
>     color
> }) => {
>     MessageList.propTypes={
>          color: PropTypes.string
>     }
> }
> ```

###### 10、contextType是什么？它有什么用？

> 类似于hooks中 useContext用法

###### 11、Consumer向上找不到Provider的时候怎么办？

> The defaultValue argument is only used when a component does not have a matching Provider above it in the tree. This can be helpful for testing components in isolation without wrapping them. Note: passing undefined as a Provider value does not cause consuming components to use defaultValue.
>
> 官方解释 : 只有在树中组件上方没有匹配的Provider时，才使用defaultValue参数。 这对于隔离测试组件而不进行包装很有帮助。 注意：将undefined传递为Provider值不会导致使用组件使用defaultValue。

###### 12、说说你对windowing的了解

> Windowing 是一种技术，它在任何给定时间只呈现一小部分行，并且可以显著减少重新呈现组件所需的时间以及创建的 DOM 节点的数量。如果应用程序呈现长的数据列表，则建议使用此技术。react-window 和 react-virtualized 都是常用的 windowing 库，它提供了几个可重用的组件，用于显示列表、网格和表格数据。

###### 13、React的严格模式有什么用处？

> [识别不安全的生命周期](https://zh-hans.reactjs.org/docs/strict-mode.html#identifying-unsafe-lifecycles)
>
> [关于使用过时字符串 ref API 的警告](https://zh-hans.reactjs.org/docs/strict-mode.html#warning-about-legacy-string-ref-api-usage)
>
> [关于使用废弃的 findDOMNode 方法的警告](https://zh-hans.reactjs.org/docs/strict-mode.html#warning-about-deprecated-finddomnode-usage)
>
> [检测意外的副作用](https://zh-hans.reactjs.org/docs/strict-mode.html#detecting-unexpected-side-effects)
>
> [检测过时的 context API](https://zh-hans.reactjs.org/docs/strict-mode.html#detecting-legacy-context-api)

###### 14、React组件的构造函数有什么作用？

> 1、指定this --> super(props)
>
> 2、设置初始化的状态 --> this.setState({});
>
> 3、为组件上的构造函数绑定this

###### 15、React中在哪捕获错误

> ​	定义错误边界当做常规组件包裹ui组件
>
> 官网例子 : 
>
> ```react
> class ErrorBoundary extends React.Component {
>   constructor(props) {
>     super(props);
>     this.state = { hasError: false };
>   }
> 
>   static getDerivedStateFromError(error) {
>     // 更新 state 使下一次渲染能够显示降级后的 UI
>     return { hasError: true };
>   }
> 
>   componentDidCatch(error, errorInfo) {
>     // 你同样可以将错误日志上报给服务器
>     logErrorToMyService(error, errorInfo);
>   }
> 
>   render() {
>     if (this.state.hasError) {
>       // 你可以自定义降级后的 UI 并渲染
>       return <h1>Something went wrong.</h1>;
>     }
> 
>     return this.props.children; 
>   }
> }
> ```
>
> 使用 
>
> ```react 
> <ErrorBoundary>
>   <MyWidget />
> </ErrorBoundary>
> ```
>
> 但是错误边界不会捕获 : 
>
> ```react
> try{}catch(err){}
> ///异步代码（例如 setTimeout 或 requestAnimationFrame 回调函数）
> ///服务端渲染
> ///它自身抛出来的错误（并非它的子组件)
> ```
>
> 

###### 16、React怎样引入svg的文件？

> 代码式 : 
>
> ```react
> const HouseTabsMenuKeyGreySvg = () => (
>     <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 14 14">
>         <g transform="translate(-218 -71)">
>             <circle cx="7" cy="7" r="7" transform="translate(218 71)" fill='#f2f2f2'/>
>             <path fill='#ddd'
>                   d="M231.89,95.73l.111-.11a.319.319,0,0,0,0-.452l-.816-.814a.322.322,0,0,0-.454,0l-1.471,1.468a.889.889,0,0,0,.011,1.26l.01.009.009.011a.893.893,0,0,0,1.259.011l.168-.168.289.288a.322.322,0,1,0,.455-.454l-.289-.289.283-.282.289.288a.322.322,0,1,0,.455-.454Z"
>                   transform="translate(-7.501 -15.86)"/>
>             <path fill='#ddd'
>                   d="M239.288,82.761A2.6,2.6,0,0,0,237.473,82a2.5,2.5,0,0,0-2.536,2.53,2.608,2.608,0,0,0,2.58,2.573,2.5,2.5,0,0,0,2.536-2.53,2.585,2.585,0,0,0-.764-1.811Z"
>                   transform="translate(-11.551 -7.5)"/>
>             <path fill='#ccc'
>                   d="M241.782,87.954a.326.326,0,0,1-.46,0,.3.3,0,1,0-.186.505.331.331,0,0,1,.349.3.318.318,0,0,1-.3.343.964.964,0,0,1-.737-.281.939.939,0,1,1,1.329-1.326.324.324,0,0,1,0,.459Z"
>                   transform="translate(-15.114 -11.055)"/>
>         </g>
>     </svg>
> );
> ```
>
> 文件式 : 
>
> ```react
> <img src='path/logo.svg' className="App-logo" alt="logo" />
> ```

###### 17、react中的const

> 讲讲 es6中的const
>
> ```react
> const framework = "weex";
> framework = "react native";
> //TypeError: Assignment to constant variable.
> 
> const weex = {"weexpack": "good", "weex-toolkit": "sucks"};
>  weex["weex-toolkit"] = "good";
> //允许
> 
> 
> ```
>
> 在 ES6 中，const 只限制对应的变量不能够被再次赋值
>
> 但是变量的属性可以被再次赋值
>
> react中声明函数,useState都用到const

