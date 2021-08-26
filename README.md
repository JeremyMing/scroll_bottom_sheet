# scrolle_bottom_sheet

解决当BottoSheet中含有listview等滚动组件时下拉退出无效问题

## 详细描述
[Flutter中由于滑动冲突导致BottomSheet中含有Listview等滚动组件时下拉效果失效问题](https://juejin.cn/post/6997774040236572680)

## 代码
```
class _MyHomePageState extends State<MyHomePage> {

  ScrollController _scrollController = ScrollController();

  /// 用来发送事件 改变弹框高度的stream
  StreamController<double> _streamController = StreamController<double>.broadcast();

  /// 列表弹起的正常高度
  double _totalHeight = 400;

  /// 记录手指按下的位置
  double _pointerDy = 0;

  void _incrementCounter() {
    showModalBottomSheet(
      barrierColor: Colors.black54,
      context: context,
      isScrollControlled: true,
      backgroundColor: Colors.white,
      shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.only(
              topLeft: Radius.circular(20), topRight: Radius.circular(20))),
      builder: (ctx) {
        return StreamBuilder<double>(
            stream: _streamController.stream,
            initialData: _totalHeight,
            builder: (context, snapshot) {
              double currentHeight = snapshot.data ?? _totalHeight;
              return AnimatedContainer(
                duration: Duration(milliseconds: 30),
                height: currentHeight,
                child: Listener(
                  onPointerMove: (event){
                    // 触摸事件过程 手指一直在屏幕上且发生距离滑动
                    if(_scrollController.offset != 0){
                      // 只有列表滚动到顶部时才触发下拉动画效果
                      print("onPointerMove:${_scrollController.offset}");
                      return;
                    }
                    double distance = event.position.dy - _pointerDy;
                    if (distance.abs() > 0) {
                      // 获取手指滑动的距离，计算弹框实时高度，并发送事件
                      double _currentHeight = _totalHeight - distance;
                      if(_currentHeight > _totalHeight){
                        return;
                      }
                      _streamController.sink.add(_currentHeight);
                    }
                  },
                  onPointerUp: (event){
                    // 触摸事件结束 手指离开屏幕
                    // 这里认为滑动超过一半就认为用户要退出了，值可以根据实际体验修改
                    if(currentHeight < (_totalHeight * 0.5)){
                      Navigator.pop(context);
                    }else{
                      _streamController.sink.add(_totalHeight);
                    }
                  },
                  onPointerDown: (event){
                    // 触摸事件开始 手指开始接触屏幕
                    _pointerDy = event.position.dy + _scrollController.offset;
                  },
                  child: ListView(
                    controller: _scrollController,
                    physics: currentHeight != _totalHeight ? NeverScrollableScrollPhysics() : ClampingScrollPhysics(),
                    children: [
                      ListTile(title: Text("11"),),
                      ListTile(title: Text("12"),),
                      ListTile(title: Text("13"),),
                      ListTile(title: Text("14"),),
                      ListTile(title: Text("15"),),
                      ListTile(title: Text("16"),),
                      ListTile(title: Text("17"),),
                      ListTile(title: Text("18"),),
                      ListTile(title: Text("19"),),
                      ListTile(title: Text("20"),),
                      ListTile(title: Text("21"),),
                      ListTile(title: Text("22"),),
                    ],
                  ),
                ),
              );
            }
        );
      },
    );
  }

  @override
  void dispose() {
    // TODO: implement dispose
    super.dispose();
    _streamController.close();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("scroll_bottom_sheet"),
      ),
      body: Center(

      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}
```
