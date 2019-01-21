---
title: 手搓 Promises/A+
date: 2019-01-21 15:55:20
tags:
  - Javascript
  - Nodejs
  - Promise
---

最近在看`Nodejs`方面的内容，突然想到了`Promise`，于是想到看看它的标准，手搓一个出来。
这是一篇长文，准备分段更新。

---

## <a name="list">目录</a>

#### 1. [标准](#standard)

更新日志：
`2019年1月21日` 完成了[标准](#standard)部分的内容。

---

## <a name="standard" href="#list">标准</a>

先从[标准](https://promisesaplus.com/)入手，下文为标准的翻译。

### 术语

1. `Promise`对象/函数拥有`then`方法，其行为符合本规范
2. `thenable`表明一个对象/函数拥有`then`方法
3. `value`是一个合法的`JavaScript`值，包括`undefined`，`thenable`和`promise`
4. `exception`是通过`throw`抛出的`value`
5. `reason`是一个用来表示`promise`被`rejected`原因的`value`

### 要求

#### Promise 的状态

1. `Pending`
   - 一个`Pormise`可以过渡到`fulfilled`或`rejected`状态。
2. `fulfilled`
   - 无法过渡到其他状态
   - 必须有一个不变的`value`
3. `rejected`
   - 无法过渡到其他状态
   - 必须有一个不变的`reason`

#### then 方法

`Promise`必须提供`then`方法来访问其当前或最终的`value`或`reason`。
`promise`的`then`方法接受两个参数：

```js
promise.then(onFulfilled, onRejected);
```

1. `onFulfilled`和`onRejected`都是可选参数
   - 如果`onFulfilled`不是函数，其将会被忽略
   - 如果`onRejected`不是函数，其将会被忽略
2. 如果`onFulfilled`是函数
   - 它必须在`fulfilled`之后被调用,以`promise`的`value`作为它的第一个参数
   - 它不能在`fulfilled`之前调用
   - 它不能调用超过一次
3. 如果`onRejected`是函数
   - 它必须在`rejected`之后被调用,以`promise`的`reason`作为它的第一个参数
   - 它不能在`rejected`之前调用
   - 它不能调用超过一次
4. > onFulfilled or onRejected must not be called until the execution context stack contains only platform code.
   > 这段不大理解。
5. `onFulfilled`和`onRejected`必须作为函数调用(比如没有`this`)
6. `then`在一个`promise`中可调用多次
   - 如果`fulfilled`，所有相应的`onFulfilled`必须根据它们调用`then`的顺序被执行
   - 如果`rejected`，所有相应的`onRejected`必须根据它们调用`then`的顺序被执行
7. `then`必须返回一个`promise`
   ```js
   promise2 = promise1.then(onFulfilled, onRejected);
   ```
   - 如果任意`onFulfilled`或`onRejected`返回了`x`，运行该`Promise`解决过程：`[[Resolve]](promise2, x)`
   - 如果任意`onFulfilled`或`onRejected`抛出了`e`，`promise2`必须为`rejected`，且以`e`作为`reason`
   - 如果`onFulfilled`不是一个函数,而`promise1`已经`fulfilled`，则`promise2`必与`promise1`所`fulfilled`的`value`一致
   - 如果`onRejected`不是一个函数,而`promise1`已经`rejected`，则`promise2`必与`promise1`所`rejected`的`reason`一致

#### Promise 解决过程

`Promise`解决过程是一个抽象操作，它将`promise`和`value`作为输入，我们将其表示为`[[Resolve]](promise, x)`。如果`x`是`thenable`的，它会尝试采用`x`的状态（假设`x`至少运作地与`promise`相似），否则它以`x`的值`fulfill`。

对`thenables`的这种处理使得`promise`实现之间能够相互操作，只要它们公开符合`Promises/A+`的方法即可。它还允许`Promises/A+`实现使用合理的方法“同化”不一致的实现。

运行`[[Resolve]](promise, x)`要执行如下步骤：

1. 如果`promise`和`x`对应的是同一个对象，以`TypeError`来`reject`这个`promise`
2. 如果`x`是一个`promise`，采用它的`state`
   - 如果`x`为`pending`，那么在其被`fulfilled`或`rejected`之前，`promise`必须保持`pending`状态
   - 如果`x`被`fulfilled`，以相同的`value`来`fulfill`这个`promise`
   - 如果`x`被`rejected`，以相同的`reason`来`rejected`这个`promise`
3. 如果`x`是对象或函数
   - 使`then`成为`x.then`
   - 如果检索属性`x.then`时抛出异常`e`，以`e`为`reason`来`reject`这个`promise`
   - 如果`then`是一个函数，则以`x`作为`this`调用它，第一个参数为`resolvePromise`，第二个参数为`rejectPromise`，其中：
     1. 如果使用`value y`调用`resolvePromise`，运行`[[Resolve]](promise, y)`
     2. 如果使用`reason r`调用`rejectPromise`，以`r`来`reject`该`promise`
     3. 如果`resolvePromise`和`rejectPromise`都被调用，或者对同一参数进行多次调用，则以第一次调用为准，忽略之后的调用。
     4. 如果调用`then`抛出了异常`e`
        - `如果resolvePromise`或者`rejectPromise`已被调用，忽略
        - 否则，以`e`作为`reason`来`reject`该`promise`
   - 如果`then`不是函数，以`x`来`fulfill`该`promise`
4. 如果`x`不是对象或函数，以`x`来`fulfill`该`promise`

> If a promise is resolved with a thenable that participates in a circular thenable chain, such that the recursive nature of `[[Resolve]](promise, thenable)`eventually causes`[[Resolve]](promise, thenable)` to be called again, following the above algorithm will lead to infinite recursion. Implementations are encouraged, but not required, to detect such recursion and reject promise with an informative TypeError as the reason. `[3.6]`
