## ~~无法为组件定义外部样式~~

**v1.3.4 版本已 fix 该问题**

> [issues 传送门](https://github.com/NervJS/taro/issues/3080)

```javascript
// Com.tsx
function Com() {
  return <View className="my-class">demo</View>;
}

Com.externalClasses = ["my-class"];

export default Com;
```

```javascript
// red-text样式不生效
<Com my-class="red-text" />
```

## taro doctor 不识别部分 ts 语法

> [issues 传送门](https://github.com/NervJS/taro/issues/3425)

```javascript
interface Itest {
  id: string // 不写;会报Parsing error: Unexpected token
  name: string
}
```

```javascript
// 关键字as报错，Parsing error: Unexpected token
let someValue: any = 'this is a string'
let strLength: number = (someValue as string).length
```

## ~~涉及小程序原生事件时，page 无法使用函数式写法~~

**作者反馈 hooks 和小程序原生钩子函数不兼容，所以涉及该类事件时，要用`Class`实现**

> [issues 传送门](https://github.com/NervJS/taro/issues/3054)

```javascript
function PageA() {
  // onPullDownRefresh等原生事件无处可写
  return <View>page</View>;
}

export default PageA;
```

## ~~在 render 组件中，无法在 map 中绑定事件~~

**map 中缺少 return 导致解析出错，应该会在 v1.3.5 中修复该问题**

> [issues 传送门](https://github.com/NervJS/taro/issues/3536)

```javascript
import Taro from "@tarojs/taro";
import { View, Block } from "@tarojs/components";

function Demo() {
  const data = [{ id: 1, val: "demo" }];
  const onClickDemo = () => {
    console.log(11);
  };
  const renderDemo = () => (
    <Block>
      {data.map(item => (
        // 不使用onClick则正常
        <View key={String(item.id)} onClick={onClickDemo}>
          {item.val}
        </View>
      ))}
    </Block>
  );

  return <View>{renderDemo()}</View>;
}

export default Demo;
```

## 以 children 形式传递 jsx 时的规则限定

```javascript
function Dialog(props) {
  return {
    // 必须使用this.props.children，解构方式的传参不会生效，例如const { children } = props;不会生效
    <View>{this.props.children}</View>
    // 必须以render开头，render后第一个字母大写
    <View>{this.props.renderOther}</View>
  }
}
```

```javascript
function Page() {
  return {
    <View>
      <Dialog renderOther={<View>2</View>}>
        <View>1</View>
      </Dialog>
    </View>
  }
}
```
