### 适用场景

- 当设计稿是无最大大小限制（例如 1200px）的全屏设计时，字体和模块等一般都比较大。
- 此时如果以 px 写死，会照成小屏幕电脑（13 寸手提之类的）显示效果太过难看。
- 需要合理动态调整字体及模块大小。

### 规则

- 假设父元素字体大小为`16px`，子元素没有设置字体大小，则父元素自身与其子孙元素 `1em = 16px`

```html
<!--  1em = 16px，相当于margin: 16px-->
<div style="font-size: 16px;margin: 1em">
  <!--  1em = 16px，相当于padding: 16px-->
  <p style="padding: 1em"></p>
</div>
```

- 假设父元素字体大小为`16px`, 子元素字体大小设置`2em`，则子元素自身与其子孙元素 `1em = 32px`

```html
<!--  1em = 16px，相当于margin: 16px-->
<div style="font-size: 16px;margin: 1em">
  <!--  1em = 2× 16px，相当于padding: 64px-->
  <p style="font-size: 2em;padding: 2em"></p>
</div>
```

- 假设父元素字体大小为`12px`, 子元素字体大小设置`0.875em`，由于 chrome 默认最小字体是`12px`，所以子元素自身与其子孙元素 `1em = 12px`

### 缺陷

毕竟是会随着父元素的字体大小变更而变更，页面复杂的情况容易造成混乱。
相比之下，`rem`能解决次问题。

### 曾经项目中的做法

### `scss`语法

```css
// px to em
@function pxToEm($px, $pSize: 0px) {
  @if $pSize == 0px {
    @return $px / $emBase * 1em
  }
  @return $px / $pSize * 1em
}
// 设置在body class处，字体大小 12-16 变化
@mixin setFontSize() {
  font-size: 12px;

  @media screen and (min-width: 1320px) {
    font-size: 13px;
  }
  @media screen and (min-width: 1520px) {
    font-size: 14px;
  }
  @media screen and (min-width: 1720px) {
    font-size: 15px;
  }
  @media screen and (min-width: 1920px) {
    font-size: 16px;
  }
}

// 例子1： 设计稿某模块高度为 80px
.demo {
  height: pxToEm(80px)
}

// 例子2 设计稿某模块高度为 80px, 而该模块的标题字体大小为26px
.demo {
  font-size: pxToEm(26px)
  height: pxToEm(80px, 26px)
}
```

- 为什么不用`rem` ? 因为项目不是统一用`rem`作为单位，某些页面单独使用`rem`的话，由于要改变`html`的字体，怕对其他历史成品页面造成影响，故选用`em`
