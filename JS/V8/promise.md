### V8 Promise

#### 前置

- `Promise`的3种状态 `pending` `fulfilled` `rejected`
- 调用`Promise`构造函数必须使用`new`操作符
- `Promise`构造函数必须传入`executor`函数，参数为`resolve`、`reject`
- `reactions_or_result` 用于存储`resolve`的值或者`then`处理函数的链表

#### then
- V8层核心处理逻辑
  - 创建一个新的`promise`用于本次then的调用结果返回
  - 获取then接收到的两个参数（`onFulfilled` `onRejected`），默认为`undefined`
  - 调用`PerformPromiseThenImpl`完成大部分工作
  - 返回新的`promise`（`then`方法返回的是一个新的 `Promise`）

#### PerformPromiseThenImpl
- `PerformPromiseThenImpl`有四个参数，分别为
  - 被调用`then`的`promise`对象
  - `onFulfilled`
  - `onRejected`
  - 返回的新`promise`
- V8层核心处理逻辑
  - 判断原`promise`状态
    - 状态为`pending`
      - 将`then`绑定的处理函数以头插链表的形式储存在`reactions_or_result`上
    - 状态为`fulfilled`
      - 将`promise`的值赋值给`reactions_or_result`，并作为参数传入`onFulfilled`
      - 将`onFulfilled`生成`microtask`任务
    - 状态为`rejected`
      - 将`promise`的值赋值给`reactions_or_result`，并作为参数传入`onRejected`
      - 将`onRejected`生成`microtask`任务
      - 如果当前`promise`未绑定过处理函数，调用`HostPromiseRejectionTracker`产生一个`microtask`任务，用于再次异步检查是否绑定处理函数
  - 将`microtask`插入微任务队列

#### HostPromiseRejectionTracker
- 该抽象方法作用为标记`promise`已经绑定了`rejected`状态的处理函数
- 如果没有 `onRejected` 处理函数，则为其添加一个处理函数，而后加入微任务队列，异步再次检测，如果还没有，则抛出异常
- 该方法在两种情况下被调用
  - 当一个`promis`e在没有任何处理函数的情况下被拒绝，标记`reject`
  - 当第一次将处理函数添加到被拒绝的`promise`中时，标记`handle`（在`microtask`任务中）

#### resolve - FulfillPromise
- 作用是将状态从`pending`改变为`fulfilled`，并将所有处理函数都变成`microtask`插入微任务队列
- V8层核心处理逻辑
  - 从`reactions_or_result`中取到处理函数链表，并将`promise`的值赋值给`reactions_or_result`
  - 改变状态为`fulfilled`
  - 反转处理函数链表，因为链表是采用头插法插入新节点的
  - 遍历链表，生成`microtask`插入微任务队列

#### reject - RejectPromise
跟`FulfillPromise`区别不大，只是如果没有绑定处理函数，会执行`HostPromiseRejectionTracker`，同`then`方法中的处理逻辑

#### catch - PromisePrototypeCatch
内部会调用`InvokeThen`执行`promise`的then方法，因此常说`catch`是语法糖

#### then 的链式调用与 microtask 队列
- 当一个 `Promise` 处于 `rejected` 状态时，如果找不到 `onRejected` 处理函数则会将 `rejected` 的状态和其值往下传递，直到找到为止。（`resolve`也是一样）
- `catch` 方法的作用就是绑定 `onRejected` 函数
- 链式调用底层依赖`PromiseReactionJob`

#### PromiseReactionJob
判断当前任务是否存在需要执行的处理函数，如果不存在则直接将上一个 `Promise` 的值作为参数调用 `FuflfillPromiseReactionJob` ，如果存在则执行这个处理函数，将执行结果当做参数调用 `FuflfillPromiseReactionJob`。

也就是说，只要一个 `Promise` 的 `onFulfilled` 或者 `onRejected` 在执行过程中只要没有抛出异常，这个 `Promise` 就会执行 `FuflfillPromiseReactionJob` 将状态修改为 `fulfilled`。如果抛出异常则执行 `RejectPromiseReactionJob`。

#### FuflfillPromiseReactionJob
调用 `RejectPromise` 来调用 `Promsie` 的 `resolve` 方法

#### ResolvePromise
基本上每一个 `Promise` 的状态需要变成 `fulfilled` 都会调用它
- V8层核心处理逻辑
  - 调用`FulfillPromise`
  - 如果返回值`resolution`是一个`promise`对象或一个包含`then`方法的对象，会调用`NewPromiseResolveThenableJobTask` 生成一个 `microtask`，然后将其加入 `microtask` 队列中

#### NewPromiseResolveThenableJobTask
目的是调用 `resolution` 的 `then` 方法，在回调函数中同步状态给 `promise`
并且这个整体的过程需要加入到微任务队列中等待执行
```text
注意: 此作业使用提供的 thenable 及其 then 方法来解决给定的 Promise。 此过程必须作为作业进行，以确保在对任何周围代码的评估完成后对 then 方法进行评估。
引至 ECMAScript NewPromiseResolveThenableJobTask 规范(作者翻译)
```
```js
microtask(() => {
  resolution.then((value) => {
    ReslovePromise(promise, value) 
  })
})
```

#### RejectPromiseReactionJob
`RejectPromiseReactionJob` 与 `FuflfillPromiseReactionJob` 是类似的，就是调用 `RejectPromise` 来调用 `Promsie` 的 `reject` 方法，这个在上面 `reject` 的地方介绍过了。

#### PromiseReactionJob的handler == Undefined分支
当一个 `task` 中的 `handler` 为 `undefined`时，会直接获取上一个 `Promise` 对象的 `value` 和 状态 同步到当前 `promise` 来



#### 参考

[月夕大佬的文章](https://juejin.cn/post/7055202073511460895#heading-34)