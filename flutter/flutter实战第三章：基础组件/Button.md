## 概述

Material 库提供了一些标准组件：`ElevatedButton`、`TextButton`、`OutlineButton` 等，它们都是直接或间接对`RawMaterialButton`组件的包装和定制。

所有 Material 库中的按钮都有如下相同点：

1. 按下时都会有“水波动画”（又称“涟漪动画”，就是点击时按钮上会出现水波扩散的动画）。
2. 有一个`onPressed`属性来设置点击回调，当按钮按下时会执行该回调，如果不提供该回调则按钮会处于禁用状态，禁用状态不响应用户点击。

## 几个常用Button

### ElevatedButton

`ElevatedButton` 即"悬浮"按钮，它默认带有阴影和灰色背景。按下后，阴影会变大，示例：

```dart
ElevatedButton(
  child: Text("normal"),
  onPressed: () {},
);
```

### TextButton

`TextButton`即文本按钮，默认背景透明并不带阴影。按下后，会有背景色，示例：

```dart
TextButton(
  child: Text("normal"),
  onPressed: () {},
)
```

### OutlineButton

`OutlineButton`默认有一个边框，不带阴影且背景透明。按下后，边框颜色会变亮、同时出现背景和阴影(较弱)

```dart
OutlineButton(
  child: Text("normal"),
  onPressed: () {},
)
```

### IconButton

`IconButton`是一个可点击的Icon，不包括文字，默认没有背景，点击后会出现背景

```dart
IconButton(
  icon: Icon(Icons.thumb_up),
  onPressed: () {},
)
```

icon需要的是一个`Widget`，也可以用`Image.asset()`加载asset图片。

## 带图标的按钮

`ElevatedButton`、`TextButton`、`OutlineButton`都有一个`icon` 构造函数，通过它可以轻松创建带图标的按钮

```dart
ElevatedButton.icon(
  onPressed: () => debugPrint('点了悬浮按钮'),
  label: const Text('悬浮按钮'),
  icon: const Icon(Icons.send),
),
```