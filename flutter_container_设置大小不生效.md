1. 为什么Container设置width和height不生效?
下面这段代码，Container设置了(width: 100, height: 100),但渲染出来是填充满屏幕的，这是为什么?

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(width: 100, height: 100, color: Colors.red);
  }
}

可以从Container的初始化和build方法看下具体原因:

Container({
  double? width,
  double? height,
  ...
}) : 
     constraints =
      (width != null || height != null)
        ? constraints?.tighten(width: width, height: height)
          ?? BoxConstraints.tightFor(width: width, height: height)
        : constraints,
     super(key: key);
     
@override
Widget build(BuildContext context) {
  Widget? current = child;

  if (color != null)
    current = ColoredBox(color: color!, child: current);

  if (constraints != null)
    current = ConstrainedBox(constraints: constraints!, child: current);

  return current!;
}
从Container初始化可以看出constraints是一个紧约束,宽高是100。
https://upload-images.jianshu.io/upload_images/1387554-d0f1ef1f487f380e.png?imageMogr2/auto-orient/strip|imageView2/2/w/764/format/webp

image.png
通过build方法生成ColoredBox和ConstrainedBox,其对应的渲染类为_RenderColoredBox和RenderConstrainedBox。其对应关系如下:

image.png
flutter约束有下面特点:
https://upload-images.jianshu.io/upload_images/1387554-634681e4e8f67304.png?imageMogr2/auto-orient/strip|imageView2/2/w/764/format/webp

image.png
可以理解RenderConstrainedBox的约束是从父节点向下传递，而大小是从_RenderColoredBox向上传递。

constraints是怎么来的？

constraints是通过RenderView.performLayout传递逐级传递下来的,此时的constraints是一个紧约束,宽高就是屏幕的高度。


image.png
_additionalConstraints是RenderConstrainedBox初始化传递进来的约束，也就是开始设置的（100,100)

_additionalConstraints.enforce(constraints)是决定了当前size的大小:

//BoxConstraints
BoxConstraints enforce(BoxConstraints constraints) {
  return BoxConstraints(
    minWidth: minWidth.clamp(constraints.minWidth, constraints.maxWidth),
    maxWidth: maxWidth.clamp(constraints.minWidth, constraints.maxWidth),
    minHeight: minHeight.clamp(constraints.minHeight, constraints.maxHeight),
    maxHeight: maxHeight.clamp(constraints.minHeight, constraints.maxHeight),
  );
}
_additionalConstraints是Container的约束(100,100),constraints是屏幕传递下来的宽度和高度(414,896)。_additionalConstraints.enforce(constraints)最后产生的结果就是屏幕的高度和宽度。而这个约束继续传递给_RenderColoredBox进行绘制颜色,这也解释了为什么Container设置了宽高不生效的原因。

2. 修改上面代码,再想想结果是什么？
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(width: 100, height: 100, color: Colors.red),
    );
  }
}
直接看下Center继承自Align,其对应的RenderObject是RenderPositionedBox,而performLayout方法做了什么？

@override
void performLayout() {
  ...
  if (child != null) {
    child!.layout(constraints.loosen(), parentUsesSize: true);
    ...
  } else {
    ...
  }
child.layout传递的是constraints.loosen(),

BoxConstraints(
  minWidth: 0.0,
  maxWidth: maxWidth,
  minHeight: 0.0,
  maxHeight: maxHeight,
)
maxWidth和maxHeight是double.infinity,也就是(100,100).enforce(constraints)返回是(100,100)。这就很好的解释了为什么Center包裹之后child就生效了。

如果了解约束的原理,下面的两个题应该很容易得出结果:

ConstrainedBox(
  constraints: const BoxConstraints(
    minWidth: 70,
    minHeight: 70,
    maxWidth: 150,
    maxHeight: 150,
  ),
  child: Container(color: Colors.red, width: 10, height: 10),
)

Center(
  child: ConstrainedBox(
    constraints: const BoxConstraints(
      minWidth: 70,
      minHeight: 70,
      maxWidth: 150,
      maxHeight: 150,
    ),
    child: Container(color: red, width: 1000, height: 1000),
  ),
)
总结:
约束是从父类向下传递的，在计算自身layout时会用自身的约束与父类约束做一个收缩，只有当子约束的最大最小高度和宽度都是在父控件的范围内才生效。否则以父约束的临界点生效。

作者：某非著名程序员
链接：https://www.jianshu.com/p/5600e3813c44
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。