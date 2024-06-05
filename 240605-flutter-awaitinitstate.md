# [Flutter](https://flutter.dev/): Waiting for an asynchronous operation in [initState](https://api.flutter.dev/flutter/widgets/State/initState.html)

Since the [initState](https://api.flutter.dev/flutter/widgets/State/initState.html) method is not asynchronous, you cannot use await within [initState](https://api.flutter.dev/flutter/widgets/State/initState.html).  
If your widget's build does not require data from an asynchronous operation,  
you can simply use the then method of the [Future class](https://api.dart.dev/stable/3.4.2/dart-async/Future-class.html).  
However, if you do need data from an asynchronous operation, how can you wait for it?  

> [!TIP]  
> In my case, I use the [Completer class](https://api.flutter.dev/flutter/dart-async/Completer-class.html) and [FutureBuilder class](https://api.flutter.dev/flutter/widgets/FutureBuilder-class.html) to address this issue.  
> Below is the sample code.

```dart
class _MyData {
  String? a;
  String? b;
}
class _MyHomePageState extends State<MyHomePage> {
  final _initCmp = Completer<_MyData>();

  Future<_MyData> _initState() async {
    var dat = _MyData();
    dat.a = await doSomething1();
    dat.b = await doSomething2();
    return dat;
  }

  @override
  void initState() {
    super.initState();
    _initState().then((dat) {
      _initCmp.complete(dat);
    }).catchError((err) {
      _initCmp.completeError(err);
    });
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<_MyData>(
      future: _initCmp.future,
      builder: (BuildContext context, AsyncSnapshot<_MyData> snapshot) {
        if (snapshot.hasData) {
          return SomeWidget(snapshot.data);
        } else if (snapshot.hasError) {
          return Center(
            child: Text('Error: ${snapshot.error}'),
          );
        } else {
          return const Center(
            child: Text('Awaiting result...'),
          );
        }
      },
    );
  }
}
```