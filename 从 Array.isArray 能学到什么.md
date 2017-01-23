# 从 Array.isArray 能学到什么？

我们假设要判断一个值是否是数组，写个isArray() 的方法，虽然从ES5就已经添加了 `Array.isArray()`
   
首先我们使用 `typeof`

```
typeof [] === 'object' //true
typeof {} === 'object' //true
typeof null === 'object' //true
```

显然使用 `typeof` 是不行的。

然后我们可能会想到 `instanceof` 或 `constructor`

```
[] instanceof Array === true //true
[].constructor === Array //true
```

乍一看貌似可以

`constructor`
> All objects inherit a constructor property from their prototype。
所有的objects 都从它们的prototype继承了constructor属性。
而这个constructor指向了这个object本身,即 

```
Array.prototype.constructor === Array //true
```
所以

```
arr.constructor === Array; //true
```
 
`instanceof`
> The instanceof operator tests presence of constructor.prototype in object prototype chain.
`instanceof` 用来判断 `constructor.prototype`是否存在于某个object的原型链上。

为了更好的理解，先解释下什么是原型？
在 JavaScript 中
用`prototype`来表示显示原型；
用` [[Prototype]]` 表示对象隐式的原型，
在一些浏览器中会暴露出一个 `__proto__` 来访问这个隐式原型。

注意：

> Following the ECMAScript standard, the notation someObject.[[Prototype]] is used to designate the prototype of someObject. This is equivalent to the JavaScript property `__proto__` (now deprecated). Since ECMAScript 6, the [[Prototype]] is accessed using the accessors Object.getPrototypeOf() and Object.setPrototypeOf().
根据 ECMAScript 标准，someObject.[[Prototype]] 符号是用于指派 someObject 的原型。这个等同于 JavaScript 的 `__proto__`  属性（现已弃用）。从 ECMAScript 6 开始, [[Prototype]] 可以用Object.getPrototypeOf()和Object.setPrototypeOf()访问器来访问。

```
var o = {}; 
Object.getPrototypeOf(o) === o.__proto__  //true
``` 

显示原型不解释。
隐式原型是用来构成javascript的原型链：
当我们访问一个对象的属性时，如果找不到，那么就会去`[[Prototype]]`里找，如果没有则会继续找下去,直到`[[Prototype]]`为`null`,即 `Object.getPrototypeOf(Object.prototype)` ,原型链至此结束。
这也验证了
> All objects in JavaScript are descended from Object; all objects inherit methods and properties from Object.prototype
> 在javascript中，Object是'祖先'，所有objects都是从Object.prototype下继承属性和方法。

现在我们可以更好的理解`instanceof`：
如果 `constructor.prototype`（上面讲过 `constructor` 指向了对象本身,拿数组为例的话，即`Array.prototype`） 等于对象(object)原型链上的某一个原型，那么instanceof运算将返回true。

由于

```
Array.prototype === Object.getPrototypeOf([]) //true
//or
Array.prototype === [].__proto__; //true
```
所以 

```
[] instanceof Array; // true  
```

回到原始问题，使用`instanceof` 或 `constructor`来实现`isArray`是否可行呢？答案是否定的。

因为在不同iframe下，就不行了。

```
var iframe = document.createElement('iframe');   
document.body.appendChild(iframe);   
var OtherArray = window.frames[window.frames.length-1].Array;   
var arr = new OtherArray(1,2,3); // [1,2,3]   

arr instanceof Array === true //false  
arr.constructor === Array //false  
```

> Different scope have different execution environments. This means that they have different built-ins (different global object, different constructors, etc.).

因为在不同iframe下，每个iframe都有一套自己的执行环境，不同的执行环境具有不同的原型链。

上面其实是借isArray的实现来解释 `typeof`, `constructor` 和 `instanceof` ,从而更深入的了解 javascript 的原型链。

那么真正的[isArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray
)实现很简单。
主要原理是利用了 `Object.prototype.toString()`
> If this method is not overridden in a custom object, toString() returns "[object type]"
如果这个方法没有被重载的话，它会返回"[object type]"，即可以得到对应的类型。

`Object.prototype.toString()`可以判断所有的内置类型，这里只是用Array举例而已。

```
var isArray = function (argv) {
    return Object.prototype.toString.call(argv) === "[object Array]";
};
```

为什么不直接 `[].toString()` ? 
因为[].toString，已经被重载了。


参考链接：
1. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain
2. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray


