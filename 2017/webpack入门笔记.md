## [webpack](http://webpack.github.io/)

前端资源加载/打包工具,如图所示。
![what-is-webpack](https://cloud.githubusercontent.com/assets/16187601/17800914/78c2b034-6619-11e6-9c20-0127d54fe60b.png)

## 安装

- 安装 nodejs

```
git clone https://github.com/creationix/nvm.git ~/.nvm
source ~/.nvm/nvm.sh
nvm install v4.5.0
```

- 安装 webpack & webpack-dev-server

```
# 全局安装
npm install webpack  -g
npm install webpack-dev-server  -g

# 局部安装
npm install webpack --save-dev
npm install webpack-dev-server  --save-dev
```

## 配置实例

### 目录结构

![2016-08-19 16 00 53](https://cloud.githubusercontent.com/assets/16187601/17803008/294b48a6-6626-11e6-8c43-af6b721e44d7.png)

### package.json 配置

- 注：package.json 文件无法添加常规注释，复制黏贴请除去#开头内容

```
{
  # 项目名字
  "name": "Webpack demo",

  # 项目版本号
  "version": "0.0.1",

  # 项目描述
  "description": "A webpack demo",

  # 关键字
  "keywords": [
    "webview"
  ],

  # 仓库地址
  "repository": {
    "type": "git",
    "url": "git@git.liveneeq.com:skotc/adataFintech-webview.git"
  },

  # 作者信息
  "author": "https://github.com/liu-dongyu",

  # 自定义命令 npm run ***
  "scripts": {
    # npm run update 自动更新所需依赖的版本号
    "update": "ncu -a",

    # npm run build 构建打包文件
    "build": "NODE_ENV=prod webpack --config webpack.config.babel.js --progress --colors",

    # npm run start 开启webpack测试服务器
    "start": "NODE_ENV=dev webpack-dev-server --config webpack.config.babel.js"
  },

  # 开发环境下依赖
  "devDependencies": {
    # 以canIuse网站为依据自动为css属性添加浏览器前缀
    "autoprefixer": "^6.4.0",

    # 将es6,es7语法解析为浏览器可辨认的es5
    "babel-core": "^6.13.2",
    "babel-loader": "^6.2.5",
    "babel-polyfill": "^6.13.0",
    "babel-preset-es2015": "^6.13.2",
    "babel-preset-stage-0": "^6.5.0",
    "babel-register": "^6.11.6",

    # 辅助webpack解析css文件
    "css-loader": "^0.23.1",

    # 将css拆分为外部css，无此插件的话webpack会将css生成在页面<style>标签中
    "extract-text-webpack-plugin": "^1.0.1",

    # 自动生成相关html文件
    "html-webpack-plugin": "^2.22.0",

    # 辅助webpack解析json文件
    "json-loader": "^0.5.4",

    # scss依赖
    "node-sass": "^3.8.0",

    # 检查更新依赖的版本号
    "npm-check-updates": "^2.8.0",

    # postcss插件，未来css标准语法糖
    "postcss-cssnext": "^2.7.0",

    # postcss插件,格式化字体设置css
    "postcss-font-normalize": "0.0.0",

    # 使用postcss插件相关插件辅助webpack解析css文件
    "postcss-loader": "^0.10.1",

    # postcss插件,将css中 :: 转化为 :，兼容低版浏览器
    "postcss-pseudoelements": "^3.0.0",

    # postcss插件 , 辅助css动画效果
    "postcss-will-change": "^1.1.0",

    # 修正css背景图片路径url()异常问题
    "resolve-url-loader": "^1.6.0",

    # 辅助webpack解析scss语法
    "sass-loader": "^4.0.0",

    # 辅助webpack解析css文件
    "style-loader": "^0.13.1",

    # 辅助webpack解析图片，字体图标等文件
    "url-loader": "^0.5.7",

    # webpack依赖
    "webpack": "^1.13.2",

    # webpack区分开发环境和生成环境工具
    "webpack-config": "^6.1.2",

    # webpack测试开发服务器
    "webpack-dev-server": "^1.14.1",

    # 可用于生成生产环境时去除console alert等提示
    "webpack-strip": "^0.1.0"
  },

  # 生产环境下依赖
  "dependencies": {
    # 一般项目都会用到的js框架
    "jquery": "^3.1.0",

    # 所有项目都会用到，初始化css基本属性，保证在每个浏览器中体现一直
    "normalize.css": "^4.2.0"
  }
}

```

### .babelrc

- babel 相关配置

```
{
        # 设定转码规则
        # ES2015转码规则 & ES7语法提案的转码规则
    "presets": ["es2015", "stage-0"]
}
```

### webpack 基础配置文件 ./webpack.config.babel.js

```
'use strict';

import Config, { environment } from 'webpack-config';

environment.setAll({
  env: () => process.env.NODE_ENV
});

export default new Config().extend('./conf/webpack.[env].config.babel.js');

```

### webpack 生产环境与开发环境共用配置 .conf/webpack.base.config.js

```
'use strict';

// 引入node自带方法，用于路径操作
import path from 'path';

// 引入webpack依赖
import webpack from 'webpack';

// 引入区分生产环境和开发环境依赖
import Config from 'webpack-config';

// 引入自动生成html依赖
import HtmlWebpackPlugin from 'html-webpack-plugin';

// 引入提取入口公用文件或者打包时分离第三方包工具
import CommonsChunkPlugin from 'webpack/lib/optimize/CommonsChunkPlugin';

// 引入打包时生成独立.css文件依赖
import ExtractTextPlugin from 'extract-text-webpack-plugin';

// 入口文件配置
const entryPoints = {
  // 定义demo入口文件
  // babel-polyfill用于转换es6新的API，例如map
  demo: ['babel-polyfill', path.resolve(__dirname, '..', 'src', 'demo/demo.js')],

  // 定义第三方包，用于打包时分开打包第三方依赖
  vendors: ['jquery']
};

export default new Config().merge({
  // 入口配置
  entry: entryPoints,

  // 配置打包生成的地址以及生产文件的名字
  output: {
    path: path.resolve(__dirname, '..', 'dist'),

   // [name]代表入口文件名字，例如此处就是demo
    filename: '[name].js',

  // 图片等文件起始路径
  publicPath: '/'
  },

  resolve: {
    // js引入依赖时可省略不写的后缀
    extensions: ['', '.js', '.scss', '.css'],

   // 快速指向， js可直接引入对应目录下的文件
    modulesDirectories: ['node_modules', 'styles', 'static', 'src'],

    // 指定对应第三方包依赖的地址，优化webpack硬盘搜索
    // import 'normalize.css/normalize.css' 要写成   import 'normalize.css'
    alias: {
      'normalize.css': path.resolve(__dirname, '..', 'node_modules', 'normalize.css/normalize.css')
    }
  },

  // 定义webpack需要用到的插件
  plugins: [
    // 将vendors入口中的文件单独打包为vendors.js
    new CommonsChunkPlugin('vendors', 'vendors.js'),

    // 全局引入jquery
    new webpack.ProvidePlugin({
      $: 'jquery',
      jQuery: 'jquery',
      'window.jQuery': 'jquery'
    }),

    // 分离css，生成以入口命名的.css文件
    new ExtractTextPlugin('[name].css'),

   // 配置自动生成html的插件
    new HtmlWebpackPlugin({
      title: 'demo',  // 网页title
      hash: true,  // 时候为所有依赖添加hash值
      filename: 'demo.html',  // 生产的html名字
      inject: 'body',  // 依赖注入位置
      chunks: ['demo', 'vendors'], // 配置需要使用的js依赖， 对应上面已配置的入口文件
      template: path.resolve(__dirname, '..', 'template', 'demo.html')  // 文件模板,会基于该模板中已有的自动生成html文件
    })
  ],

  // 重头戏，配置webpack所需要的依赖
  module: {
    loaders: [
      {
        test: /(\.js)$/,  // 指定需要解析的文件类型，正则
        loaders: ['babel?cacheDirectory'], // 指定用什么来进行解析
        include: path.resolve(__dirname, '..', 'src') // 指定需要进行解析的文件目录地址
      },
      {
        test: /(\.scss)$/,
        include: path.resolve(__dirname, '..', 'styles'),
        loader: ExtractTextPlugin.extract('style', 'css!resolve-url!sass!postcss') // 第一个一定是style, 第二个值可以指定多个loader，使用!分割
      },
      {
        test: /\.css?$/,
        include: [path.resolve(__dirname, '..', 'node_modules')],
        loader: ExtractTextPlugin.extract('style', 'css')
      },
      {
        test: /\.(woff|woff2|eot|ttf|svg)(\?.*$|$)/,
        loader: 'url-loader?importLoaders=1&name=[name].[ext]'
      },
      {
        test: /\.(png|jpg)$/,
        loader: 'url-loader?limit=25000&name=[name]-[hash:base64:5].[ext]' // 图片大小不超过25K则转化为base64
      },
      {
        test: /\.json$/,
        loader: 'json'
      }
    ]
  },

  // 定义postcss需要用到什么插件，有些插件位置摆放有讲究，不能顺序颠倒，看具体插件文档描述
  postcss: () => [
    require('postcss-will-change'),
    require('postcss-cssnext'),
    require('postcss-pseudoelements'),
    require('postcss-font-normalize')
  ]
});
```

### webpack 生产环境配置 ./conf/webpack.prod.config.babel.js

```
'use strict';

import Config from 'webpack-config';
import path from 'path';
import webpack from 'webpack';
import WebpackStrip from 'webpack-strip';

export default new Config().extend('./conf/webpack.base.config.babel.js').merge({
  resolve: {
    // 定义所有第三方依赖min包位置
    alias: {
      'jquery': path.resolve(__dirname, '..', 'node_modules', 'jquery/dist/jquery.min.js')
    }
  },
  module: {
    loaders: [
      {
        test: /\.js/,
        loader: WebpackStrip.loader('console.log',  'alert'), // 使用WebpackStrip插件，除掉代码中的console.log 和 alert
        include: path.resolve(__dirname, '..', './styles')
      },
    ]
  },
  plugins: [
    // 定义当前环境为生产环境， 可在js中使用， process.env.NODE_ENV
    new webpack.DefinePlugin({
      'process.env': {
        'NODE_ENV': '"production"'
      }
    })
  ]
});

```

### webpack 开发环境配置 ./conf/webpack.dev.config.js

```
'use strict';

const webpack = require('webpack');
const WebpackConfig = require('webpack-config');

module.exports = new WebpackConfig().extend('./config/webpack.base.config.js').merge({
  // 配置开发服务器
  devServer: {
    hot: true, // 热加载关键
    inline: true, // 热加载关键
    host: '0.0.0.0', // 监听ip地址
    port: 3000 // 监听端口
  },
  plugins: [
    // 引用热加载插件，webpack自带
    new webpack.HotModuleReplacementPlugin()
  ],

  // 配置source-map类型， 有多种类型，具体看文档
  devtool: 'eval-source-map'
});
```

### demo 相关代码

- ./src/demo/demo.js

```
import 'normalize.css'
import 'demo/demo.scss'
import webpackImg from 'image/what-is-webpack.png'

$(document).ready(function(){
  var tpl = `<img class="set-img" src='${webpackImg}'>`;
  $('body').append(tpl);
});
```

- ./styles/demo/demo.scss

```
body {
  background-color: lightyellow;
}
strong {
  padding: 20px;
  display: block;
}
.set-img {
  width: 50%;
}
```

- ./template/demo.html

```
<!DOCTYPE html>
<html lang="zh-Hans">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="renderer" content="webkit">
  <meta name="screen-orientation" content="portrait">
  <meta name="format-detection" content="telephone=no">
  <meta name="nightmode" content="disable">
  <meta name="layoutmode" content="standard">
  <meta name="imagemode" content="force">
  <meta name="x5-orientation" content="portrait">
  <meta name="x5-page-mode" content="app">
</head>
<body>
<strong>welcome!</strong>
</body>
</html>
```

### 结果

`npm run start` 然后打开http://localhost:3000/demo.html，看见如图所示即为成功
![2016-08-19 16 56 21](https://cloud.githubusercontent.com/assets/16187601/17804363/e98359e0-662d-11e6-9fec-ed11705d9b1e.png)

`npm run build` 打包后在 dist 文件可看见如图所示
![2016-08-19 16 58 04](https://cloud.githubusercontent.com/assets/16187601/17804417/1c17dbf6-662e-11e6-99e1-c34471f3acd9.png)
