# ES6 + Webpack + React + Babel 如何在低版本浏览器上愉快的玩耍(下)
## 回顾

> ## 起因
> 某天，某测试说：“这个页面在 IE8 下白屏，9也白。。”

> 某前端开发: 吭哧吭哧。。。一上午的时间就过去了，搞定了。

> 第二天，某测试说：“IE 又白了。。”

> 某前端开发: 嘿咻嘿咻。。。谁用的 `Object.assign`，出来我保证削不屎你。

在[上篇](http://mp.weixin.qq.com/s?__biz=MzI2NzExNTczMw==&mid=2653285091&idx=1&sn=d503cb4a5f4d7c33f291e43569d39e08&scene=0#wechat_redirect)，我们主要抛出了两个问题，并给出了第一个问题的解决方案。
1. `SCRIPT5007: 无法获取属性 xxx 的值，对象为 null 或未定义`，这种情况一般是组件继承后，无法继承到在构造函数里定义的属性或方法，同样类属性或方法也同样无法继承
2.  `SCRIPT438: 对象不支持 xxx 属性或方法`，这种情况一般是使用了 es6、es7 的高级语法，`Object.assign` `Object.values` 等，这种情况在移动端的一些 ‘神机’ 也一样会挂。

本篇将给出第二个问题的解决方案, 并对第一个问题的解决方案有了更新的进展。

文章略长，请耐心看~嘿嘿嘿~

![image](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/platform/youshu-design/701aaa1afe05b3d77ba6a178e18d1da8/image.png)

## 正文开始
想要不支持该方法的浏览器支持，无非两种办法
1. 局部引用，引入一个相同的方法代替，其缺点则是使用起来比较麻烦，每个用到的文件都要去引入。
2. 全局实现，与之相反的方法是使用 polyfill ，其优点便是使用方便，缺点则是会全局污染，特别是实例方法，涉及到修改其 prototype ，不是你的类，你去修改它原型是不推荐的。

针对这两种办法，提供出以下几种方案，供大家参考

-----

### 方案一：引入额外的库
拿最常用的 assign 来说，可以这样

``` javascript
import assign from 'object-assign';
assign({}, {});
```
其实这种也是我们之前的使用方式，缺点就是需要去找到对应的库，比如 Promise 我们可以使用 [lie](https://github.com/calvinmetcalf/lie) 

另一方面一旦有人没有按照这个规则，而直接使用了 `Object.assign`，那这个人就可能被削。

-----

### 方案二： 全局引入 babel-polyfill

在项目的程序入口

``` javascript
import 'babel-polyfill';
```

babel 提供了这个 polyfill，有了它，你就可以尽情使用高级方法，包括 `Object.values` `[].includes` `Set` `generator` `Promise` 等等。其底层依赖的是 [core-js](https://github.com/zloirock/core-js) 。

但是这种方案显然有些暴力， polyfill 构建并 uglify 后的大小为 98k，gzip 后为32.6k，32k 对与移动端还是有点大的。

性能与使用是否方便自己权衡，比如离线包后或也可以接受。

-----

### 方案三：手动引入 [core-js](https://github.com/zloirock/core-js)
这个方案也稍微有些麻烦， core-js 里实现了大部分 e6、es7 的高级语法，具体列表可以去这里查看 https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js

我先截取一部分做下参考

``` javascript
  Object: {
      assign: "object/assign",
      create: "object/create",
      defineProperties: "object/define-properties",
      defineProperty: "object/define-property",
      entries: "object/entries",
      freeze: "object/freeze",
      ...
  }
```

具体怎么使用呢？找到要使用的方法的值，如：assign 是 "object/assign"，将其拼接至一个固定路径。

```javascript
import assign from 'core-js/library/fn/object/assign'
```
或

```javascript
import 'core-js/fn/object/assign'
```

这里包含上述所说的局部使用和全局实现的两种

直接引入 'core-js/fn/' 下的即为全局实现，你可以在程序入口引入你想使用的，这样相对于方案二避免了多余的库的引入

引入 'core-js/library/fn/' 下的即为局部使用，和方案一一样，只是省去了自己去寻找类库。

但是，实际使用，import 要写辣么长的路径，还是感觉有些麻烦。


-----

### 方案四：使用 [babel-plugin-transform-runtime](https://babeljs.io/docs/plugins/transform-runtime/)
本文会重点介绍下这个插件

先看下如何使用

``` javascript
// without options
{
  "plugins": ["transform-runtime"]
}

// with options
{
  "plugins": [
    ["transform-runtime", {
     "helpers": false, // defaults to true; v6.12.0 (2016-07-27) 新增;
      "polyfill": true, // defaults to true
      "regenerator": true, // defaults to true
      // v6.15.0 (2016-08-31) 新增
      // defaults to "babel-runtime"
      // 可以这样配置
      // moduleName: path.dirname(require.resolve('babel-runtime/package'))
      "moduleName": "babel-runtime"
    }]
  ]
}

```

该插件会做三件事情

> The runtime transformer plugin does three things:

> * Automatically requires babel-runtime/regenerator when you use generators/async functions.
> * Automatically requires babel-runtime/core-js and maps ES6 static methods (Object.assign) and built-ins (Promise).
> * Removes the inline babel helpers and uses the module babel-runtime/helpers instead.

* 第一件，如果你想使用 generator ， 有两个办法，一个就是引入 bable-polyfill 这个大家伙儿，另一个就是使用这个插件，否则你会看到这个错误

 ```
 Uncaught ReferenceError: regeneratorRuntime is not defined
 ```
* 第二件，就是能帮助我们解决一些高级语法的问题，它会在构建时帮你自动引入，用到什么引什么。

但是它的缺陷是它只能帮我们引入静态方法和一些内建模块，如 `Object.assign` `Promise` 等。实例方法是不会做转换的，如 `"foobar".includes("foo")` ,官方提示在这里:
 
> NOTE: Instance methods such as "foobar".includes("foo") will not work since that would require modification of existing builtins (Use babel-polyfill for that).

翻译一下就是，不要越俎代庖，不是你的东西你别乱碰，欠儿欠儿的。

![image](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/aliux_articles/articles/62b4c9e618b5844cbc171205142e6e31/image.png)


所以这个方案不会像方案二那样随心所欲的使用，但其实也基本够用了。

没有的实例方法可以采用方案三委屈下。

个人还是比较推荐这两种合体的方案。

需要注意的一点是：

开启 polyfill 后，会与 `export * from 'xx'` 有冲突

请看构建后的代码：

``` javascript
...
/***/ },
/* 106 */
/***/ function(module, exports, __webpack_require__) {

	'use strict';
	// 这是什么鬼。
	import _Object$defineProperty from 'babel-runtime/core-js/object/define-property';
	import _Object$keys from 'babel-runtime/core-js/object/keys';
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	...
```

截止 2016-09-10，官方尚未解决此 [issue](https://github.com/babel/babel-loader/issues/195), 只有先避开 `export * from 'xx'` 这种写法。或在[这里](https://webcache.googleusercontent.com/search?q=cache:zy5NcASoKlEJ:https://phabricator.babeljs.io/T2877+&cd=1&hl=zh-CN&ct=clnk&gl=cn)找答案。



* 第三件，是会引入一些 helper 来代替每次都生成的通用函数，看个例子就明白了

原来构建好的代码每个模块都有类似这种代码：

``` javascript
function _classCallCheck(instance, Constructor)...

function _possibleConstructorReturn(self, call)...

function _inherits(subClass, superClass)...

```

开启 helper 后：

``` javascript
var _classCallCheck2 = require('babel-runtime/helpers/classCallCheck');

var _possibleConstructorReturn2 = require('babel-runtime/helpers/possibleConstructorReturn');

var _inherits2 = require('babel-runtime/helpers/inherits');
```

这样统一引用了 helper，去处了冗余，看起来也更优雅了。

在 v6.12.0 之前 helper 也是默认开启的，没有配置可改，其他的 ployfill regenerator 都是有配置可以设置的。也许是推荐你使用 helper 。

但是 v6.12.0 (2016-07-27) 增加了 helper 的配置。为什么呢？

我最开始用这个插件的时候也很诧异，按道理来说，去除了冗余代码，代码的体积应该变小才对，但实际测试却变大了，我测试时是未经 uglify 的代码从 18k 增加到了 78k，查看构建模块增加了将近 100 个 [详情](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/vision/render-engine/8e6be96d5fbf2a99ea549fd071a607ec/image.png)。

原因是从 babel-runtime 里引入的 helper 依赖很多，全部都是兼容最底层的。比如 `Object.create` `typeof` 这种方法全部被重写了。

后来 gaearon 大神都忍不了了，他测试的结果是增加了 5kB min+gzip [详情](https://github.com/facebookincubator/create-react-app/pull/238)。

于是有了 helper 这个配置项。

另外还有一点，如果开启了 helper 的话，你会发现之前引用的 [babel-plugin-transform-proto-to-assign](https://github.com/babel/babel/tree/master/packages/babel-plugin-transform-proto-to-assign) 就失效了,虽然他本来就不该被使用，后面会讲到。

所以目前看来这个 helper 不用也罢。

再说下 moduleName 这个参数是干什么的？

还记得开启 helper 后的代码吗

``` javascript
var _classCallCheck2 = require('babel-runtime/helpers/classCallCheck');
```

看下这个路径，如果是本地项目安装了 babel-runtime 是没问题的，但如果你是用的通用构建工具，比如 [nowa](https://github.com/nowa-webpack/nowa)，所有的构建依赖库都是在公共的地方，毕竟 babel 太太了。这里就会报错了。

```
Cannot resolve module babel-runtime/regenerator
```

gaearon 大神在写  [create-react-app](https://github.com/facebookincubator/create-react-app) 时也发现了这个问题， [详情](https://github.com/facebookincubator/create-react-app/issues/255)

虽然这个问题可以通过 webpack 的 resolve.root 来解决，但是 gaearon 大神看其不爽，觉得依赖 webpack 不够优雅，[#3612](https://github.com/babel/babel/pull/3612) 于是乎就有了 moduleName 这个参数，已发布 v6.15.0 (2016-08-31)。

## 放弃 loose 模式， 放弃 ie8
上篇中提到了开启了 loose 模式来解决低版本浏览器无法继承到在构造函数里定义的属性或方法。

我们是通过 `babel-preset-es2015-ie` 这个插件，主要是改写了 `babel-plugin-transform-es2015-classes: {loose: true}` 和添加了插件 `babel-plugin-transform-proto-to-assign`（解决类方法继承的问题）

在 `babel-preset-es2015` v6.13.0 (2016-08-04) 时，presets 已经支持了参数配置，可以直接开启 loose 模式。

它内部会把开启一些插件的 loose 模式，不只是`babel-plugin-transform-es2015-classes`

``` javascript
{
  presets: [
    ["es2015", { "loose": true }]
  ]
}
```

这样我们就可以直接使用 `babel-preset-es2015`，至于 `babel-plugin-transform-proto-to-assign` 可以单独配置，也可不使用，因为类方法本来就不该被继承，要使用就直接 `Parent.defaultProps` 就可以了。

在上文中并没有提到开启 loose 模式的另一个原因是解决 ie8 下的两个 es3 属性名关键字的问题，因为上文测试均在 ie9 上，所以上述的方案也是停留在必须支持 ie8。

那么如果我们放弃了 ie8 ,看一看是不是会海阔天空。

在 `babel-plugin-transform-es2015-classes` v6.14.0 (2016-08-23) 一个 ‘大胡子哥’（原谅我不认识他） 修复了 `__proto__ `这个问题 [#3527 Fix class inheritance in IE <=10 without loose mode.](https://github.com/babel/babel/pull/3527) 
这样我们就可以在 ie9+ 上使用正常的 es6 模式了。

毕竟我们该向前看，loose 模式有点后退的赶脚。

[这篇文章](http://www.2ality.com/2015/12/babel6-loose-mode.html?utm_source=tuicool&utm_medium=referral)也表达了不推荐使用 loose 模式

> Con: You risk getting problems later on, when you switch from transpiled ES6 to native ES6. That is rarely a risk worth taking.

当然，如果真的离不开 ie8，就针对 es3 关键字的问题引用两个插件即可

```javascript
require('babel-plugin-transform-es3-member-expression-literals'),
require('babel-plugin-transform-es3-property-literals'),
```
我们再稍微看下‘大胡子哥’的修改，其实很简单,也很巧妙，看一行关键代码

```javascript
// 修改后生成的代码多了一个 先取 `xxx.__proto__` 再使用 `Object.getPrototypeOf`
  var _this = _possibleConstructorReturn(this, (Test.__proto__ || Object.getPrototypeOf(Test)).call(this, props));

```

回顾下 inherits 方法的实现

```javascript
function _inherits(subClass, superClass) {
	...
	// 虽然 ie9/10 不支持 `__proto__`，这里只是作为了普通对象给予赋值，`Object.getPrototypeOf` 获取不到但可以直接 `.__proto__` 获取
  Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
  ...
```

如果你看懂了实现方式，不知道你有没有发现 `babel-plugin-transform-proto-to-assign`（解决类方法继承的问题）这个家伙真的不能用了

```javascript
function _inherits(subClass, superClass) { 
  ...
  // 因为它会将 `__proto__` 转为 `_default` 
  Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : _defaults(subClass, superClass);
}
```

这样上述的修复就无效了。切记不能使用，还是那句话，类方法本来就不该被继承。

最后看下终极方案的通用配置

```javascript
{
  plugins: [
    ["transform-runtime", {
      "helpers": false,
      "polyfill": true,
      "regenerator": true
    }],
    'add-module-exports',
    'transform-es3-member-expression-literals',
    'transform-es3-property-literals',
  ],
  "presets": [
    'react',
    'es2015',
    'stage-1'
  ],
}
```

更简单、完整的解决方案，请查看 [nowa](https://github.com/nowa-webpack/nowa)

感谢阅读。

## 参考链接： 
* https://babeljs.io/docs/plugins/transform-runtime
* https://github.com/babel/babel


## 广告时间： 请献出你的小星星 
* https://github.com/saltjs/salt-ui
* https://github.com/uxcore/uxcore

