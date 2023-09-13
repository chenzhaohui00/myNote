WillPopScope组件用来拦截返回键，包括appBar上的返回以及设备的物理返回键，构造方法：

```dart
onst WillPopScope({
  ...
  required WillPopCallback onWillPop, //返回回调，在其中返回true或flase代表当前路由是否出栈
  required Widget child
})
```

onWillPop需要返回一个Future对象，使用时传入一个async函数即可，sample：

```dart
import 'package:flutter/material.dart';

class WillPopScopeTestRoute extends StatefulWidget {
  @override
  WillPopScopeTestRouteState createState() {
    return WillPopScopeTestRouteState();
  }
}

class WillPopScopeTestRouteState extends State<WillPopScopeTestRoute> {
  DateTime? _lastPressedAt; //上次点击时间

  @override
  Widget build(BuildContext context) {
    return WillPopScope(
      onWillPop: () async {
        if (_lastPressedAt == null ||
            DateTime.now().difference(_lastPressedAt!) > Duration(seconds: 1)) {
          //两次点击间隔超过1秒则重新计时
          _lastPressedAt = DateTime.now();
          return false;
        }
        return true;
      },
      child: Container(
        alignment: Alignment.center,
        child: Text("1秒内连续按两次返回键退出"),
      ),
    );
  }
}
```