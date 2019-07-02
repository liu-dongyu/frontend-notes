## [postCSS](https://github.com/postcss/postcss)

1. postCSS 直接对 css 文件进行转化，不需要额外的模板语言例如.scss 等
2. postCSS 提供了各种各样的插件(autoprefix, cssnext 等)，可自定义需要的功能

## 配合 webpack

### 安装依赖

```
npm install postcss-loader postcss-cssnext autoprefixer --save-dev
```

### 配置

```javascript
module: {
  loaders: [
    {
      test: /\.css?$/,
      loaders: ['style', 'css', 'postcss'],
    },
  ]
},
postcss: function() {
  return [require('postcss-cssnext')];
}
```

## 插件

1. [autoprefix](https://github.com/postcss/autoprefixer)，自动补全浏览器前缀

   ```javascript
   postcss: function() {
     return [require('autoprefix')];
   }
   ```

2. [postcss-cssnext](https://github.com/MoOx/postcss-cssnext)，允许你尝试未来的 css 属性，插件会自动转化为兼容浏览器的 css(已包含 autoprefix)

   ```javascript
   postcss: function() {
     return [require('postcss-cssnext')];
   }
   ```

3. [postcss-will-change](https://github.com/postcss/postcss-will-change)，为 `will-change` 属性添加回退

   ```javascript
   // 注意，要写在postcss-cssnext 或者 autoprefix 的后面
   postcss: function() {
     return [require('postcss-will-change')];
   }
   ```

4. [postcss-color-rgba-fallback](https://github.com/postcss/postcss-color-rgba-fallback)，给 `rgba()` 颜色创建降级方案

   ```javascript
   // 注意，要写在postcss-cssnext 或者 autoprefix 的后面
   postcss: function() {
     return [require('postcss-color-rgba-fallback')];
   }
   ```

5. [postcss-opacity](https://github.com/iamvdo/postcss-opacity)，给 `opacity` 提供降级方案

   ```javascript
   // 注意，要写在postcss-cssnext 或者 autoprefix 的后面
   postcss: function() {
     return [require('postcss-opacity')];
   }
   ```

6. [postcss-pseudoelements](https://github.com/axa-ch/postcss-pseudoelements)，将伪元素的 `::` 转换为 `:`

   ```javascript
   // 注意，要写在postcss-cssnext 或者 autoprefix 的后面
   postcss: function() {
     return [require('postcss-pseudoelements')];
   }
   ```

7. [node-pixrem](https://github.com/robwierzbowski/node-pixrem)，给 `rem` 添加 `px` 作为降级处理

   ```javascript
   // 注意，要写在postcss-cssnext 或者 autoprefix 的后面
   postcss: function() {
     return [require('pixrem')({
       browsers: ['ie >= 8', '> 1%', 'last 2 versions']
     })]
   }
   ```

8. [postcss-safe-parser](https://github.com/postcss/postcss-safe-parser)，修复存在的语法错误

   ```javascript
   postcss: function() {
     return {
       parser: require('postcss-safe-parser')
     }
   }
   ```

9. [postcss-flexbugs-fixes](https://github.com/luisrudge/postcss-flexbugs-fixes)，修复常见 `flexbox` 问题

   ```javascript
   // 注意，要写在postcss-cssnext 或者 autoprefix 的后面
   postcss: function() {
     return [require('postcss-flexbugs-fixes')];
   }
   ```

10. [postcss-flexibility](https://github.com/7rulnik/postcss-flexibility) [flexibility](https://github.com/10up/flexibility)，`flexbox` 兼容

    ```javascript
    module: {
    noParse: '../node_modules/flexibility/flexibility.js'
    }
    // 注意，要写在postcss-cssnext 或者 autoprefix 的后面
    postcss: function() {
      return [require('postcss-flexibility')];
    }
    ```

11. [doiuse](https://github.com/anandthakker/doiuse)，检查代码的浏览器兼容性

    ```javascript
    // 注意，要写在 postcss-font-normalize 前面
    postcss: function() {
      return [
        require('doiuse')({
          browsers: ['ie >= 10', 'last 2 versions'],
          // 要忽略不检查的属性，一定在要先做了相应属性的polyfill再填入这里
          ignore: [
            'css-appearance',
            'flexbox'
          ]
        })
      ]
    }
    ```

12. [postcss-font-normalize](https://github.com/iahu/postcss-font-normalize)，修正 `font-family` 格式

    ```javascript
    postcss: function() {
      return [require('postcss-font-normalize')];
    }
    ```

13. [postcss-reporter](https://github.com/postcss/postcss-reporter)，让 `postCss` 的插件生成的警告或者错误信息更好看

    ```javascript
    // 放在最后
    postcss: function() {
      return [require('postcss-reporter')({ clearMessages: true }),];
    }
    ```

> [优秀教程的传送门](http://www.w3cplus.com/blog/tags/516.html)
