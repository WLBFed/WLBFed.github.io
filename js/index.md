# Javascript 基础

## 1. 什么是作用域？

作用域就是变量的访问规则，也就是规定了那些变量可以被访问，哪些变量不可以被访问。

JS 中有哪些作用域？

- 全局作用域：挂载在 window 上，通过 var 声明在最外层。
- 函数作用域：通过 return 访问函数内部变量，通过闭包访问函数内部变量
- 块级作用域：循环、if 等{}代码块里。
- 词法（静态）作用域：函数内部访问变量，总是寻找最近的那个作用域。
- 动态作用域：作用域是基于调用栈，而不是代码中的作用域嵌套，js 中除了 this 都是词法作用域。

## 2. 闭包

函数 A 内部有一个函数 B，函数 B 能访问到函数 A 内部变量，函数 B 就是闭包。

函数闭包的意义是让我们可以间接访问函数内部的变量。

```
for(var i = 1; i<= 5; i++){
    setTimeout(()=>{
        console.log(i)
    },i * 1000)
}

for(var i = 1;i<=5; i++){
    ;(function(i){
        setTimeout(()=>{
            console.log(i)
        },i * 1000)
    })(i)
}

for(let i = 1;i<=5; i++){
    setTimeout(()=>{
        console.log(i)
    },i * 1000)
}
```

闭包应用：

- 防抖函数
- 使用闭包设计单例模式

```
class CreateUser{
    constructor(name){
        this.name = name
        this.getName()
    }
    getName() {
        return this.name
    }
}
// 代理实现单例模式
let ProxyMode = (function() {
    let instance = null
    return function(name) {
        if(!instance){
            instance = new CreateUser(name)
        }
    }
    return instance
})()

let a = ProxyMode('aaa')
let b = ProxyMode('bbb')
// 因为单例模式只实例化一次，所以下面实例是相等的
console.log(a === b) // true
```

- 私有变量
  比如 redux 的设计，store 的唯一性就是通过闭包实现，只能通过 getStore 获取 store 实例，而不能直接操作。

## 3. 原型和继承

JS 继承的原理就是原型 prototype，这种实现继承的方式，我们称之为原型继承。

JS 中一些全局内置函数，分别为 Function/Array/Object

```
1.__proto__ === Number.prototype // true
'1'.__proto__ === String.prototype // tue
true.__proto__ === Boolean.prototype // true
```

一个对象的`__proto__` 总是指向它构造函数的 prototype。
`Object.prototype.__proto__` 为 null，这是原型链的终点。

### _原型链是什么？_

当我们访问一个对象的属性的时候，首先会在当前对象的 prototype 进行查找，如果找不到就会访问该对象的`__proto__`，如果`__proto__`有了就返回，如果没有就递归上述过程，知道`__proto__`为 null，这种查找属性的方式我们称为原型链上的查找，这条链我们称为原型链。

## 4. 用 ES5 实现一个继承

- 原型链继承：子类的原型指向父类的实例。
  `SubType.prototype = new SuperType()`
- 借用构造函数继承：

```
function Parent(value){
    this.val = value
}
Parent.prototype.getValue = function(){
    console.log(this.val)
}
function Child(value){
    Parent.call(this, value)
}

Child.prototype = new Parent()
```

> ES6 中我们会使用 class 继承，class 实现集成的核心在于 extends 表明哪个父类，并在子类的构造中调用 super，因为这段代码可以看做 `Parent.call(this, val)`

## 5. new 的实现

1. 新建一个空对象
2. 然后将这个空对象的`__proto__`指向构造函数的`prototype`
3. 调用构造函数去填充我们创建的空对象 4.更改 this 指向，将 this 指向我们刚刚创建的新对象

```
function new(constructor, ...args){
    const obj = {}
    obj.__proto__ = constructor.prototype
    const ret = constructor.call(obj, ...args)
    return ret instanceof Object? ret: obj
}

```

### 6. 深拷贝、浅拷贝

- 浅拷贝是精确拷贝，如果基本类型拷贝的就是值，如果是引用类型拷贝就是内存地址
- 深拷贝是将一个对象从内存汇总完整的拷贝一份，从堆内存中开辟一个新的区域用于存放新对象，且修改新对象不会影响源对象。

`浅拷贝实现`

1. Object.assign()方法可以把任意多个源对象自身的可枚举属性拷贝给目标对象，然后返回目标对象。
2. 展开运算符
3. Array.prototype.slice()

```
function cloneShallow(source){
    let target = {}
    for(let key in source){
        if(Object.prototype.hasOwnProperty.call(source,key)){
            target[key] = source[key]
        }
    }
}
```

`深拷贝实现`

1. JSON.parse(JSON.stringify())
2. 递归：遍历对象、数组直到里面是基本数据类型，然后再去复制

```
function isObject(obj){
    return Object.prototype.toString.call(obj) === '[object Object]'
}

function deepClone(source){
    if(!isObject(source)) return source
    let target = Array.isArray(source)?[]:{}
    for(let key in source){
        if(Object.prototype.hasOwnProperty(source,key)){
            if(isObject(source[key])){
                target[key] = deepClone(source[key])
            }else{
                target[key] = source[key]
            }
        }
    }
    return target
}
```

## 7. 判断数组和对象的类型

1. `typeof` 判断基本数据类型，但是不能判断数组。
2. `instanceof` 是根据原型链判断
   ，但是由于数组也在 Object 的原型链上，所以不能精确判断数组和对象。

3. 通过 Object.prototype.toString.call(arg) 判断变量类型
   通过 apply 也可以判断。

   ```
   Object.prototype.toString.call(function(){}) //  "[object Function]"
   Object.prototype.toString.call() // "[object Undefined]"
   Object.prototype.toString.call([]) // "[object Array]"
   ```

## 8. instanceof 实现

```
function instanceof(left, right){
    let prototype = right.prototype
    let proto = left.__proto
    while(true){
        if(proto === prototype){
            return true
        }
        if(proto === null){
            return false
        }
        proto = proto.__proto__
    }
}
```

### 9. 箭头函数

1. 箭头函数不会创建自己的 this
2. 箭头函数的 this 指向永远不变，不会通过 apply/call/bind 动态修改 this 指向
3. 箭头函数没有 arguments
4. 箭头函数没有原型
5. 箭头函数不能作为构造函数使用

### 10. for in? for of？iterator?

1. for in: 用于数组循环返回的是数组下标和挂载在原型上的键。

2. for of

- 可以遍历具有 iterator 接口的数据，一个数据只要部署了 Symbol.iterator 属性，就被视为具有 iterator 接口，`也就是说for...of循环内部调用时数据结构的Symbol.iterator`。
- iterator 的实现思想源于单向链表。
- forEach 循环中无法用 break 或 return 命令终止，单 for...of 可以。
- for...in 遍历数组遍历的是键名，所以适合遍历对象，for...of 遍历数组遍历的是键值。

- iterator: 一个数组结构只要具有 Symbol.iterator 属性，就认为是`可遍历的`，原生具备 iterator 接口的数据结构如下：

  1. Array
  2. Map
  3. Set
  4. String
  5. 函数的 arguments 对象
  6. NodeList 对象

```
function Car(name){
    this.name = name
}
const newCar = new Car('Bens')
Car.prototype.price = 160000
for(let key in newCar){
    console.log(key)
}
// name , price
for(let v of newCar){
    console.log(v)
}
// Uncaught TypeError: newCar is not iterable

```

**_Object 默认是没有 iterator 接口的_**

## 11. call/apply/bind 的区别与实现

- 相同点： 三者都是用来改变函数的上下文，也就是 this 指向的。
- 不同点：
  - fn.bind：不会立即调用，而是返回一个绑定后的新函数，this 指向第一个函数参数，后面可能更多参数，这些参数都是 fn 函数的参数。
  - fn.call：立即调用，返回函数执行结果，this 指向第一个函数，后面可能更多参数，并且这些都是 fn 函数的参数。
  - fn.apply：立即调用，返回的函数的执行结果，this 指向第一个参数，第二个参数是个数组，这个数组里内容是 fn 函数的参数。

## 12. promise 实现原理（怎么实现取消？怎么实现 promise all、race 等？）

Promise 实现

```
const PENDING = 'PENDING'
const RESOLVED = 'RESOLVED'
const REJECT = 'REJECT'

function Promise(fn){
    const self = this
    self.state = PENDING
    self.resolvedCallbacks = []
    self.rejectedCallbacks = []
    function resolve(value){
        if(self.state === PENDING){
            self.state = RESOLVED
            self.value = value
            self.resolvedCallbacks.map(cb => cb(self.value))
        }
    }
    function reject(value){
        if(self.state === REJECT){
            self.state = REJECT
            self.value = value
            self.rejectedCallbacks.map(cb => cb(self.value))
        }
    }
    try{
        fn(resolve, reject)
    }catch(e){
        reject(e)
    }
}

Promise.prototype.then = function(onResolved, onRejected){
    const self = this
    onResolved = typeof onResolved === 'function'? onResolved: v => v
    onRejected = typeof onRejected === 'function'? onRejected: v => v

    if(self.state === PENDING){
        self.resolvedCallbacks.push(onResolved)
        self.rejectedCallbacks.push(onRejected)
    }
    if(self.state === RESOLVED){
        onResolved(self.this)
    }
    if(self.state === REJECT){
        onRejected(self.this)
    }
}
```

Promise.all

```
Promise.all = function(promises){
    return new Promise((resolve, reject)=>{
        let resolvedCounter = 0
        let resolvedValues = new Array(promises.length)
        for(let i = 0;i< promises.length;i++){
            Promise.resolve(promise[i]).then((value)=>{
                resolvedCounter++
                resolvedValues[i] = value
                if(resolvedCounter === promises.length){
                   return resolve(resolvedValues)
                }
            }, (reject) => {
                return reject(reason)
            })
        }
    })
}
```

Promise.race：返回一个 Promise，Promises 里有一个解决或 2 拒绝，返回的 promise 就回解决

```
Promise.race = function(promises){
    return new Promise((resolve, reject)=>{
        for(let i = 0;i < promises.length; i++){
            Promise.resolve(promises[i]).then((value)=>{
                return resolve(reason)
            },(reason)=>{
                return reject(reason)
            })
        }
    })
}
```

## 13. JS 中的事件机制以及在浏览器和 Node 中的区别。

## 14. CommonJS、ES6 Module

## 15. 输入 URL 到显示页面，都经历了什么

## 16. v8 垃圾回收机制

## 17. 如何解决 script 标签阻塞渲染问题
