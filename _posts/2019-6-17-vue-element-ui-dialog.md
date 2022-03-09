---
layout: post
title: "vue-dialog弹框拖动"
date: 2019-06-17 17:56
categories: vue element-ui
tags: element-ui vue
---

---

**element-ui 中 Dialog 弹框拖动**

[自己码的源码](https://github.com/Quietly-20201113/element-el-dragDialog.git)

> 借用网上样例，网上样例不支持在浏览器可见范围内拖动，在结合[vue-element-admin](https://panjiachen.github.io/vue-element-admin-site/zh/)中弹框拖动的模块合并更改的。
>
> > 问题：与容器的 position: absolute 冲突

```js
export default {
  bind(el, binding, vnode) {
    //element-ui弹框拖动效果：双击表头全屏，可拖动（区域限制在可视化区域），可拉取宽高
    const dialogHeaderEl = el.querySelector(".el-dialog__header");
    const dragDom = el.querySelector(".el-dialog");
    dialogHeaderEl.style.cssText += ";cursor:move;";
    dragDom.style.cssText += ";top:0px;";
    dialogHeaderEl.onselectstart = new Function("return false");
    let minWidth = 400;
    let minHeight = 300;
    //初始非全屏
    let isFullScreen = false;
    //当前宽高
    let nowWidth = 0;
    let nowHight = 0;
    //当前顶部高度
    let nowMarginTop = 0;
    // 获取原有属性 ie dom元素.currentStyle 火狐谷歌 window.getComputedStyle(dom元素, null);
    const getStyle = (function () {
      if (window.document.currentStyle) {
        return (dom, attr) => dom.currentStyle[attr];
      } else {
        return (dom, attr) => getComputedStyle(dom, false)[attr];
      }
    })();
    let moveDown = (e) => {
      // 鼠标按下，计算当前元素距离可视区的距离
      const disX = e.clientX - dialogHeaderEl.offsetLeft;
      const disY = e.clientY - dialogHeaderEl.offsetTop;
      const dragDomWidth = dragDom.offsetWidth;
      const dragDomHeight = dragDom.offsetHeight;

      const screenWidth = document.body.clientWidth;
      const screenHeight = document.body.clientHeight;
      console.log(document);
      const minDragDomLeft = dragDom.offsetLeft;
      const maxDragDomLeft = screenWidth - dragDom.offsetLeft - dragDomWidth;

      const minDragDomTop = dragDom.offsetTop;
      const maxDragDomTop = screenHeight - dragDom.offsetTop - dragDomHeight;
      // 获取到的值带px 正则匹配替换
      let styL = getStyle(dragDom, "left");
      let styT = getStyle(dragDom, "top");
      if (styL.includes("%")) {
        styL = +document.body.clientWidth * (+styL.replace(/\%/g, "") / 100);
        styT = +document.body.clientHeight * (+styT.replace(/\%/g, "") / 100);
      } else {
        styL = +styL.replace(/\px/g, "");
        styT = +styT.replace(/\px/g, "");
      }
      document.onmousemove = function (e) {
        // 通过事件委托，计算移动的距离
        let left = e.clientX - disX;
        let top = e.clientY - disY;
        //---------边界处理（可视化区域）----------------------------------------------------------------------------------------------------------//
        if (-left > minDragDomLeft) {
          left = -minDragDomLeft;
        } else if (left > maxDragDomLeft) {
          left = maxDragDomLeft;
        }
        if (-top > minDragDomTop) {
          top = -minDragDomTop;
        } else if (top > maxDragDomTop) {
          top = maxDragDomTop;
        }
        //--------------------------------------------------------------------------------------------------------------------------------------------//
        // 移动当前元素
        dragDom.style.cssText += `;left:${left + styL}px;top:${top + styT}px;`;
        // emit onDrag event
        vnode.child.$emit("dragDialog");
      };
      document.onmouseup = function (e) {
        document.onmousemove = null;
        document.onmouseup = null;
      };
    };
    dialogHeaderEl.onmousedown = moveDown;
    dialogHeaderEl.ondblclick = (e) => {
      if (isFullScreen == false) {
        nowHight = dragDom.clientHeight;
        nowWidth = dragDom.clientWidth;
        nowMarginTop = dragDom.style.marginTop;
        dragDom.style.left = 0;
        dragDom.style.top = 0;
        dragDom.style.height = "100VH";
        dragDom.style.width = "100VW";
        dragDom.style.marginTop = 0;
        isFullScreen = true;
        dialogHeaderEl.style.cursor = "initial";
        dialogHeaderEl.onmousedown = null;
      } else {
        dragDom.style.height = "auto";
        dragDom.style.width = nowWidth + "px";
        dragDom.style.marginTop = nowMarginTop;
        isFullScreen = false;
        dialogHeaderEl.style.cursor = "move";
        dialogHeaderEl.onmousedown = moveDown;
      }
    };
    dragDom.onmousemove = function (e) {
      let moveE = e;
      if (
        e.clientX > dragDom.offsetLeft + dragDom.clientWidth - 10 ||
        dragDom.offsetLeft + 10 > e.clientX
      ) {
        dragDom.style.cursor = "w-resize";
      } else if (
        el.scrollTop + e.clientY >
        dragDom.offsetTop + dragDom.clientHeight - 10
      ) {
        dragDom.style.cursor = "s-resize";
      } else {
        dragDom.style.cursor = "default";
        dragDom.onmousedown = null;
      }
      dragDom.onmousedown = (e) => {
        const clientX = e.clientX;
        const clientY =
          e.clientY > document.documentElement.clientHeight
            ? document.documentElement.clientHeight
            : e.clientY;
        let elW = dragDom.clientWidth;
        let elH = dragDom.clientHeight;
        let EloffsetLeft = dragDom.offsetLeft;
        let EloffsetTop = dragDom.offsetTop;
        // dragDom.style.userSelect = 'none';
        let ELscrollTop = el.scrollTop;
        //判断点击的位置是不是为头部
        if (
          clientX > EloffsetLeft &&
          clientX < EloffsetLeft + elW &&
          clientY > EloffsetTop &&
          clientY < EloffsetTop + 100
        ) {
          //如果是头部在此就不做任何动作，以上有绑定dialogHeaderEl.onmousedown = moveDown;
        } else {
          document.onmousemove = function (e) {
            // e.preventDefault(); // 移动时禁用默认事件
            //左侧鼠标拖拽位置
            if (clientX > EloffsetLeft && clientX < EloffsetLeft + 10) {
              //往左拖拽
              if (clientX > e.clientX) {
                dragDom.style.width = elW + (clientX - e.clientX) * 2 + "px";
              }
              //往右拖拽
              if (clientX < e.clientX) {
                if (dragDom.clientWidth < minWidth) {
                } else {
                  dragDom.style.width = elW - (e.clientX - clientX) * 2 + "px";
                }
              }
            }
            //右侧鼠标拖拽位置
            if (
              clientX > EloffsetLeft + elW - 10 &&
              clientX < EloffsetLeft + elW
            ) {
              //往左拖拽
              if (clientX > e.clientX) {
                if (dragDom.clientWidth < minWidth) {
                } else {
                  dragDom.style.width = elW - (clientX - e.clientX) * 2 + "px";
                }
              }
              //往右拖拽
              if (clientX < e.clientX) {
                dragDom.style.width = elW + (e.clientX - clientX) * 2 + "px";
              }
            }
            //底部鼠标拖拽位置
            if (
              ELscrollTop + clientY > EloffsetTop + elH - 20 &&
              ELscrollTop + clientY < EloffsetTop + elH
            ) {
              //往上拖拽
              if (clientY > e.clientY) {
                if (dragDom.clientHeight < minHeight) {
                } else {
                  dragDom.style.height = elH - (clientY - e.clientY) * 2 + "px";
                }
              }
              //往下拖拽
              if (clientY < e.clientY) {
                dragDom.style.height = elH + (e.clientY - clientY) * 2 + "px";
              }
            }
          };
          //拉伸结束
          document.onmouseup = function (e) {
            document.onmousemove = null;
            document.onmouseup = null;
          };
        }
      };
    };
  },
};
```

---

```JS
import drag from './drag'
//用法，需要使用页面路径引入：import elDragDialog from '@/directive/el-dragDialog'
//页面注册命令：  directives: { elDragDialog }
//模块化单独引用，禁止改动
//不全局引用
const install = function(Vue) {
  Vue.directive('el-drag-dialog', drag)
}

if (window.Vue) {
  window['el-drag-dialog'] = drag
  Vue.use(install); // eslint-disable-line
}
drag.install = install
export default drag

```
