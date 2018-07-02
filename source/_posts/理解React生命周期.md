---
title: 理解React生命周期
date: 2017-11-31 20:33:04
tags: React
image: ''
categories: 技术
---

# 理解React生命周期

<img src="https://www.lifeofpix.com/wp-content/uploads/2018/04/Glen-Etive-1600x1067.jpg" alt="" style="width:100%" />

<!-- more -->

React 组件的生命周期有三个主要场景：

- **装载（Mounting）**：组件被插入到 DOM 中；
- **更新（Updating）**：组件重新渲染以更新 DOM；
- **卸载（Unmounting）**：组件从 DOM 中移除。

不同的场景会调用不同的生命周期方法，包含 `will` 的方法在某个时间节点之前执行，包含 did 方法在某个时间节点之后执行。

## 初始渲染（装载）

**getDefaultProps**

```js
object getDefaultProps()
```

该方法在 **组件创建时（createClass）** 执行一次并缓存返回值。如果组件使用时未设置属性，就从缓存中读取默认属性。**注意：** ``getDefaultProps()`` 返回的缓存数据会在所有实例间共享。

>注意：
>``getDefaultProps`` 在任何实例创建之前执行，不在装载阶段执行，放在这里只是为了方便理解 React 组件初始化流程。

**getInitialState**

```js
object getInitialState()
```

组件装载之前执行一次，返回值用作 `this.state` 的初始值。

**componentWillMount**

```js
void componentWillMount()
````

初始渲染之前执行一次，前后端都有。如果在该方法中调用 ``setState``，``render()`` 将接收到更新后的数据，并且只会执行一次（即使状态已经改变）。


**componentDidMount**

```js
void componentDidMount()
```

初始渲染完成后立即执行一次，只在客户端执行（服务器端没有）。

子组件的 ``componentDidMount()`` 方法先于父组件之前执行，此时可以操作子组件的任何引用（如操作子组件 DOM）。

在此方法中可进行：

- 与其他 JavaScript 框架集成，如初始化 jQuery 插件；
- 使用 ``setTimeout/setInterval`` 设置定时器；
- 通过 Ajax/Fetch 获取数据；
- 绑定 DOM 事件；
- ……

## 更新

更新会在 React 组件初始渲染之后、卸载之前多次发生，属性、状态改变都会触发更新。

**componentWillReceiveProps**

```js
void componentWillReceiveProps(
  object nextProps, object nextContext
)
```

组件即将接收新属性之前执行，初始渲染不执行。

该方法用于比较当前属性（`this.props`）和新属性（`nextProps`），以便决定是否通过 `this.setState() `进行状态转换。此方法中调用 `this.setState()` 不会触发额外的渲染。

```js
componentWillReceiveProps: function(nextProps) {
  this.setState({
    likesIncreasing: nextProps.likeCount > this.props.likeCount
  });
}
```

>注意：
>一个常犯的错误是编写此方法内的代码时，假定属性已经改变。也就是说，即使属性没有改变，React 仍会在后续更新时执行此方法，所以在编写相关逻辑时，应该先判断属性是否发生改变。更多细节可[参见此文](https://reactjs.org/blog/2016/01/08/A-implies-B-does-not-imply-B-implies-A.html)。

**shouldComponentUpdate**

```js
boolean shouldComponentUpdate(
  object nextProps, object nextState, object nextContext
)
```
  
组件接收到新属性或状态时执行，初始渲染及调用 `forceUpdate` 时不执行。

通过比较 `this.props` 与 `nextProps` 及 `this.state` 与 `nextState`，如果确定新属性、状态无需更新组件，则可以返回 `false`。

```js
shouldComponentUpdate: function(nextProps, nextState) {
  return nextProps.id !== this.props.id;
}
```

如果 `shouldComponentUpdate` 返回 `false`，此次更新的将跳过 `render()`，此外，`componentWillUpdate` 和 `componentDidUpdate`也不会执行。

`shouldComponentUpdate` 默认始终返回`true`，以保证组件渲染与状态同步。

如果遇到性能瓶颈，尤其是有成百上千组件时，可以考虑使用 `shouldComponentUpdate` 优化应用。


**componentWillUpdate**

```js
void componentWillUpdate(
  object nextProps, object nextState, object nextContext
)
```

组件接收到新属性或状态即将重新渲染之前执行，初始渲染不执行。

>注意：
>不能在此方法里使用 `this.setState()`。如果需要更新状态以响应属性变化，使用 `componentWillReceiveProps` 替代之。

**componentDidUpdate**

```js
void componentDidUpdate(
  object prevProps, object prevState, object prevContext
)
```
组件更新后立即执行，初始渲染不执行。可用作操作发生变化 DOM 的时机。

##卸载

**componentWillUnmount**
```js
void componentWillUnmount()
```
组件即将从 DOM 中卸载之前执行，可在此进行定时器清除、事件解绑等清理工作。

