---
title: 关于传送门React-Portal
date: 2018-01-30 22:56:06
categories:
  - Javascript
tags: 
  -Javascript
  -React
---

## 什么是Portal

在React v16.0的更新中，React给我们带来了一个新特性，那就是Portals，这个单词有一个非常形象的翻译 --- "传送门"。

如果大家看过叮当猫，相信对它的一个道具印象非常深刻，那就是任意门。叮当猫的任意门就是一个传送门，当人们钻进这个门，可以从指定的另一个地方出来。

## 官方解释

> Portals provide a first-class way to render children into a DOM node that exists outside the DOM hierarchy of the parent component.

## 用法

    
    render() {
      // The first argument (child) is any renderable React child, such as an element, 
         string, or fragment. The second argument (container) is a DOM element.  
      return ReactDOM.createPortal(
        this.props.children,
        domNode,
      );
    }

## 为什么我们需要Portal

在使用React的时候，如果我们需要在某个组件中使用Dialog，大致代码如下:

    render() {
      return (
        <div className="dialog-element-parent">
          {components}
          <Dialog />
        </div>
      );
    }

一般来说，我们希望Dialog显示在屏幕正中央，但是如果`div.dialog-element-parent`仅仅只是一个小窗口，那我们实现起来就比较麻烦了。

如果我们的项目需要的各类Dialog样式相对较统一，我们可以将Dialog放到组件树的最顶层，然后通过redux等组件间通信方式，来给Dialog发送信息去展示内容。但是这样Dialog的内容无法支持完全定制。

所以我们需要一个Portal来实现我们想要的效果:

- 声明式的写在一个组件中

- 并不真正render在被声明的地方

<!-- more -->

## 在React v16.0之前，如何实现Portal

需要用到ReactDom中的两个函数

- unstable_renderSubtreeIntoContainer

- unmountComponentAtNode


    export default Class DialogWrap extends React.Component {
      componentDidMount() {
        const container = document.createElement('div');
        document.body.appendChild(container);
        this.container = container;
        this.renderDialog(this.props);
      }
      componentDidUpdate() {
        this.renderDialog(this.props);
      }
      componentWillUnmount() {
        if (this.container) {
          ReactDOM.unmountComponentAtNode(this.container);
          this.container = null;
        }
      }
      renderDialog(props) {
        ReactDOM.unstable_renderSubtreeIntoContainer(
          this, 
          <Dialog {...props} />,   // 组件
          this.container,   // 需要渲染<Dialog />的Dom node
        );
      }
      render() {
        return null;
      }
    }
    
    
> 注意，在最新的React v16.0中，这两个函数还能使用。但是推荐替换为createPortal.
    
## React v16.0的实现方式

    export default Class DialogWrap extends React.Component {
      constructor(props) {
        super(props);
        const container = document.createElement('div');
        document.body.appendChild(container);
        this.container = container;
      }
      componentWillUnmount() {
        if (this.container) {
          this.container.parentNode.removeChild(this.container); 
          this.container = null;
        }
      }
      render() {
        return (ReactDOM.createPortal(<Dialog {...props} />, this.container));
      }
    }
    
## React Portal的事件冒泡

React v16.0通过createPortal创建出来的组件，它的事件可以传递到其React组件对应的父组件上，而不是真实绑定的Dom node上。通过*unstable_renderSubtreeIntoContainer*创建出来的组件无法实现该特性的。

## 总结

Portals的典型用法就是当父节点存在 *overflow: hidden* 或者 *z-index* 属性的时候，允许子节点打破这些限制。例如Dialog, Modal, tooltips等组件.

> 文章内容参考自:
>
> [React官网 - Portals](https://reactjs.org/docs/portals.html)
>
> [知乎 - 《传送门: React Portal》](https://zhuanlan.zhihu.com/p/29880992)
>
> [react-component - DialogWrap.tsx](https://github.com/react-component/m-dialog/blob/master/src/DialogWrap.tsx)