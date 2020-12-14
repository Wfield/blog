## 在 JavaScript 中函数的定义
  > 在 JavaScript中，函数是**头等(**first-class**)**对象，因为它们可以像任何其他**对象**一样具有属性和方法。它们与其他对象的区别在于函数可以被调用。简而言之，它们是[Function](https://developer.mozilla.org/zh-cn/JavaScript/Reference/Global_Objects/Function "zh-cn/Core_JavaScript_1.5_Reference/Global_Objects/Function")对象

## Function
- javascript 标准内置对象
- 每个 js 函数都是一个 Function 类型的对象
  `(function(){}).constructor === Function // true`
  `typeof (function(){})  //  function`
- Function 同时是一个构造器
  `new Function ([arg1[, arg2[, ...argN]],] functionBody)`
  以调用函数的方式调用 Function 的构造函数（而不是使用 `new` 关键字) 跟以构造函数来调用是一样的。
  ```
  const sum1 = new Function('a', 'b', 'return a + b');
  sum1(1, 2)  // 3
  const sum2 = Function.constructor('a', 'b', 'return a + b');
  sum2(1, 2) // 3
  ```
- 属性和方法
  > 全局的 `Function` 对象没有自己的属性和方法，但是，因为它本身也是一个函数，所以它也会通过原型链从自己的原型链 [`Function.prototype`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype) 上**继承**一些属性和方法。


## Function.prototype
  Function 对象通过原型链从自己的原型链`Function.prototype`上**继承**一些属性和方法
  **Function.prototype 的值**
`ƒ () { [native code] }`

**函数的 prototype**
```
function fn() {}
fn.proptotype 
// 输出：
{
    constructor: ƒ doSomething(),
    __proto__: {
        constructor: ƒ Object(),
        hasOwnProperty: ƒ hasOwnProperty(),
        isPrototypeOf: ƒ isPrototypeOf(),
        propertyIsEnumerable: ƒ propertyIsEnumerable(),
        toLocaleString: ƒ toLocaleString(),
        toString: ƒ toString(),
        valueOf: ƒ valueOf()
    }
}
```
**函数的 `__proto__`**
```
function fn() {}
fn.__proto__
// 输出：
ƒ () { [native code] }

fn.__proto__ === Function.prototype
```

```javascript
typeof Function.prototype  // 'function'
Function.prototype()  //  空函数

function fn() {};
typeof fn // function
typeof fn.prototype // object
```
## 构造函数
当使用 [new 操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new) 来作用这个函数时，它就可以被称为构造方法（`构造函数`）

## 函数的实例
```
function Graph() {
  this.vertices = [];
  this.edges = [];
}

Graph.prototype = {
  addVertex: function(v){
    this.vertices.push(v);
  }
};

var g = new Graph();
// g 是生成的对象，他的自身属性有 'vertices' 和 'edges'。
// 在 g 被实例化时，g.[[Prototype]] 指向了 Graph.prototype。
// g.__proto__ === Graph.prototype
```
