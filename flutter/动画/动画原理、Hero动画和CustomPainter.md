## 动画原理

无论是显式动画还是隐式动画，都是每一帧通过Ticker的回调，调用setState刷新页面。由于flutter对渲染的优化，即使每秒调用60次甚至120次setState都不会有很大的问题。

## Hero动画

就是一个页面的某个widget通过动画进入到另一个页面中去。

很简单，就是把在两个页面分别写上同一个widget，并且在这两个widget之外，包裹上一个`Hero`组件，`Hero`组件需要一个`tag`属性，需要传相同的值，告诉flutter，这两个widget是同一个。

栗子：

```dart
class HeroAnimationRoute extends StatelessWidget {
  HeroAnimationRoute({super.key});

  List<String> paths = [
    "hacker_avatar.png",
    "plus_icon.png",
    "sea.jpg",
    "send_icon.png",
    "thumb_up_icon.png"
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("HeroAnimationRoute"),
      ),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        crossAxisAlignment: CrossAxisAlignment.center,
        children: [
          imageWidget(context, paths[0]),
          imageWidget(context, paths[1]),
          imageWidget(context, paths[2]),
          imageWidget(context, paths[3]),
          imageWidget(context, paths[4]),
        ],
      ),
    );
  }

  Widget imageWidget(BuildContext context, String path) {
    String finalPath = "images/$path";
    return GestureDetector(
      onTap: () =>
          Navigator.push(context, MaterialPageRoute(builder: (context) {
        return HeroDetailRoute(finalPath);
      })),
      child: Hero(
          tag: finalPath,
          child: Image.asset(
            finalPath,
            width: 100,
          )),
    );
  }
}

class HeroDetailRoute extends StatelessWidget {
  String path;

  HeroDetailRoute(this.path, {super.key});

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
        onTap: () {
          Navigator.pop(context);
        },
        child: Container(
            width: double.infinity,
            height: double.infinity,
            color: Colors.white,
            child: Hero(tag: path, child: Image.asset(path))));
  }
}
```



## 直接操作底层的CustomPainter

其实就是跟Android的自定义View在onDraw里面使用canvas直接绘制一样。

示例代码：

```dart
class CustomPainterRoute extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return CustomPainterState();
  }
}

class CustomPainterState extends State<CustomPainterRoute>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  List<_Snowflake>? _snowflakes;

  @override
  void initState() {
    super.initState();
    _controller =
        AnimationController(duration: const Duration(seconds: 1), vsync: this)
          ..repeat();
    //处理所有❄️
    _snowflakes ??= List.generate(100, (index) {
      // debugPrint("生成新的SnowFlake");
      return _Snowflake(const Size(360, 700));
    });
  }

  @override
  void dispose() {
    super.dispose();
    _controller.dispose();
  }

  @override
  Widget build(BuildContext context) {

    return Scaffold(
      appBar: AppBar(
        title: const Text("CustomPainterRoute"),
      ),
      body: Center(
        child: Container(
          decoration: BoxDecoration(
              gradient: LinearGradient(
                  begin: Alignment.topCenter,
                  end: Alignment.bottomCenter,
                  colors: [Colors.blue[300]!, Colors.white],
                  stops: const [0.4, 0.9])),
          constraints: const BoxConstraints.expand(),
          child: AnimatedBuilder(
            animation: _controller,
            builder: (BuildContext context, Widget? child) {
              return CustomPaint(
                painter: _MyPainter(_snowflakes),
              );
            },
          ),
          // decoration: ,
        ),
      ),
    );
  }
}

class _MyPainter extends CustomPainter {

  List<_Snowflake>? _snowflakes;

  _MyPainter(this._snowflakes);

  final Paint _paint = Paint()..color = Colors.white;

  @override
  void paint(Canvas canvas, Size size) {
    //画☃️
    canvas.drawCircle(size.center(const Offset(0, 80)), 40, _paint);
    canvas.drawOval(
        Rect.fromCenter(
            center: size.center(const Offset(0, 250)), width: 150, height: 300),
        _paint);
    // debugPrint("_snowflakes:${_snowflakes.hashCode}");
    // 画❄️
    for (var snowflake in _snowflakes!) {
      canvas.drawCircle(Offset(snowflake.x, snowflake.y),
          snowflake.radius.toDouble(), _paint);
      snowflake.fall();
    }
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}

class _Snowflake {
  double x, y, speed = Random().nextDouble() * 4 + 2;
  double radius = Random().nextDouble() * 2 + 2;
  Size maxSize;

  _Snowflake(this.maxSize)
      : x = Random().nextDouble() * maxSize.width,
        y = Random().nextDouble() * maxSize.height;

  fall() {
    y += speed;
    if (y > maxSize.height.toInt()) {
      y = 0;
    }
    // debugPrint("snowflake:${toString()}");
  }

  @override
  String toString() {
    return '_Snowflake{x: $x, y: $y, speed: $speed, radius: $radius}';
  }
}
```