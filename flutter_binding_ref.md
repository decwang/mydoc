前言
Flutter中所有的运转都是在各种Binding中调度的，也正是这些绑定器的存在彻底解耦了Widget 、 Element 、RenderObject 对 Platform端的依赖，阅读此文需要有一定的Flutter基础，如：Flutter的绘制流程、Flutter与Platform通信原理、Flutter 三棵树各自的职责。

WidgetsFlutterBinding
WidgetsFlutterBinding是一个单例存，它继承了GestureBinding, SchedulerBinding, ServicesBinding, PaintingBinding, SemanticsBinding, RendererBinding, WidgetsBinding，负责Widget和Engine之间的粘合工作

class WidgetsFlutterBinding extends BindingBase with GestureBinding, SchedulerBinding, ServicesBinding, PaintingBinding, SemanticsBinding, RendererBinding, WidgetsBinding {
  static WidgetsBinding ensureInitialized() {
    if (WidgetsBinding.instance == null)
      WidgetsFlutterBinding();
    return WidgetsBinding.instance!;
  }
}
所有的Binding都是为上层的Widget，Element，RenderObject提供调用的，如下图所示：
168b72f9b2e14d8fcc7c6c2e293709f2.png
image-20220223100108329

事件调度绑定器 — GestureBinding
GestureBinding是对Flutter中的事件分发管理，具体可以参考浅谈Flutter核心机制之— 事件分发
https://blog.csdn.net/qq_38401950/article/details/123071402

1. 注册Engine 层的事件监听 ,onPointerDataPacket是Engine层回调的一个方法

  void initInstances() {
    super.initInstances();
    _instance = this;
    window.onPointerDataPacket = _handlePointerDataPacket;
  }
2. 分发事件到RenderObject中 dispatchEvent

  void dispatchEvent(PointerEvent event, HitTestResult? hitTestResult) {
 ...
    for (final HitTestEntry entry in hitTestResult.path) {
      try {
        entry.target.handleEvent(event.transformed(entry.transform), entry);
      } catch (exception, stack) {
      }
    }
  }
3. 处理事件方法handleEvent, 处理所有RenderObject中注册的手势识别器,进行竞技

  @override // from HitTestTarget
  void handleEvent(PointerEvent event, HitTestEntry entry) {
    pointerRouter.route(event);
    if (event is PointerDownEvent) {
      gestureArena.close(event.pointer);
    } else if (event is PointerUpEvent) {
      gestureArena.sweep(event.pointer);
    } else if (event is PointerSignalEvent) {
      pointerSignalResolver.resolve(event);
    }
  }
任务调度绑定器 — SchedulerBinding
SchedulerBinding是任务调度器，它负责处理对各种类型任务调度的时机, 执行 UI构建前/UI构建后的一些任务，除此之外还可以对任务进行优先级排序

1. handleBeginFrame是执行里面scheduleFrameCallback注册的回调

  void handleBeginFrame(Duration? rawTimeStamp) {
    Timeline.startSync('Frame', arguments: timelineArgumentsIndicatingLandmarkEvent);
    _firstRawTimeStampInEpoch ??= rawTimeStamp;
    _currentFrameTimeStamp = _adjustForEpoch(rawTimeStamp ?? _lastRawTimeStamp);
    if (rawTimeStamp != null)
      _lastRawTimeStamp = rawTimeStamp;
 
    _hasScheduledFrame = false;
    try {
      // TRANSIENT FRAME CALLBACKS
      Timeline.startSync('Animate', arguments: timelineArgumentsIndicatingLandmarkEvent);
      _schedulerPhase = SchedulerPhase.transientCallbacks;
      final Map<int, _FrameCallbackEntry> callbacks = _transientCallbacks;
      _transientCallbacks = <int, _FrameCallbackEntry>{};
      callbacks.forEach((int id, _FrameCallbackEntry callbackEntry) {
        if (!_removedIds.contains(id))
          _invokeFrameCallback(callbackEntry.callback, _currentFrameTimeStamp!, callbackEntry.debugStack);
      });
      _removedIds.clear();
    } finally {
      _schedulerPhase = SchedulerPhase.midFrameMicrotasks;
    }
  }

2. handleDrawFrame是执行addPersistentFrameCallback/addPostFrameCallback中注册的回调,UI流水线构建的回调也在里面执行

  void handleDrawFrame() {
    assert(_schedulerPhase == SchedulerPhase.midFrameMicrotasks);
    Timeline.finishSync(); // end the "Animate" phase
    try {
      // PERSISTENT FRAME CALLBACKS
      _schedulerPhase = SchedulerPhase.persistentCallbacks;
      for (final FrameCallback callback in _persistentCallbacks)
        _invokeFrameCallback(callback, _currentFrameTimeStamp!);
 
      // POST-FRAME CALLBACKS
      _schedulerPhase = SchedulerPhase.postFrameCallbacks;
      final List<FrameCallback> localPostFrameCallbacks =
          List<FrameCallback>.from(_postFrameCallbacks);
      _postFrameCallbacks.clear();
      for (final FrameCallback callback in localPostFrameCallbacks)
        _invokeFrameCallback(callback, _currentFrameTimeStamp!);
    } finally {
      _schedulerPhase = SchedulerPhase.idle;
      Timeline.finishSync(); // end the Frame
      assert(() {
        if (debugPrintEndFrameBanner)
          debugPrint('▀' * _debugBanner!.length);
        _debugBanner = null;
        return true;
      }());
      _currentFrameTimeStamp = null;
    }
  }

3. GPU光栅化耗时回调 – addTimingsCallback，可以用来做GPU耗时检测

  void addTimingsCallback(TimingsCallback callback) {
    _timingsCallbacks.add(callback);
    if (_timingsCallbacks.length == 1) {
      assert(window.onReportTimings == null);
      window.onReportTimings = _executeTimingsCallbacks;
    }
    assert(window.onReportTimings == _executeTimingsCallbacks);
  }
4. 优先级执行异步任务 – scheduleTask

  Future<T> scheduleTask<T>(TaskCallback<T> task,Priority priority, {String? debugLabel, Flow? flow,}) {
    final bool isFirstTask = _taskQueue.isEmpty;
    final _TaskEntry<T> entry = _TaskEntry<T>(task, priority.value, debugLabel, flow,);
    _taskQueue.add(entry);
    if (isFirstTask && !locked)
      _ensureEventLoopCallback();
    return entry.completer.future;
  }
5. 在下一帧构建任务前的任务 – scheduleFrameCallback,在handleBeginFrame中调用(参考1)，例如: 动画中的数据值更新 （ps：能够被cancelFrameCallbackWithId方法取消）

  int scheduleFrameCallback(FrameCallback callback, { bool rescheduling = false }) {
    scheduleFrame();
    _nextFrameCallbackId += 1;
    _transientCallbacks[_nextFrameCallbackId] = _FrameCallbackEntry(callback, rescheduling: rescheduling);
    return _nextFrameCallbackId;
  }
6. 永久回调任务addPersistentFrameCallback，drawFrame的任务就是注册在此方法中。ps：注册了该回调每次下一帧之前都会执行一次

  void addPersistentFrameCallback(FrameCallback callback) {
    _persistentCallbacks.add(callback);
  }
7. addPostFrameCallback在addPersistentFrameCallback之后，只会被调用一次，在handleDrawFrame中调用

 void addPostFrameCallback(FrameCallback callback) {
    _postFrameCallbacks.add(callback);
  }
8. scheduleFrame()通知Engine有UI更新需要被回调

  void scheduleFrame() {
    if (_hasScheduledFrame || !framesEnabled)
      return;
    ensureFrameCallbacksRegistered();
    window.scheduleFrame();
    _hasScheduledFrame = true;
  }
服务绑定器 — ServicesBinding
ServicesBinding注册管理了一些平台服务，如消息通信信使_defaultBinaryMessenger，平台的各种生命周期等等 

void initInstances() {
    super.initInstances();
    _instance = this;
    //与platform通信的信使
    _defaultBinaryMessenger = createBinaryMessenger();
    //状态恢复回调
    _restorationManager = createRestorationManager();
    window.onPlatformMessage = defaultBinaryMessenger.handlePlatformMessage;
    initLicenses();
    //系统消息处理，如内存低 。。
    SystemChannels.system.setMessageHandler((dynamic message) => handleSystemMessage(message as Object));
    //生命周期
    SystemChannels.lifecycle.setMessageHandler(_handleLifecycleMessage);
    readInitialLifecycleStateFromNativeWindow();
  }

1. _defaultBinaryMessenger是负责与platform通信，详情可参考Flutter中MethodChannel/EventChannel的原理,两个和新方法如下：

//发送二进制消息发给platform
  Future<ByteData?> _sendPlatformMessage(String channel, ByteData? message) {
    final Completer<ByteData?> completer = Completer<ByteData?>();
    ui.PlatformDispatcher.instance.sendPlatformMessage(channel, message, (ByteData? reply) {
      try {
        completer.complete(reply);
      } catch (exception, stack) {
    });
    return completer.future;
  }
 
//处理platform发过来的二进制消息
  @override
  Future<void> handlePlatformMessage(
    String channel,
    ByteData? data,
    ui.PlatformMessageResponseCallback? callback,
  ) async {
    ByteData? response;
    try {
      final MessageHandler? handler = _handlers[channel];
      if (handler != null) {
        response = await handler(data);
      } else {
        ui.channelBuffers.push(channel, data, callback!);
        callback = null;
      }
    } 
  }

2. handleSystemMessage处理系统消息，如字体改变、内存不足

  //该方法被子类重写了
  @protected
  @mustCallSuper
  Future<void> handleSystemMessage(Object systemMessage) async {
    final Map<String, dynamic> message = systemMessage as Map<String, dynamic>;
    final String type = message['type'] as String;
    switch (type) {
      case 'memoryPressure':
        handleMemoryPressure();
        break;
    }
    return;
  }
3. _parseAppLifecycleMessage生命状态回调的处理

  static AppLifecycleState? _parseAppLifecycleMessage(String message) {
    switch (message) {
      case 'AppLifecycleState.paused':
        return AppLifecycleState.paused;
      case 'AppLifecycleState.resumed':
        return AppLifecycleState.resumed;
      case 'AppLifecycleState.inactive':
        return AppLifecycleState.inactive;
      case 'AppLifecycleState.detached':
        return AppLifecycleState.detached;
    }
    return null;
  }
4. RestorationManager 数据保存/恢复管理

@protected
RestorationManager createRestorationManager() {
  return RestorationManager();
}
图像绑定器 — PaintingBinding
PaintingBinding比较简单,就是管理了Flutter图像缓存和创建图像编解码器,以及对GPU着色器程序预热

辅助服务绑定器 — SemanticsBinding
SemanticsBinding处理Platform上辅助服务事件

渲染绑定器 — RendererBinding
RendererBinding是一个核心组件，它里面管理了渲染管线PipelineOwner(管理RenderObject)，以及注册platform显示相关的监听,如：亮度改变，字体缩放因子改变等等，它里面创建了第一个RenderObject ---- RenderView

1. initInstances 初始化

void initInstances() {
    super.initInstances();
    _instance = this;
    //渲染管线，管理需要刷新的RenderObject
    _pipelineOwner = PipelineOwner(
      onNeedVisualUpdate: ensureVisualUpdate,
      onSemanticsOwnerCreated: _handleSemanticsOwnerCreated,
      onSemanticsOwnerDisposed: _handleSemanticsOwnerDisposed,
    );
    window
      ..onMetricsChanged = handleMetricsChanged
      ..onTextScaleFactorChanged = handleTextScaleFactorChanged
      ..onPlatformBrightnessChanged = handlePlatformBrightnessChanged
      ..onSemanticsEnabledChanged = _handleSemanticsEnabledChanged
      ..onSemanticsAction = _handleSemanticsAction;
      //创建第一个 RenderObject  ---- RenderView
    initRenderView();
    _handleSemanticsEnabledChanged();
    assert(renderView != null);
    //注册绘制流水线回调，可参考SchedulerBinding中第6条
    addPersistentFrameCallback(_handlePersistentFrameCallback);
    initMouseTracker();
    if (kIsWeb) {
      addPostFrameCallback(_handleWebFirstFrame);
    }
  }

2. 绘制核心方法 drawFrame

  @protected
  void drawFrame() {
    assert(renderView != null);
    //刷新需要布局
    pipelineOwner.flushLayout();
    pipelineOwner.flushCompositingBits();
    //刷新绘制
    pipelineOwner.flushPaint();
    if (sendFramesToEngine) {
        //生成Scene发送到Engine
      renderView.compositeFrame(); // this sends the bits to the GPU
      pipelineOwner.flushSemantics(); // this also sends the semantics to the OS.
      _firstFrameSent = true;
    }
  }


Widget绑定器 — WidgetsBinding
WidgetsBinding是Widget三棵树的入口，处理Widget, Element之间的一些业务,如：给Widget层注册生命周期的监听，亮度改变等等

1. runApp Widget入口方法

void runApp(Widget app) {
  WidgetsFlutterBinding.ensureInitialized()
    ..scheduleAttachRootWidget(app)
    ..scheduleWarmUpFrame();
}
2. initInstances 初始化化

  void initInstances() {
    super.initInstances();
    _instance = this;
        //管理Element标脏，回收
    _buildOwner = BuildOwner();
    buildOwner!.onBuildScheduled = _handleBuildScheduled;
    //语言，时区
    window.onLocaleChanged = handleLocaleChanged;
    window.onAccessibilityFeaturesChanged = handleAccessibilityFeaturesChanged;
    SystemChannels.navigation.setMethodCallHandler(_handleNavigationInvocation);
    FlutterErrorDetails.propertiesTransformers.add(transformDebugCreator);
  }
3. drawFrame 核心方法

  void drawFrame() {
        ...
    try {
      if (renderViewElement != null)
        // 对标脏的Element进行重建,从而生成新的三棵树
        buildOwner!.buildScope(renderViewElement!);
        
        //见RendererBinding#drawFrame
      super.drawFrame();
      //卸载不再使用的Element
      buildOwner!.finalizeTree();
    } finally {
    }  
  }

总结
通过以上分析可以得知Flutter中各Binding组件之间的协同工作构建出Flutter UI交互系统，它们之间又相对独立的只负责一类职责这样很好的解耦了业务之间的耦合度