# ES6 + Webpack + React + Babel 如何在低版本浏览器上愉快的玩耍(上)
## 起因
某天，某测试说：“这个页面在 IE8 下白屏，9也白。。”

某前端开发: 吭哧吭哧。。。一上午的时间就过去了，搞定了。

第二天，某测试说：“IE 又白了。。”

某前端开发: 吭哧吭哧。。。谁用的 `Object.assign`，出来我保证削不屎你。

![](https://gw.alicdn.com/tfscom/TB1AR1.LXXXXXaNXpXXXXXXXXXX.png)

原谅我不禁又黑了一把 IE。

有人可能会想，都要淘汰了，还有什么好讲的？

也许几年后，确实没用了，但目前我们的系统还是要对 ie8+ 做兼容，因为确实还有个别用户，尽管他没朋友。。。

记录下本次在 IE 下踩得坑，让后面的同学能够不再在这上面浪费时间了。

## 经过
### 测试

首先，看下面代码(以下测试在 IE9)

```javascript
class Test extends React.Component {
  constructor(props) {
	super(props);
  }
  render() {
    return <div>{this.props.content}</div>;
  }
}

module.exports = Test;
```

这段代码跑的妥妥的，没什么问题。

 一般来说，babel 在转换继承时，可能会出现兼容问题，那么，再看这一段

```javascript
class Test extends React.Component {
  constructor(props) {
	super(props);
  }
  test() {
  	console.log('test');
  }
  render() {
    return <div>{this.props.content}</div>;
  }
}

Test.defaultProps = {
  content: "测试"
};

class Test2 extends Test {
  constructor(props) {
	super(props);
	this.test();
  }
}

Test2.displayName = 'Test2';

module.exports = Test2;
```
这段代码同样也可以正常运行

也就是说在上述这两种情况下，不做任何处理（前提是已经加载了 es5-shim/es5-sham），在 IE9 下都可以正常运行。

然后我们再看下会跑挂的代码

```javascript
class Test extends React.Component {
  constructor(props) {
	super(props);
	this.state = {
	  test: 1,
	};
  }
  test() {
  	console.log(this.state.value);
  }
  render() {
    return <div>{this.props.content}</div>;
  }
}

Test.defaultProps = {
  content: "测试"
};

class Test2 extends Test {
  constructor(props) {
	super(props);
	// SCRIPT5007: 无法获取属性 "value" 的值，对象为 null 或未定义
	this.test();
	
	// SCRIPT5007: 无法获取属性 "b" 的值，对象为 null 或未定义
	this.a = this.props.b;
  }
}
// undefined
console.log(Test2.defaultProps);

Test2.displayName = 'Test2';

module.exports = Test2;
```

这段代码在高级浏览器中是没问题的，在 IE9 中会出现注释所描述的问题

从这些问题分析，可得出3个结论

1. 在构造函数里的定义的属性无法被继承
2. 在构造函数里不能使用 `this.props.xx`
3. 类属性或方法是无法被继承的

也就是说，只要规避了这三个条件的话，不做任何处理（前提是已经加载了 es5-shim/es5-sham），在 IE9 下都可以正常运行。

> 第二点，是完全可以避免的，切记在 `constructor` 直接使用 `props.xxx`, 不要再用 `this.props.xxx`

> 第三点，也是可以完全避免的，因为从理论上来说，类属性就不该被继承，如果想使用父类的类属性可以直接`Test2.defaultProps = Test.defaultProps;`

> 第一点，可避免，但无法完全避免

### 原因

第一点，有时是无法完全避免的，那么就要查询原因，才能找到解决方案

我们把 babel 转义后的代码放出来就能查出原因了

```javascript
'use strict';

var _createClass = function () {
  ...
}();

function _classCallCheck(instance, Constructor) { 
  ...
}

function _possibleConstructorReturn(self, call) { 
  ...
  // 这个方法只是做了下判断，返回第一个或第二参数
  return call && (typeof call === "object" || typeof call === "function") ? call : self; }

function _inherits(subClass, superClass) { 
  ...; 
  // 这里的 _inherits 是通过将子类的原型[[prototype]]指向了父类，所以如果在高级浏览器下，子类的可以继承到类属性
  // 根本问题也是出在这里，IE9 下既没有 `setPrototypeOf` 也没有 `__proto__`
  if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass; 
}

var Test = function (_React$Component) {
  ...
  return Test;
}(React.Component);

Test.defaultProps = {
  content: "测试"
};

var Test2 = function (_Test) {
  _inherits(Test2, _Test);

  function Test2(props) {
    _classCallCheck(this, Test2);
	 // 这里的 this 会通过 _possibleConstructorReturn，来获取父类构造函数里定义的属性
	 // _possibleConstructorReturn 只是做了下判断，如果第二个参数得到了正确执行，则返回执行结果，否则返回第一个参数，也就是子类的 this
	 // 也就是说问题出在 Object.getPrototypeOf 
	 // 在 _inherits 中将子类的原型指向了父类， 这里通过 getPrototypeOf 来获取父类，其实就是 _Test
	 // Object.getPrototypeOf 不能正确的执行，导致了子类无法继承到在构造函数里定义的属性或方法，也无法继承到类属性或方法
    var _this2 = _possibleConstructorReturn(this, Object.getPrototypeOf(Test2).call(this, props));

    _this2.test();
    console.log(_this2.props.children);
    return _this2;
  }

  return Test2;
}(Test);

console.log(Test2.defaultProps);

Test2.displayName = 'Test';

module.exports = Test2;
```

通过上述的代码注释，可以得出有两处问题需要解决

1. 正确的获取父类（解决无法继承到在构造函数里定义的属性或方法）
2. 正确的将子类的原型指向了父类（解决无法继承到类属性或方法）

### 解决方案
通过文档的查询，发现只要开启 es2015-classes 的 loose 模式即可解决第一个问题

#### loose 模式
 
> Babel have two modes:

> * A normal mode follows the semantics of ECMAScript 6 as closely as possible.
> * A loose mode produces simpler ES5 code.

Babel 有两种模式：

* 尽可能符合 ES6 语义的 normal 模式。
* 提供更简单 ES5 代码的 loose 模式。

尽管官方是更推荐使用 normal 模式，但为了兼容 IE，我们目前也只能开启 loose 模式。

在 babel6 中，主要是通过 babel-preset-2015 这个插件，来进行转义的
我们看下 babel-preset-2015

```javascript
 plugins: [
    require("babel-plugin-transform-es2015-template-literals"),
    require("babel-plugin-transform-es2015-literals"),
    require("babel-plugin-transform-es2015-function-name"),
    ...
    require("babel-plugin-transform-es2015-classes"),
    ...
    require("babel-plugin-transform-es2015-typeof-symbol"),
    require("babel-plugin-transform-es2015-modules-commonjs"),
    [require("babel-plugin-transform-regenerator"), { async: false, asyncGenerators: false }],
  ]
```

是一堆对应转义的插件，从命名上也可看出了大概，比如 babel-plugin-transform-es2015-classes 就是做类的转义的，也就是我们只需把它开启 loose 模式，即可解决我们的一个问题

```javascript
[require('babel-plugin-transform-es2015-classes'), {loose: true}],
```


看下开启了 loose 模式的代码，你会发现它的确更接近 ES5

```javascript
var Test = function (_React$Component) {
  ...
  // 这里是 ES5 的写法
  Test.prototype.test = function test() {
    console.log(this.state.value);
  };
  /* normal 模式是这样的
  {
    key: 'test',
    value: function test() {
      console.log(this.state.value);
    }
  }
  */
  return Test;
}(React.Component);

var Test2 = function (_Test) {
  _inherits(Test2, _Test);

  function Test2(props) {
    _classCallCheck(this, Test2);
    // 这里直接拿到了父类 _Test, 即解决了无法继承到在构造函数里定义的属性或方法
    var _this2 = _possibleConstructorReturn(this, _Test.call(this, props));

    _this2.test();
    return _this2;
  }

  return Test2;
}(Test);
```

我们可以通过去安装 [babel-preset-es2015-loose](https://github.com/bkonkle/babel-preset-es2015-loose), 这个插件来开启 loose 模式。

但从我们团队的 __老司机__ 口中

![image](http://aligitlab.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/aliux_articles/articles/313a2902f26a6f399456142c40cb1224/image.png)

得到了一个更好插件[babel-preset-es2015-ie](https://github.com/jmcriffey/babel-preset-es2015-ie)，看下这个插件的代码，发现它和原来的 babel-preset-2015 只有两行区别

```javascript
[
  [require('babel-plugin-transform-es2015-classes'), {loose: true}],
  require('babel-plugin-transform-proto-to-assign'),
]
```
刚好解决我们上述碰到的两个问题

这个 `babel-plugin-transform-proto-to-assign` 插件会生成一个 _defaults 方法来处理原型

```javascript
function _inherits(subClass, superClass) { 
  ...; 
  if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : _defaults(subClass, superClass);
}
```

```javascript
function _defaults(obj, defaults) {
 var keys = Object.getOwnPropertyNames(defaults);
  for (var i = 0; i < keys.length; i++) {
   var key = keys[i]; 
   var value = Object.getOwnPropertyDescriptor(defaults, key);
    if (value && value.configurable && obj[key] === undefined) {
     Object.defineProperty(obj, key, value); 
     } 
   }
  return obj;
}

```

> 这个插件正确的将子类的原型指向了父类（解决无法继承到类属性或方法）


### 总结
本文讲述低版本浏览器报错的原因和解决方案

* 一方面是提示下在构造函数里不要使用 `this.props.xx`

* 另一方面也对继承的机制有了更好的理解

在这次项目中发现在低版本浏览器跑不起来的两点主要原因：

1. `SCRIPT5007: 无法获取属性 xxx 的值，对象为 null 或未定义`，这种情况一般是组件继承后，无法继承到在构造函数里定义的属性或方法，同样类属性或方法也同样无法继承
2.  `SCRIPT438: 对象不支持 xxx 属性或方法`，这种情况一般是使用了 es6、es7 的高级语法，`Object.assgin` `Object.keys` 等，这种情况在移动端的一些 ‘神机’ 也一样会挂。


第一点本文已经分析，预知第二点讲解请见下篇。

备注：下篇会主要介绍下如何让 用了 `Object.assign` 的那位同学可以继续用，又不会被削。


