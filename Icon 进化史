# Icon 进化史
## “南方古猿”之 png sprite
![](https://gw.alicdn.com/tfscom/TB1aUbHPXXXXXbbapXXXXXXXXXX.png)

看到上面这张图，相信好多资深前端会感到很亲切。

早期为了减少资源的请求，人们想到了将小的 png 图片合并到一张图上，然后根据 `background-position`   来显示不同的图片。

早期还有靠人肉来测量坐标，随着构建工具的发展，我们可以用一些插件，如 [grunt-spritesmith](https://github.com/Ensighten/grunt-spritesmith)、[gulp.spritesmith](https://github.com/twolfson/gulp.spritesmith) 等。它可以帮助我们自动合成，并生成好 css， 位置都计算好的。

那么使用 png 图片这种方式它的优点就是兼容性好。但是一旦开发多了，它的不便变体现出来了，换颜色、改大小、透明度、多倍屏等等。

所以对于这种方式我们只能缅怀。

## “能人”之 Iconfont
于是人们又想出了用字体文件取代图片文件：Iconfont。

虽然早期制作或寻找合适字体比较麻烦，但随着各种字体库的网站出现，如： http://www.iconfont.cn ，那都不是事了。再加上 css 的自定义字体的兼容性非常好，Iconfont 迅速开始流行起来。以这个站为例，大概看下使用方法：

1. 在 Iconfont 中创建自己的项目，将需要使用的图标添加到自己的项目中。
2. 复制出 Unicode 或 Font class
3. 全部代码如下

```css
@font-face {
  font-family: 'iconfont';  /* project id 38792 */
  src: url('//at.alicdn.com/t/font_1444792316_9706304.eot');
  src: url('//at.alicdn.com/t/font_1444792316_9706304.eot?#iefix') format('embedded-opentype'),
  url('//at.alicdn.com/t/font_1444792316_9706304.woff') format('woff'),
  url('//at.alicdn.com/t/font_1444792316_9706304.ttf') format('truetype'),
  url('//at.alicdn.com/t/font_1444792316_9706304.svg#iconfont') format('svg');
}
```

```
.iconfont {
  font-family:"iconfont" !important;
  font-size:16px;
  font-style:normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.icon-tishi:before { content: "\e600"; }
```

```html
<i class="iconfont icon-tishi"></i>
```
[这里有demo](https://codepen.io/maming/pen/YNpMwd)

在实际开发中，我们会把常用的一些图标封装成[组件](http://uxcore.coding.me/css/iconfont/)，直接使用。像这样

```html
<i class="kuma-icon kuma-icon-success"></i>
```
Iconfont 用起来挺方便的，而且兼容性也十分的好，大小、颜色可随意改变。

但它仍有缺陷：
1. 字体需要加载资源
2. 有时候可能会出现锯齿
3. 只能被渲染成单色或者css3的渐变色

所以我们要继续进化。

## “直立人”之 svg symbol
使用 svg ，这里所谓的进化并不是 svg 本身的进化，因为 svg 并不晚于 Iconfont。只是环境（兼容性）的原因导致它无用武之地。就像最近火的一塌糊涂的 AI, 其实最早在 1956 年就提出了。随着外界因素的进化，IE6/7/8 的淘汰， android 4.x 的开始，svg 的机会变到来了。先看下兼容性：
![image](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/vision-components/vc-multi-select-field/f0d483f9f7bc662bbfc3bb71695f57e5/image.png)

### [svg](https://developer.mozilla.org/en-US/docs/Web/SVG) 的使用方式：
* 保存成文件
    * 需要请求加载资源
* inline 方式
    * 在 html 一坨坨，很麻烦 
* [symbol](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/symbol)
    * 适合我们做组件

> The `<symbol>` element is used to define graphical template objects which can be instantiated by a `<use>` element.

通过 <symbol> 定义的 svg 模板，我们可以使用 <use> 来加载它。

```html
<svg width="120" height="140">
<!-- symbol definition  NEVER draw -->
<symbol id="sym01" viewBox="0 0 150 110">
  <circle cx="50" cy="50" r="40" stroke-width="8"
      stroke="red" fill="red"/>
  <circle cx="90" cy="60" r="40" stroke-width="8"
      stroke="green" fill="white"/>
</symbol>

<!-- actual drawing by "use" element -->
<use xlink:href="#sym01"
     x="0" y="0" width="100" height="50"/>
</svg>
```

那么 <symbol> 是怎么来的呢？

同样，在这个构建工具十分发达的时刻。
最开始我们使用了 [gulp-svg-symbols](https://github.com/Hiswe/gulp-svg-symbols)，它可以将指定目录中的 svg 自动合并到一个 svg 文件中，文件里包括了所有 icon 的 symbol 模板，然后再将这个模板将其隐藏放到 index.html 中。

但是这个库有个坑点是它依赖了一个 Unicode 的包，这个包在国内安装炒鸡慢，于是我们弃用了它，使用了[gulp-svgstore](https://github.com/w0rm/gulp-svgstore)

按照这种方式我们成功的开发一 [salt-icon](https://github.com/saltjs/salt-ui/blob/master/demo/src/pages/icon/PageIcon.js) 这个组件，里面包括了一些常用的 icon。使用方式：

```javascript
    <Icon name="success" className="icon-success"/>
```
这样我们在 mobile 端用 svg 替代了 Iconfont，解决了上述 Iconfont 提到的问题。

但是很快我们就发现，在 index.html 中引入那一坨 symbol 模板是极其恶心的。

随着 [webpack](https://webpack.github.io/) 打包的成熟，各种 loader，我们将那一坨 symbol 模板直接打包成一个 salt-icon-source.js 文件，在这个文件中将其 append 到 body 上。

同时发现了上述提到的 [iconfont 网站](http://www.iconfont.cn/)也支持直接导出 symbol 文件。

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import IconSource from './svg/salt-icon-symbols.svg';


const WRAPPER_ID = '__SaltIconSymbols__';
const doc = document;
let wrapper = doc.getElementById(WRAPPER_ID);
if (!wrapper) {
  wrapper = doc.createElement('div');
  wrapper.id = WRAPPER_ID;
  wrapper.style.display = 'none';
  doc.body.appendChild(wrapper);
  ReactDOM.render(<IconSource />, wrapper);
}
```
这样虽然解决了引入模板的那个问题，但是后面又发现的 symbol 在安卓 4.3.x 下无法显示，也就是说 symbol 的兼容性并没有直接使用 svg 好。

然后我们通过使用一个叫 [svg4everybody](https://github.com/jonathantneal/svg4everybody) 的库，解决了上述兼容性问题。（它的原理是如果发现不支持 symbol 的，它会通过 xlink:href 拿到 svg 的资源，然后动态创建一个 svg,插入到当前位置）

虽然解决了兼容性的问题，但是我们深深的感觉到了这种方式的不优雅。

讲的这里，可能会有人会有疑问，既然已经有 [svg-react-loader](https://github.com/jhamlet/svg-react-loader) 了，为什么不直接 import svg 文件？

业务中使用的图片当然可以直接 import 加载，但一些通过的图标我们希望是能统一起来，做出组件，更方便的使用。

而且我们组件中还会对 svg 处理了它事件不能冒泡的问题，也就是在某些低版本的安卓机上，svg 图标是无法点击的。解决方案有两种：

* 贴膜，不过这样会导致多一层结构嵌套
* 去掉事件

```css
svg {
  pointer-events: none;
}
```

不过，这个问题可以给我带来启示，‘既然已经有 [svg-react-loader](https://github.com/jhamlet/svg-react-loader) 了’，那么 svg-loader 里做了什么呢？symbol 的方式或许真的可以淘汰了。

## “智人”之 svg
看下 [svg-react-loader](https://github.com/jhamlet/svg-react-loader) 的实现
首先有一个模板

```
  render () {
    const props = this.props;
      return (
        <svg {...props}>
          <%= innerXml %>
        </svg>
      );
    }
```

然后有一个 sanitize.js ，会对 svg 做一些处理，加上标准的 xml namespace, 把 React 特有的属性 class / for 转化为 className / htmlFor, 把属性名转化为驼峰。

最后根据模板生成这样一段代码

```javascript
var React = require('react');

module.exports = React.createClass({

    displayName: "Test",

    getDefaultProps () {
        return {"width":"1024","height":"1024","viewBox":"0 0 1024 1024","version":"1.1","xmlns":"http://www.w3.org/2000/svg"};
    },

    render () {
        var props = this.props;

        return <svg {...props}>
          <path d="M512.002047... fill="#272636"/>
        </svg>;
    }
});"
```

这样的代码我们就可以直接在 react 中直接使用了。

所以我们的组件借助这样的思想，完全弃用了 symbol 模式。

我们先扫描对应的 svg 文件，将其按上面的思路生成一个个单独的 js 文件。
在组件层面可以再封装一层，统一引入，提供一个通用的调用方式，和上面一样。

```javascript
    <Icon name="success" className="icon-success"/>
```

更好的是你也可以单独引用每一个文件，减小使用体积。

最后我们憧憬一下，随着 react 15.x 的发布，react 对 svg 的支持越来越好了，随着 IE 8 也即将被遗弃，我们的 PC 端也有望从 Iconfont 转换到 svg 了。 

