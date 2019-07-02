## 背景

项目使用`vw`作为基本单位，页面需要自适应 980px 到 1920px，希望当小于 980 或大于 1920 时使用`px`单位替换`vw`。在不借助 Javascript 的情况下，只能使用`@media`针对性处理，但为每个`vw`写对应的`@media`显得很繁碎，希望能自动化这个过程。

```css
// 输入
.demo {
  width: 10vw;
}

// 输出
.demo {
  width: 10vw;
}
@media (max-width: 980) {
  .demo {
    width: 98px;
  }
}
@media (min-width: 1920) {
  .demo {
    width: 192px;
  }
}
```

## 解决方案

借助[postcss](https://github.com/postcss/postcss)在编译过程实现，由于需求具备特殊性，翻遍了 google 也找不到相关插件，故而只能自己写

## 一些概念

postcss 是一款 css 预编译器，通过解析 css 文件，将相关内容转化为 ast 抽象语法树，用户可以针对该语法树做增删改操作

ast 抽象语法树顶层为 root，二层为 atRult rule 等，三层为 decl

```css
// 根部就是root层

// 一般的匹配方式就是rule层
.demo {
  width: 10vw; // 相关规则就是decl层
}

// 带有@开头的就是atRule层
@media (min-width: 1920) {
}
```

## 一些 API

- `walkRules`，遍历所有 rule，第一个参数接受正则，表示只匹配符合的规则，第二个参数是 callback，返回当前正在处理的 rule
- `walkDecls`，遍历所有 decl，第一个参数接受正则，第二个参数是 callback，返回当前正在处理的 decl
- 更多 API 参考[postcss-api](http://api.postcss.org/)

## 如何实现

```javascript
const postcss = require("postcss");
// lodash不是必要的，可以不用
const _ = require("lodash");

// postcss-viewport-to-px代表插件名
module.exports = postcss.plugin("postcss-viewport-to-px", () => {
  // 回调函数返回当前ast抽象语法数
  return css => {
    // 统计该语法树中一共需要多少media
    const medias = [];

    // 遍历rule层
    css.walkRules(rule => {
      // 相同rule层下的decl放到一起
      const decl1920 = [];
      const decl980 = [];

      // 只处理顶层root，包裹在类似media中的vw不处理
      if (rule.parent.type !== "root") {
        return;
      }

      // 遍历当前rule层中的decl规则
      rule.walkDecls(decl => {
        // 碰到vw/font-size/calc都不做处理
        // decl.value表示内容，代表width: 10px中的10px
        // decl.prop表示属性名，代表width: 10px中的width
        if (
          decl.value.indexOf("vw") === -1 ||
          decl.prop.indexOf("font-size") !== -1 ||
          decl.value.indexOf("calc") !== -1
        ) {
          return;
        }

        // 将vw转为px
        // 由于有可能是padding: 10vw 1vw 2vw 3vw这种形式，所以按照空格切分
        const vals = decl.value.split(/\s/);
        let str1920 = "";
        let str980 = "";
        // 遍历切换得出的数组，只处理vw单位的内容
        _.forEach(vals, (val, index) => {
          if (val.indexOf("vw") === -1) {
            str1920 += val;
            str980 += val;
          } else {
            let tmp = Number(val.replace("vw", ""));
            // 由于当前项目的设计稿宽度为1920px，所以100vw = 1920px
            str1920 += `${((tmp * 1920) / 100).toFixed(2)}px`;
            str980 += `${((980 / 100) * tmp).toFixed(2)}px`;
          }
          if (index !== vals.length - 1) {
            str1920 += " ";
            str980 += " ";
          }
        });

        // 保存相同rule层中的decl规则
        decl1920.push({
          value: str1920,
          prop: decl.prop
        });
        decl980.push({
          value: str980,
          prop: decl.prop
        });
      });

      // 排除没有匹配decl的rule层
      if (_.isEmpty[decl1920] || _.isEmpty(decl980)) {
        return;
      }

      // 将rule层放进meida，稍后统一处理
      medias.push(
        {
          parent: rule.parent, // 父层
          selector: rule.selector, // selector代表选择器，类似.demo {}
          decls: decl980,
          source: rule.source, // source表示来源代码，也就是你写的css代码
          media: "(max-width: 980px)"
        },
        {
          parent: rule.parent,
          selector: rule.selector,
          source: rule.source,
          media: "(min-width: 1920px)",
          decls: decl1920
        }
      );
    });

    _.forEach(medias, m => {
      // 生成media容器
      const atRule = postcss.atRule({
        name: "media",
        params: m.media,
        source: m.source
      });

      // 生成css空白代码片段
      const mediaRule = postcss.rule({
        selector: m.selector
      });

      // 为rule添加具体规则
      _.forEach(m.decls, decl => {
        mediaRule.append({
          prop: decl.prop,
          value: decl.value
        });
      });
      // 将相关规则放进media中
      atRule.append(mediaRule);

      // 添加到css语法树中
      m.parent.prepend(atRule);
    });
  };
});
```

## 还有什么问题

当前代码会生成过多 media 语法，如下代码，需要借助[css-mqpacker](https://github.com/hail2u/node-css-mqpacker)合并相同的`media`

```css
.demo {
  width: 10vw;
}
.demo1 {
  width: 10vw;
}

// 输出
.demo {
  width: 10vw;
}
.demo1 {
  width: 10vw;
}
@media (max-width: 980) {
  .demo {
    width: 98px;
  }
}
@media (min-width: 1920) {
  .demo {
    width: 192px;
  }
}
@media (max-width: 980) {
  .demo1 {
    width: 98px;
  }
}
@media (min-width: 1920) {
  .demo1 {
    width: 192px;
  }
}
```
