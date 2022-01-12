### bind、call、apply的区别

#### 区别
- `call`和`apply`只是传参方式不同，第一个参数均为`this`指向，第二个参数`call`接收参数列表（逐一传入），`apply`接收参数数组
- `bind`不立即执行，而是返回一个函数，传参方式跟`call`类似，如果有传参数列表，将作为目标函数调用时，优先置入绑定函数的参数列表
- 特别的，第一个参数传递的任何原始值都会被转换为`object`


#### 实现一个简单的call函数
```js
Function.prototype.myCall = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  
  context = context || window
  
  if (typeof context !== 'object') context = new Object(context)

  context.fn = this

  context.fn(...[...arguments].slice(1))
  
  delete context.fn
}
```

#### 实现一个简单的apply函数
```js
Function.prototype.myApply = function (context, args = []) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  
  context = context || window
  
  if (typeof context !== 'object') context = new Object(context)

  context.fn = this

  context.fn(...args)

  delete context.fn
}
```

#### 实现一个简单的bind函数

```js
Function.prototype.myBind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  
  context = context || window
  
  if (typeof context !== 'object') context = new Object(context)

  const _this = this
  
  const args = [...arguments].slice(1)

  return function FN () {
    if (this instanceof FN) {
      // new方式调用，忽略context
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat([...arguments]))
  }
}
```

