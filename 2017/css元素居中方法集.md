#### 左右居中

- 行内( inline ) 或 inline\_\* 元素 （例如文本或者链接）[点我查看例子](http://jsbin.com/qexizalogu/1/edit?html,css,output)

```
.center-children {
  text-align: center;
}
```

- 单个块元素 [点我查看例子](https://jsbin.com/napolufivu/1/edit?html,css,output)

```
// 必须设置宽度（块元素默认100%宽度，无居中可言）
.center-me {
  width: 50%;
  margin: 0 auto;
}
```

- 多个块元素 [点我查看例子](https://jsbin.com/xawicoceze/edit?html,css,output)

```
.flex-center {
  display: flex;
  justify-content: center;
}

div {
  width: 100px;
  height: 100px;
  background-color: red;
  position: absolute;
  top:0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
}
```

#### 上下居中

- 行内( inline ) 或 inline\_\* 元素 （例如文本或者链接） [点我查看例子](https://jsbin.com/tupulerudu/3/edit?html,css,output)

```
.center-link {
  padding: 20px;
}

// 仅适用于单行上下居中
.center-text {
  height: 100px;
  line-height: 100px;
}
```

- 多行文字 [点我查看例子](https://jsbin.com/takiqiwexa/edit?html,css,output)

```
// 要声明父元素的高度
.parent {
  display: table;
  height: 200px; // or min-height
}
.parent .child {
  display: table-cell;
  vertical-align: middle;
}

// 要声明父元素的高度
.flex-parentNod {
  height: 200px; // or min-height
  display: flex;
  flex-direction: column;
  justify-content: center;
}
```

- 块元素 [点我查看例子](https://jsbin.com/xixepupepo/4/edit?html,css,output)

```
// 方法1
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  height: 100px;
  margin-top: -50px; // height的一半
}

// 方法2
 .parent {
   position: relative;
 }
 .child {
   position: absolute;
   top: 0;
   bottom: 0;
   margin: auto 0;
   height: 100px;
 }

 // 方法3
 .parent {
   display: flex;
   flex-direction: column;
   justify-content: center;
 }
```

#### 上下左右居中 [点我查看例子](https://jsbin.com/xixepupepo/edit?html,css,output)

```
// 方法1
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  left: 50%;
  height: 100px;
  width: 100px;
  margin-top: -50px; // height的一半
  margin-left: -50px; // width的一半
}

// 方法2
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  height: 100px;
}

// 方法3
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

// 方法4
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
```
