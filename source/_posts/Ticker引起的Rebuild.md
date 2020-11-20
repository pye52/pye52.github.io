---
title: Ticker引起的Rebuild
date: 2020-11-20 17:55:00
comments: true
tags: 
- flutter
- navigator
- ticker
---
Flutter应用在路由跳转时会遇到Rebuild场景。

当前文章Flutter示例运行版本：1.22.4

<!-- more -->

## 案例

当Widget混入TickerProvider或其子类后，若其通过Navigator进行界面跳转后或从其他界面返回，会导致页面Rebuild，如下所示：

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: FirstPage(),
    );
  }
}

class FirstPage extends StatefulWidget {
  @override
  _FirstPageState createState() => _FirstPageState();
}

class _FirstPageState extends State<FirstPage>
    with SingleTickerProviderStateMixin {
  @override
  void initState() {
    super.initState();
    TabController(length: 0, vsync: this);
  }

  @override
  Widget build(BuildContext context) {
    print("First Page build");
    return Scaffold(
      appBar: AppBar(),
      body: Column(children: [
        Container(
          height: 160,
          child: Text("First Page"),
        ),
        FlatButton(
          onPressed: () {
            print("Go!");
            Navigator.of(context).push(MaterialPageRoute(
              builder: (context) => SecondPage(),
            ));
          },
          child: Text("Go"),
        )
      ]),
    );
  }
}

class SecondPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    print("Second Page build");
    return Scaffold(
      appBar: AppBar(),
      body: Text("Second Page"),
    );
  }
}
```

运行后点击按钮跳转时，会有如下输出：

```console
I/flutter (31909): First Page build
I/flutter (31909): Go!
I/flutter (31909): Second Page build
I/flutter (31909): First Page build
// 点击AppBar的返回按钮
I/flutter (31909): First Page build
```

说明从FirstPage跳转到SecondPage，及从SecondPage返回到FirstPage时，FirstPage都会Rebuild一次。

## 原因

这个问题也被报告在Flutter的issue上：[issue1](https://github.com/flutter/flutter/issues/64444)、[issue2](https://github.com/flutter/flutter/issues/63312)

追根溯源，可以发现引起Rebuild的原因是`TickerMode`(涉及内容太多，此处省略跟踪源码的细节)，实际上，去掉`TabController`及`SingleTickerProviderStateMixin`，FirstPage修改成以下代码，仍然会Rebuild。

```dart
class FirstPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    TickerMode.of(context);
    print("First Page build");
    return Scaffold(
      appBar: AppBar(),
      body: Column(children: [
        Container(
          height: 160,
          child: Text("First Page"),
        ),
        FlatButton(
          onPressed: () {
            print("Go!");
            Navigator.of(context).push(MaterialPageRoute(
              builder: (context) => SecondPage(),
            ));
          },
          child: Text("Go"),
        )
      ]),
    );
  }
}
```

`TickerMode.of(context)`实际上是调用了`context.dependOnInheritedWidgetOfExactType<_EffectiveTickerMode>`，这导致`FirstPage`会跟随`TickerMode`一起Rebuild。而`TickerMode`源码是这样的(省略了无关代码)：

```dart
class TickerMode extends StatelessWidget {
    const TickerMode({
    Key key,
    @required this.enabled,
    this.child,
  }) : assert(enabled != null),
       super(key: key);
  ...
  @override
  Widget build(BuildContext context) {
    return _EffectiveTickerMode(
      enabled: enabled && TickerMode.of(context),
      child: child,
    );
  }
  ...
}
```

它是一个`StatelessWidget`，仅仅只是对child再包裹了一层`_EffectiveTickerMode`。而`_EffectiveTickerMode`是一个`InheritedWidget`，且其本身也只定义了一个`enabled`属性。

查阅[官方文档](https://api.flutter.dev/flutter/widgets/TickerMode-class.html)便能知道`TickerMode`作用是控制Widget树的tickers，简单点说就是控制Widget在可见和不可见时动画的播放及暂停(`enabled`的变动)。

但这个`TickerMode`到底是什么时候放到Widget树的呢？运行APP，检查Widget树，可以看到`MaterialApp`下的`Overlay`包裹着一个`TickerMode`，然后才是我们自己的`FirstPage`：

{% asset_img overlay.png Ticker %}

因此可以这么理解：

FirstPage -> SecondPage，FirstPage进入hide状态，`TickerMode`的`enabled`由true变为false，需要播放动画的终止帧

SecondPage -> FirstPage时，FirstPage进入show状态，`TickerMode`的`enabled`由false变为true，需要播放动画的起始帧

以上两种情况都需要Rebuild。

## 影响

由于Flutter独特的渲染机制，实际上这个Rebuild对性能影响并不大，并且该Rebuild并不是bug，真的只是个Feature罢了。但我们也需要了解该细节，以避免在build方法执行耗时较长的操作。尤其对于很多刚入门Flutter的新手，会在build方法里执行网络请求，这经常会导致奇怪的现象。

