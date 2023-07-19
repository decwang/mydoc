Bloc入门之Cubit详解

于 2022-06-22 16:15:05 发布
前面学写了一些基础的Flutter开发知识，但是对于项目的整体架构以及开发模式并没有进行一个系统的学习，经过上网查询，发现目前很多项目采用了Bloc(业务该逻辑组件)这种开发模式，下面就找到了该库的官网

优点
Bloc可以比较轻松地将展示层的代码与业务逻辑分开，从而使您的代码快速，易于测试且可重复使用

动手
首先对于Stream熟悉的同学可以忽略此部分，Java8的新特性里面也有流这部分内容。

从零开始
从0开始，写个简单的程序认识一下Stream这个类新建一个文件夹：
blocTest
1
新建一个test.dart文件，输入以下内容，简单了解以下流这个类型 ：
// 返回一个流，流里面是不大于max的数，要注意，每次yield的时候，都会向流里面推数据
Stream<int> countStream(int max) async* {
    for (int i = 0; i < max; i++) {
        yield i;
    }
}
// 使用await等待流中的每个数据，然后返回和,而且是`Future`类型
Future<int> sumStream(Stream<int> stream) async {
    int sum = 0;
    await for (int value in stream) {
        sum += value;
    }
    return sum;
}

void main() async {
  Stream<int> stream = countStream(10);
  int sum = await sumStream(stream);
  print(sum); // 45
预热以后，开始上手，安装库
在根文件夹里面新建一个文件pubspec.yaml并输入以下内容

name: blocTest
environment:
  sdk: ">=2.17.0-0 <3.0.0"
dependencies:
  bloc: ^8.0.0

执行dart pub get 下载库

PS C:\Users\XXX\Desktop\blocTest> dart pub get
Resolving dependencies...
Got dependencies!
3
开始上手
这是偷来官网的图：
https://img-blog.csdnimg.cn/ed6ba5cfe8ea42c99434dd47eed0cdc5.png

状态是从 Cubit 中输出的，代表应用程序状态的一部分。可以通知 UI 组件状态，并根据当前状态重绘其自身的某些部分。

创建一个Cubit 类型
// 定义了要管理的状态类型，此处以int示例，复杂业务可以自定义类
class CounterCubit extends Cubit<int> {
  //CounterCubit() : super(0);//指定初始状态为0
  //也可以外部传入初始值：
  CounterCubit(int initialState) : super(initialState);
}
定义一个CounterCubit
final cubitA = CounterCubit(0); // state starts at 0
final cubitB = CounterCubit(10); // state starts at 10
1
2
添加输出状态的能力
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
  void increment() => emit(state + 1);
  //emit只能在 Cubit 内部使用。state是当前状态
}
使用
import 'package:bloc/bloc.dart';
void main() {
  final cubit = CounterCubit();
  print(cubit.state); // 0
  cubit.increment();
  print(cubit.state); // 1
  cubit.close();
}
订阅
实际上Cubit 是 Stream 的一种特殊类型，可以通过订阅来更新其状态

void myPrint(int data)  {
  print(data);
}

Future<void> main() async {
  final cubit = CounterCubit();
  final subscription = cubit.stream.listen(myPrint); // 1
  cubit.increment(); //输出1
  await Future.delayed(Duration.zero);
  cubit.increment();//输出2

  await Future.delayed(Duration.zero);
  await subscription.cancel();
  await cubit.close();
}

观察
可以通过添加onChange函数来观察Cubit的变化

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);

  @override
  void onChange(Change<int> change) {
    super.onChange(change);
    print(change);
  }
}

修改main函数：

void main() {
  CounterCubit()
    ..increment()
    ..increment()
    ..close();
}

输出结果：

Change { currentState: 0, nextState: 1 }
Change { currentState: 1, nextState: 2 }
1
2
可见输出了当前状态和下一次状态

使用BlocObserver
添加一个一个Observer:
class SimpleBlocObserver extends BlocObserver {
  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    print('${bloc.runtimeType} $change');
    print(bloc.state);//访问Cuit实例状态
  }
}

修改main函数：

void main() {
  BlocOverrides.runZoned(
    () {
      CounterCubit()
        ..increment()
        ..close();
    },
    blocObserver: SimpleBlocObserver(),
  );
}

上面的代码在调用CounterCubit.increment()后会依次调用CounterCubit本身的onChange和SimpleBlocObserver的onChange，其实在BlocObserver 中也可以访问Cubit实例（bloc变量）

错误处理
再次修改代码，人为在在increment()函数里面调用addError制造错误，此时会调用Cubit的onError方法进行错误处理

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() {
    addError(Exception('increment error!'), StackTrace.current);
    emit(state + 1);
  }

  @override
  void onChange(Change<int> change) {
    super.onChange(change);
    print(change);
  }

  @override
  void onError(Object error, StackTrace stackTrace) {
    print('$error, $stackTrace');
    super.onError(error, stackTrace);
  }
}

此时运行的话就会报错了
————————————————
版权声明：本文为CSDN博主「武器大师72」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_18454025/article/details/125403831