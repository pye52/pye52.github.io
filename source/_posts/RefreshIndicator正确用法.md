---
title: RefreshIndicator正确用法
date: 2020-11-23 17:10:00
comments: true
tags: 
- flutter
- refreshindicator
---

`RefreshIndicator`是Flutter用于下拉刷新的控件，但在数据需要异步请求时，则存在一些常见的误区。

当前文章Flutter示例运行版本：1.22.4

<!-- more -->

#### 场景一——`FutureBuilder`：

```dart
import 'dart:math';

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
      home: HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  List<String> _data;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: FutureBuilder<List<String>>(
        future: _fetchData(),
        builder: (context, snapshot) {
          if (snapshot.connectionState != ConnectionState.done) {
            return Container();
          } else {
            _data = snapshot.data;
            return RefreshIndicator(
              onRefresh: () {
                setState(() {});
                return Future.value();
              },
              child: ListView.separated(
                itemBuilder: (context, index) => Container(
                  height: 60,
                  child: Container(
                    padding: const EdgeInsets.all(8),
                    color: Colors.grey,
                    child: Text(_data[index]),
                  ),
                ),
                separatorBuilder: (context, index) => SizedBox(height: 8),
                itemCount: _data.length,
              ),
            );
          }
        },
      ),
    );
  }

  Future<List<String>> _fetchData() async {
    await Future.delayed(Duration(seconds: 1));
    return List.generate(Random().nextInt(20), (index) => "Test $index");
  }
}
```

#### 场景二——直接请求：

```dart
import 'dart:math';

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
      home: HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  List<String> _data;

  @override
  void initState() {
    super.initState();
    _fetchData();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: RefreshIndicator(
        onRefresh: () {
          _fetchData();
          return Future.value();
        },
        child: ListView.separated(
          itemBuilder: (context, index) => Container(
            height: 60,
            child: Container(
              padding: const EdgeInsets.all(8),
              color: Colors.grey,
              child: Text(_data[index]),
            ),
          ),
          separatorBuilder: (context, index) => SizedBox(height: 8),
          itemCount: _data?.length ?? 0,
        ),
      ),
    );
  }

  void _fetchData() async {
    await Future.delayed(Duration(seconds: 1));
    setState(() {
      _data = List.generate(Random().nextInt(20), (index) => "Test $index");
    });
  }
}
```

#### 解决方案——`StreamBuilder`：

无论是以上的哪种方案，要么`RefreshIndicator`的动画未完成界面便Rebuild了(方案一)，要么`RefreshIndicator`动画完成后一段时间数据才刷新(方案二)。究其原因就是`setState`和`onRefresh`是互斥的。

通过`StreamBuilder`，则可以实现最完美的效果：

```dart
import 'dart:async';
import 'dart:math';

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
      home: HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final StreamController<List<String>> _streamController = StreamController();

  @override
  void initState() {
    super.initState();
    _init();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: RefreshIndicator(
        onRefresh: () async {
          List<String> data = await _fetchData();
          _streamController.sink.add(data);
          return Future.value();
        },
        child: StreamBuilder(
          stream: _streamController.stream,
          builder: (context, snapshot) {
            List<String> _data = snapshot.data;
            return ListView.separated(
              itemBuilder: (context, index) => Container(
                height: 60,
                child: Container(
                  padding: const EdgeInsets.all(8),
                  color: Colors.grey,
                  child: Text(_data[index]),
                ),
              ),
              separatorBuilder: (context, index) => SizedBox(height: 8),
              itemCount: _data?.length ?? 0,
            );
          },
        ),
      ),
    );
  }

  void _init() async {
    List<String> data = await _fetchData();
    _streamController.sink.add(data);
  }

  Future<List<String>> _fetchData() async {
    await Future.delayed(Duration(seconds: 1));
    return List.generate(Random().nextInt(20), (index) => "Test $index");
  }

  @override
  void dispose() {
    super.dispose();
    _streamController.close();
  }
}
```