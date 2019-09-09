## 前言

Material 组件库提供了一套基础样式组件，例如`RaisedButton` `FlatButton`等。在实际开发场景中，除了个人项目，其他一般不直接使用默认样式，需要根据设计稿定制，所以本文记录如何制作适合自己的`button`组件

## 开始

### 制定`enum`

根据设计稿中按钮的样式总类，制定枚举类型，例如有红按钮和绿按钮

```dart
enum MyButtonType {
  RED, // 红按钮
  GREEN // 绿按钮
}
```

### 确定组件需要定制的样式，并指定能外部覆盖的属性

按钮存在高度、宽度、背景色、内边距、外边距、圆形边框等样式，将允许外部覆盖的样式写进构造方法中，且可以在构造方法中指定默认样式

```dart
final double width; // 宽度

/// 构造方法
const MyButton({
  Key key,
  this.width,
  }) : super(key: key);
```

### 根据枚举类型制定相关样式

一种枚举类型代表一种样式，通过`switch case`赋予对应属性

```dart
Color _backgroundColor;

switch(type) {
  case MyButtonType.RED:
    _backgroundColor = const Color(0xffff0000);
    break;

  case MyButtonType.GREEN:
    _backgroundColor = const Color(0xff00ff38);
    break;
}
```

### 选择要使用的按钮组件

`RaisedButton` `FlatButton`等都是直接或间接对`RawMaterialButton`组件的包装定制，故而选`RawMaterialButton`

## 完整代码

```dart
import 'package:flutter/material.dart';

enum MyButtonType {
  RED, // 红按钮
  GREEN // 绿按钮
}

class MyButton extends StatelessWidget {
  final MyButtonType type;
  final Widget child;
  final double width; // 宽度
  final double height; // 高度
  final VoidCallback onPressed; // 点击事件
  final EdgeInsetsGeometry margin; // 外边距
  final EdgeInsetsGeometry padding; // 内边距
  final TextStyle textStyle; // 文案整体样式覆盖
  final double textSize; // 文案尺寸
  final String text; // 文案

  const MyButton({
    Key key,
    this.type = MyButtonType.NORMAL,
    this.child,
    this.width,
    this.height = 44,
    this.onPressed,
    this.margin,
    this.padding = const EdgeInsets.symmetric(horizontal: 9.0),
    this.textSize = 15,
    this.textStyle,
    this.text,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    Color _backgroundColor;
    Color _textColor;

    switch (type) {
      case MyButtonType.RED:
        _backgroundColor = const Color(0xffff0000);
        _textColor = = const Color(0xffffffff);
        break;

      case MyButtonType.GREEN:
        _backgroundColor = const Color(0xff00ff38);
        _textColor = = const Color(0xffffffff);
        break;
    }

    Widget rlt = RawMaterialButton(
      shape: RoundedRectangleBorder(
        borderRadius: const BorderRadius.all(Radius.circular(5)),
      ),
      textStyle: textStyle ??
          TextStyle(
            fontSize: textSize,
            color: _textColor,
          ),
      constraints: BoxConstraints(),
      padding: padding,
      onPressed: onPressed,
      child: child ?? Text(text),
    );

    return Container(
      width: width,
      height: height,
      margin: margin,
      decoration: BoxDecoration(
        borderRadius: const BorderRadius.all(Radius.circular(5)),
        gradient: _gradient,
        color: _backgroundColor,
      ),
      child: rlt,
    );
  }
}

```
