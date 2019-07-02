> 主要场景：微信  
> 技术栈：`vue + vue-router + vuex`

## 配置篇

### vuecli 中 runtimeCompiler 是什么

表示是否需要运行时编译器，默认`false`。一般来说不需要用到，因为`*.vue`文件内部的模版会在构建过程进行编译。当使用到`template`时，则需要用包含编译器的 vue 版本，该版本会额外增加 10KB(gzip)

```javascript
// 需要编译器
new Vue({
  template: "<div>hello word</div>"
});
```

### 开发环境怎么反向代理 API

反向代理即前端请求 A 服务器，A 服务器将该请求转发到目标 API 服务器，这么做主要是为了避免浏览器跨域问题，以及 API 服务地址可动态变更。可以在`vue.config.js`中进行如下配置

```javascript
module.exports = {
  // webpack自带的开发服务器，仅用于开发
  devServer: {
    // 请求标识，例如请求地址为https://demo.c/hmy/getCity
    "/hmy": {
      // 目标服务器地址
      target: "https://target.cn",
      // 将请求header中的origin改为目标服务器，不设置则为代理服务器
      changeOrigin: true,
      // 代理服务器转发时去除/hmy，所以实际会请求
      pathRewrite: {
        "^/hmy": ""
      }
    }
  }
};
```

### 怎么自定义环境变量

开发过程，有时希望在`vue.config.js`中添加自定义环境变量，从而根据不同指令得到不同的结果。

**方法 1**

`npm install cross-env --save-dev`（`cross-env`用于兼容不同操作系统）

```JavaScript
// package.json
"scripts": {
  // PRO_PATH则为自定义的环境变量
  // 在js中可以通过process.env.PRO_PATH获取
  // 在入口index.html中不可获取
  "pbuild": "cross-env PRO_PATH=1 npm run build",
}
```

**方法 2**

使用`mode`，`vuecil`默认存在`development`和`production`模式，允许添加自定义模式

1. 项目根目录处添加`.env.test`，表示`test`模式
2. `.env.test`文件中添加`VUE_APP_TEST=1`（前缀必须为`VUE_APP_`）。如不指定`NODE_ENV`，则默认`development`
3. js 中通过`process.env.VUE_APP_TEST`获取，入口`.html`通过`<%= VUE_APP_TEST %>`获取

## 布局篇

### 怎么做自适应

本项目主要场景是移动端（微信），需要全局适应不同设备的尺寸大小（小屏字更小，大屏字更大）。有三种方法供选择

- ~~使用`px`，不同分辨率使用`media query`进行变换~~
- 使用`rem`，基于淘宝[flexible.js](https://github.com/amfe/article/issues/17)
- 使用`vw`，viewport 单位，表示将屏幕均分 100 份，`1vw`就是 1 份

`flexible`主要干了两件事，根据写死的`dpr`(iOS 写死 1、2、3，android 写死 1)，动态设置缩放比（`meta viewport scale`）；动态设置`<html>`的`font-size`和`data-dpr`。由于`flexible`对安卓没做处理，且手淘（作者）已废弃该方案全面转向`vw`，故而本系统也拥抱`vw`

> 设备像素比(dpr)：物理像素 / 设备独立像素  
> 物理像素(pp): 只与硬件设备有关。 屏幕可视宽为 320 像素，非 retina 屏的 pp 是 320，retina 屏的则至少为 640  
> 设备独立像素(dip)：是个虚拟单位，在 web 世界中也称为 css 像素，屏幕可视宽为 320 像素，dip 也就是 320 像素

**具体做法**

- 设置`<meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no" />`
- 使用[postcss-px-to-viewport](https://github.com/evrone/postcss-px-to-viewport)，自动将`px`转为`vw`

```javascript
  'postcss-px-to-viewport': {
    viewportWidth: 750, // 设计稿宽度
    viewportHeight: 1334, // 设计稿高度
    unitPrecision: 5, // 指定`px`转换为视窗单位值的小数位数
    viewportUnit: 'vw', // 指定需要转换成的视窗单位
    selectorBlackList: ['.ignore'], // 指定不转换为视窗单位的类
    minPixelValue: 1, // 小于或等于`1px`不转换为视窗单位
    mediaQuery: false, // 允许在媒体查询中转换`px`
    exclude: [
      /demo/, // 不转化demo文件夹中的任何
    ],
  },
```

**vw fallback**

需要兼容不支持`vw`的机型，[viewport-units-buggyfill](https://github.com/rodneyrehm/viewport-units-buggyfill) [postcss-viewport-units](https://github.com/springuper/postcss-viewport-units)

```html
<!-- 入口html，body下添加 -->
<!-- 不喜欢阿里的cdn，也可以用自己的或download到本地 -->
<script src="//g.alicdn.com/fdilab/lib3rd/viewport-units-buggyfill/0.6.2/??viewport-units-buggyfill.hacks.min.js,viewport-units-buggyfill.min.js"></script>
<script>
  window.onload = function() {
    if (window.viewportUnitsBuggyfill && window.viewportUnitsBuggyfillHacks) {
      window.viewportUnitsBuggyfill.init({
        hacks: window.viewportUnitsBuggyfillHacks
      });
    }
  };
</script>
```

```javascript
// postcss.config.js
module.exports = {
  plugins: {
    // 如果使用了postcss-px-to-viewport，必须放到它后面
    'postcss-viewport-units': {
      filterFile: file => {
        return (
          // 排除文件路径含有/test/的文件
          !file.includes('/test/') &&
        );
      },
      filterRule: rule => {
        return (
          // 排除after和before选择器以及img
          !rule.selector.includes(':after') &&
          !rule.selector.includes('::after') &&
          !rule.selector.includes(':before') &&
          !rule.selector.includes('::before') &&
          !rule.selector.includes('img')
        );
      },
    },
  },
};
```

### 怎么处理 1px 线条问题

在`vw`方案中，如果没有特殊处理，`1px`的线条在 dpr 大于 1 的设备会比较粗。根本原因是`1px`可能会占用两个物理像素点或以上。  
[知乎专栏](https://zhuanlan.zhihu.com/p/38141740)  
[border-image 原理](https://www.zhangxinxu.com/wordpress/2010/01/css3-border-image/)

**具体做法**

- 使用[postcss-write-svg](https://github.com/jonathantneal/postcss-write-svg)，自动生成`1px`svg

```scss
// 1px.scss
$color: #ddd;

@svg 1px-border {
  width: 4px;
  height: 4px;
  @rect {
    fill: transparent;
    width: 100%;
    height: 100%;
    stroke-width: 25%;
    stroke: var(--color, black);
  }
}

@mixin setTopLine($c: $color) {
  border: 0;
  border-top: 1px solid;
  border-image: svg(1px-border param(--color $c)) 1 stretch;
}
```

```scss
// test.scss
@import "1px.scss" .test {
  @include setTopLine;
}
```

## vue 篇

### 构建公用组件可视化文档

使用[vue-styleguidist](https://github.com/vue-styleguidist/vue-styleguidist)可创建一个包含组件样式、用法等的 html，有利于多人协助时降低沟通成本和避免编写重复组件

- vuevli >= 3
- `npm install vue-styleguidist vue-styleguidist --save-dev`
- 项目根目录创建`styleguide.config.js`

```javascript
// styleguide.config.js
module.exports = {
  title: "Component guide", // 网站标题
  components: "src/components/**/[A-Z]*.vue", // 表示所有src/components下的.vue都会生成文档
  serverPort: "7070" // 监听的端口
};
```

```
// 目录结构
--src
  --components
    -- Demo.vue
    -- demo.md
```

```
// Demo.vue
<template>
  <div class="demo">
    Hello word
  </div>
</template>

<script>
/**
 * @example ./demo.md
 */
export default {
  name: 'Demo'
}
</script>
```

```md
# demo.md（要去掉空格）

# jsx 里可使用<tempalte><script><style>等

` ``jsx <Demo /> ` ``
```

- 运行`vue-cli-service styleguidist`
- 查看 127.0.0.1:7070

### 无限滚动列表

列表不断加载更多数据，由于 dom 元素剧增，如不做任何处理，性能问题是必然的。解决思路： 固定 dom 渲染数量，监听滚动位置，动态变更数据。[vue-virtual-scroller](https://github.com/Akryum/vue-virtual-scroller)

[Online Demo](https://codesandbox.io/embed/vue-template-cu5tr)

### 上拉加载更多

结合无限滚动列表，使用[vue-waypoint](https://github.com/scaccogatto/vue-waypoint)，监控加载更多标识出现。

[Online Demo](https://codesandbox.io/s/vue-template-cu5tr)

### 图片懒加载

当页面存在多张大图时，应当只加载可视区域内的图片，别的通过监听滚动条，出现在可视范围内再请求。1 来节省网络资源，2 来加快首屏加载速度。结合[loadz](https://github.com/ApoorvSaxena/lozad.js?files=1)，封装`Himg.vue`

[Online Demo](https://codesandbox.io/s/vue-template-repg5)

### 多个子元素 transition

`<tansition>`内单次只能有一个元素，多元素动画效果需要用到`<transition-group>`。在`<transition-group>`中，关键的一点就是要将即将消失的元素脱离文档流

[Online Demo](https://codesandbox.io/s/transition-group-r2n78)

### 如何将组件内容的全局弹窗移动到最外层（body）

在组件的世界里，弹窗一类的组件有可能存在于不同`.vue`中，这时该组件往往不在`body`下，为了避免`z-index`覆盖失效等问题，可以将此类组件通过自定义指令丢到`body`下。

[Online Demo](https://codesandbox.io/s/zidongjiangzujianbandaobodyxia-3t6k3)

## 优化篇

> vuecli 已基于 webpack 自带了很多优化，例如压缩、小图转`base64`、移除未引用代码(Tree Shaking)、代码分割（例如合并第三方依赖到 vendors）、`preload&prefetch`异步依赖等  
> [雅虎 32 军规](https://github.com/creeperyang/blog/issues/1)  
> 2-5-8 原则，用户访问某网站得到响应的时间(白屏时间)决定了留存，2 秒内用户爽，2-5 秒用户觉得还行，5-8 用户觉得超慢，8 秒后用户可能已离开

### dns prefetch

对于一定会使用到的非当前域名的资源，提前进行 DNS 解析，能细微的降低加载时间

```html
<!-- 写在html head内，例如 -->
<head>
  <link rel="dns-prefetch" href="//hm.baidu.com" />
</head>
```

### icon 图标

- 使用[iconfont](https://www.iconfont.cn/)，将图标制作为字体包，通过 css`@font-face`引入，能降低 http 请求数及 icon 资源大小，同时由于 font 特性，可自由切换颜色以及不失真放大缩小
- 由于 iconfont 的存在以及 webpack 会自动给小于某个大小（可配置）的图片自动转 base64 嵌套在页面中，个人并不推荐使用雪碧图，主要是比较难维护。
  - [Online CSS Sprites](https://www.toptal.com/developers/css/sprite-generator/)

```scss
/*
 * sprites.scss
 * 根据目标区域所需高度，计算雪碧图属性
 * $tarH: 目标区域高度
 * $imgW: 雪碧图对应属性宽度
 * $imgH: 雪碧图对应属性高度
 * $spritesW: 雪碧图宽度
 * $spritesH: 雪碧图高度
 * $posX: 对应属性在雪碧图中的x坐标
 * $posY: 对应属性在雪碧图中的y坐标
 */
@mixin setSpritesWithH(
  $tarH,
  $imgW,
  $imgH,
  $spritesW,
  $spritesH,
  $posX,
  $posY
) {
  $comp: $imgH / $tarH;
  width: $imgW / $comp;
  height: $tarH;
  background-size: ($spritesW / $comp) ($spritesH / $comp);
  background-position: ($posX / $comp) ($posY / $comp);
}
```

```scss
@import "sprites.scss" .demo-icon {
  background: url("./sprites.png");
  // 雪碧图的制作以及其中某个icon的位置信息从https://www.toptal.com/developers/css/sprite-generator/中获取
  // 配合vw方案，可实现动态变更icon大小
  @include setSpritesWithH(40px, 200, 137, 660, 641, -230, -339);
}
```

### 组件、页面懒加载

为了加快首屏响应速度，非本页面的代码以及弹窗一类需要用户触发的组件应该进行懒加载。 配合 webpack `code-splitting`，vue 和 vue-router 要实现起来非常简单。

**懒加载组件**

```JavaScript
<template>
  <div>
    <div v-if="show">
      <SomeCusComponnet />
    </div>
  </div>
</template>

// demo.vue
<script>
export default {
  name: 'Demo',
  data() {
    return { show: false };
  },
  components: {
    // 该组件会生成独立chunk，只在需要时才会加载该模块
    // 也就是当上面v-if="show"，show为true时才会从服务端加载对应chunk.js
    SomeCusComponnet: () => import('SomeCusComponnet'),
  },
};
</script>
```

当网络不好或者 async chunk 过大时，用户操作完后，可能需要消耗一定的时间才能将对应.js 拉取回来，造成体验不佳。针对该问题，vuecli 会对项目内所有异步引入（例如`import()`）的资源生成`prefetch`link，使得浏览器能提前下载该资源。  
如果对流量特别的敏感，不希望`prefetch`所有资源，可以进行关闭。

```JavaScript
// vue.config.js

module.exports = {
  chainWebpack: config => {
    // 取消prefetch资源
    config.plugins.delete('prefetch');
  },
};
```

```JavaScript
export default {
  name: 'Demo',
  components: {
    // 单独prefetch该资源
    SomeCusComponnet: () => import(/* webpackPrefetch: true */ 'SomeCusComponnet'),
  },
};
</script>
```

> prefetch: `<link href="demo.js" rel="prefetch">` 告诉浏览器请在空闲时间（由浏览器自己决定时间）提前下载该资源

**懒加载页面**

出于性能考虑，不希望 A 页面加载 B 页面的东西，仅进入 B 页面才加载对应资源，配合 vue-router，实现很简单

```JavaScript
const PageB = {
  path: '/b',
  name: 'b',
  // 会打包生产独立chunk，进入B页面时才会加载
  // 默认情况下，vuecli也会提前prefetch改资源
  component: () => import('B'),
};
```

### CDN

分布式的 CDN 能有效解决跨地域带来的网络消耗以及降低服务器压力等，如果公司穷逼买不起 CDN，可以酌情将例如 vue vux vue-router 等不变动的资源使用第三方免费 CND 引入，并添加本地 fallback

```html
// 入口index.html
<link rel="dns-prefetch" href="//cdn.bootcss.com" />
<script src="//cdn.bootcss.com/vue/2.6.10/vue.min.js"></script>
<script>
  // 当cdn资源失效时，加载本地资源
  window.Vue || document.write('<script src="./vue.min.js"><\/script>');
</script>
```

```javascript
// vue.config.js
module.exports = {
  // 在externals中定义外部依赖，不参与打包
  externals: {
    vue: "Vue",
    "vue-router": "VueRouter",
    vuex: "Vuex"
  }
};
```

```javascript
import Vue from "vue";
import Router from "vue-router";

// 如果vuex、vue-router也通过上述方式引入，则不需要显示得使用.use
// Vue.use(Router);  // no need
```

### 报错、监控等

[bugsnag](https://app.bugsnag.com)  
支持自定义错误内容、自动解析 sourecemap、记录用户报错时行为、邮件及 webhooks 报错等，免费版已够用

[mmtrix](http://www.mmtrix.com/)  
支持白屏、首屏等加载速度、资源大小、服务器网络分析、请求数量等在线测评，免费次数多，性能优化时可做参考

[oneapm](https://user.oneapm.com)  
记录浏览器分布、页面加载时间、地域分布、运营商加载速度、脚本错误、`ajax`调用次数响应时间等。一定功能免费版

## 微信篇

### 怎么在真机 or 微信开发者工具测试

如果需要用到例如微信登录等功能，只能使用已备案的域名（因为要在微信公众号内绑定），才能在真机或微信开发者工具内使用。由于开发过程总不能一直部署到服务器来测试，希望用本地的环境进行调试，所以需要用到内网穿透，简单说就是在你本地和内网穿透服务器间搭一条通道，访问内网穿透服务器的某个域名时，再转发到你本地。

内网穿透需要一台公网服务器和已备案的域名，笔者都没有。所以开发时使用了付费方案[natapp](https://natapp.cn)，也不贵，100m 带宽服务器 10 包月，1.1 元每 G，2 级域名 3 元包年..

### 微信开发工具中如何使用 vue-tool

[vue-devtools](https://github.com/vuejs/vue-devtools)

```js
// package.json
script: {
  // 通过命令行开启外部vue-devtool
  "devtools": "./node_modules/.bin/vue-devtools"
}
```

```html
<!-- 入口html  -->
<% if (NODE_ENV === 'development') { %>
<!-- 开启外部vue-devtool，用于在微信开发者工具内调试vue -->
<script src="http://localhost:8098"></script>
<% } %>
```
