```dart
import 'dart:async';
import 'dart:math';
import 'package:flutter/material.dart';

class GameUsingStream extends StatefulWidget {
  const GameUsingStream({super.key});

  @override
  State<StatefulWidget> createState() {
    return GameUsingStreamState();
  }
}

class GameUsingStreamState extends State<GameUsingStream> {
  final _inputStream = StreamController<int>.broadcast(); //多个监听的stream需要使用broadcast
  final _scoreStream = StreamController<int>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: DefaultTextStyle(
            style: Theme.of(context).textTheme.headlineMedium!,
            child: StreamBuilder<int>(
                stream: _scoreStream.stream.transform(ScoreTransformer()), //transform一个stream
                builder: (context, snapshot) {
                  if (snapshot.hasData) {
                    return Text(
                      "分数：${snapshot.data}",
                    );
                  }
                  return const Text("分数:0");
                }),
          ),
        ),
        body: Stack(children: [
          ...List.generate(4, (index) => Puzzle(_inputStream.stream, _scoreStream)),
          Align(alignment: Alignment.bottomCenter, child: KeyPad(_inputStream)),
        ]));
  }

  @override
  void dispose() {
    _inputStream.sink.close();
    super.dispose();
  }
}

class ScoreTransformer extends StreamTransformerBase<int, int> {

  int score = 0;
  final StreamController<int> _controller = StreamController();

  ///实现方式，接受一个stream，返回的transform后的stream。期间可以listen并做对应处理。
  @override
  Stream<int> bind(Stream<int> stream) {
    stream.listen((event) {
      score += event;
      _controller.add(score);
    });
    return _controller.stream;
  }

}

class Puzzle extends StatefulWidget {
  Stream<int> _input;
  StreamController<int> _score;

  Puzzle(this._input, this._score, {super.key});

  @override
  State<Puzzle> createState() => _PuzzleState();
}

///气球控件
class _PuzzleState extends State<Puzzle> with SingleTickerProviderStateMixin {
  int _a = 0;
  int _b = 0;
  double _x = 0;
  late Duration _duration;

  late Color _color;

  late AnimationController _animationController;

  @override
  void initState() {
    super.initState();

    _animationController = AnimationController(vsync: this);

    reset(Random().nextDouble());

    _animationController.addStatusListener((status) {
      //掉落下去还未处理，扣分
      if (status == AnimationStatus.completed) {
        reset();
        widget._score.add(-3); //给分数stream发分数改变的事件，简单做法直接把事件值当分数了
      }
    });

    widget._input.listen((event) {
      //输入了正确数字，加分
      if(event == _a+_b) {
        reset();
        widget._score.add(5);
      }
    });
  }

  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }

  ///重置气球，会重置整个气球，包括位置、颜色、速度
  void reset([animationFrom = 0.0]) {
    _a = Random().nextInt(4) + 1;
    _b = Random().nextInt(4);
    _x = Random().nextDouble();
    _color = Colors.primaries[Random().nextInt(Colors.primaries.length)];
    _duration = Duration(milliseconds: Random().nextInt(5000)+5000);
    _animationController.duration = _duration;
    _animationController.forward(from: 0);
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _animationController,
      builder: (BuildContext context, Widget? child) {
        return Positioned(
            top: MediaQuery.of(context).size.width * _animationController.value,
            left: (MediaQuery.of(context).size.width - 100) * _x,
            child: Container(
                decoration: BoxDecoration(
                    color: _color.withOpacity(0.5),
                    border: Border.all(color: Colors.black, width: 2),
                    borderRadius: BorderRadius.circular(24)),
                padding: const EdgeInsets.all(8),
                child: Text(
                  "$_a+$_b",
                  style: const TextStyle(fontSize: 30),
                )));
      },
    );
  }
}

//键盘控件
class KeyPad extends StatelessWidget {
  final StreamController _controller;

  const KeyPad(this._controller, {super.key});

  @override
  Widget build(BuildContext context) {
    return GridView.count(
        crossAxisCount: 3,
        childAspectRatio: 2.5,
        physics: const NeverScrollableScrollPhysics(),
        shrinkWrap: true,
        children: List.generate(9, (index) {
          return TextButton(
            style: ButtonStyle(
                backgroundColor:
                    MaterialStateProperty.all(Colors.primaries[index][500]),
                shape:
                    MaterialStateProperty.all(const RoundedRectangleBorder())),
            onPressed: () {
              _controller.add(index + 1); //点击后发送对应的数字事件
            },
            child: Text(
              "${index + 1}",
              style: const TextStyle(fontSize: 30, color: Colors.white),
            ),
          );
        }));
  }
}
```