> [项目地址](https://ticket.zspassenger.com.cn)

## 优化一

### 问题

打包后发现`vendors.js`过大，将近 600k，查看源码得知 vue-cli 使用`splitChunks`将所有 node_modules 中使用到的依赖打包到一起，所以优化方向为拆包以及去除无用依赖

```javascript
// vue-cli splitChunks配置代码片段
cacheGroups: {
  vendors: {
    test: /[\\/]node_modules[\\/]/,
    priority: -10
  },
}
```

### 过程

- 使用`webpack-bundle-analyzer`分析项目依赖

```javascript
// 在根目录下的vue.config.js添加，没有该文件自行新建
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer")
  .BundleAnalyzerPlugin;

module.exports = {
  chainWebpack: config => {
    if (process.env.NODE_ENV === "production") {
      config.plugin("report").use(BundleAnalyzerPlugin, [{}]);
    }
  }
};
```

- 根据`webpack-bundle-analyzer`结果发现，`vendors.js`中包含了`moments.js`，然而项目并没有使用该库，猜测由某个第三方库引入，全局搜索后发现由`pikaday.js`引入，通过`webpack.IgnorePlugin`移出构建，`vendors.js`减少 200k+

```javascript
// vue.config.js
const webpack = require("webpack");

module.exports = {
  chainWebpack: config => {
    // 解决pikaday自动引入moment
    config.plugin("ignore").use(webpack.IgnorePlugin, [/moment/]);
  }
};
```

- `pikaday.js`只被`Datepicker.vue`组件依赖，解决方法为将`Datepicker.vue`懒加载形成独立 chunk，并将`pikaday.js`库移出`vendors.js`

```javascript
// Search.vue
components: {
  Datepicker: () => import('@/components/Datepicker.vue'),
},

// vue.config.js
module.exports = {
  configureWebpack: {
    optimization: {
      splitChunks: {
        cacheGroups: {
          // 手动排除出vendor
          vendors: {
            test: /[\\/]node_modules[\\/](?!pikaday)/,
            priority: -10,
          },
        },
      },
    },
  },
};
```

4.由于监控平台工具`fundebug.js`使用 npm 管理，导致也被打包进`vendors.js`，最好的办法是使用 cdn 外部依赖，由于还没有自己的 cdn，先将它放在本地，在 html 中用过`script`引用，不参与打包过程

### 结果

`vendors.js`包大小缩减为 254K，相比原来缩减 57%

---

## 优化二

### 问题

首屏需要加载 507K 的 js 文件，响应速度不尽人意。  
有两种全局性的资源会在首屏中加载，一个是包含所有`node_modules`的`vendors.js`，一个是整体`App.vue`的依赖打包而成的`app.js`  
尝试优化`app.js`，缩减大小

### 过程

- 登录组件`loginBox.vue`需要全局使用，所以定义在`App.vue`，但并不希望首屏就加载该组件，希望是用户有登录要求才动态加载该组件，解决方法: 结合`v-if`以及组件动态加载实现

```javascript
// 当showLoginBox为true时，才会加载登录组件
<template>
  <LoginBox v-if="showLoginBox" />
</template>

<script>
components: {
  LoginBox: () => import('@/components/LoginBox.vue'),
}
</script>
```

- 全局头部组件`Header.vue`和全局底部组件`Footer.vue`相对较大，使用动态组件方式加载，形成独立 chunks

### 结果

`app.js`相比之前缩小了将 35.22K，但是首屏依旧需要加载 497k 的 js 文件，优化 2%，效果不理想

---

## 优化三

### 问题

偶然发现，全部资源`cache-control`都是设置为`max-age: 0`（如下图所示），显然没有好好利用 http 缓存机制，造成不必要的页面等待时间
<img src="https://user-images.githubusercontent.com/16187601/43114181-5329beac-8f30-11e8-9ffe-a9ce362ff911.jpeg" width="400"/>

### 方法

服务端基于 koa，使用[static-cache](https://github.com/koajs/static-cache)设置下静态资源`max-age`就可以了
