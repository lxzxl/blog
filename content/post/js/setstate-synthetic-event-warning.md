---
title: "setState的一个Synthetic Event Warning"
date: 2017-12-17T23:01:47+08:00
lastmod: 2017-11-06T22:09:00+08:00
keywords: ["setState", "Synthetic Event Warning", "react"]
description: "setState的一个Synthetic Event Warning"
tags: ["react"]
categories: ["javascript"]
author: "lxzxl"
---

今天使用 React 时遇到一个警告：

![clipboard.png](/images/react/setstate-warning.png)

> Warning: This synthetic event is reused for performance reasons. If you're seeing this, you're accessing the property `target` on a released/nullified synthetic event. This is set to null. If you must keep the original synthetic event around, use event.persist(). See https://fb.me/react-event-pooling for more information.

查看文档解释：

> The SyntheticEvent is pooled. This means that the SyntheticEvent object will be reused and all properties will be nullified after the event callback has been invoked. This is for performance reasons. As such, you cannot access the event in an asynchronous way.

> SyntheticEvent 对象是通过合并得到的。 这意味着在事件回调被调用后，SyntheticEvent 对象将被重用并且所有属性都将被取消。这是出于性能原因。因此，您无法以异步方式访问该事件。

意思就是说我访问 event.target 时处于异步操作中。我的代码是：

```javascript
handleTitleChange: React.ChangeEventHandler<HTMLInputElement> = (event) => {
    this.setState((prevState) => ({
        collection: {
            title: event.target.value,
            content: prevState.content
        }
    }));
}
```

看了很多遍没发现哪里有异步操作，没办法只好加`event.persist()`先解决掉警告：

```javascript
handleTitleChange: React.ChangeEventHandler<HTMLInputElement> = (event) => {
    event.persist();
    this.setState((prevState) => ({
        collection: {
            title: event.target.value,
            content: prevState.content
        }
    }));
}
```

但是，我突然回想起`setState`并不保证是同步的：

> Think of setState() as a request rather than an immediate command to update the component. For better perceived performance, React may delay it, and then update several components in a single pass. React does not guarantee that the state changes are applied immediately.
> setState() **does not always immediately update** the component. It may batch or defer the update until later.

> 把 setState 当作是请求更新组件，而不是立即更新组件。为了性能，React 会延迟更新，会把多个组件的更新放在一次操作里。React 不保证 state 的改变会立刻发生。
> setState 并不总是立即更新组件。它可能会推后至批量更新。

所以，在我的这段代码中显然 setState 是异步的，因为 Event Polling 我们不能在 setState 的更新函数中访问 event 变量。既然找到了原因，解决方法就有了：

**不要在 setState 的更新函数中访问`event`变量:**

```javascript
handleTitleChange: React.ChangeEventHandler<HTMLInputElement> = (event) => {
    const val = event.target.value;
    this.setState((prevState) => ({
        collection: {
            title: val,
            content: prevState.content
        }
    }));
}
```

或者如果不需要 prevState 的话：

```javascript
handleTitleChange: React.ChangeEventHandler<HTMLInputElement> = (event) => {
    this.setState({
        collection: {
            title: event.target.value,
            content: this.state.content
        }
    });
}
```
