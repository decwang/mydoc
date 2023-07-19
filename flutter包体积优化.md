flutter包体积优化方案

作者：ych
方案一 熊猫压缩法(减少 0.7 MB)
方案二 so优化(减少14MB)
方案三 混淆优化(减少0.4MB)
评论区
方案一 熊猫压缩法(减少 0.7 MB)
压缩对象: 1.Flutter引用到的资源文件 2.Android启动页的背景图

方案二 so优化(减少14MB)
flutter build apk --target-platform android-arm,android-arm64,android-x64 --split-per-abi
首先flutter build apk表示当前构建release包；
后面android-arm,android-arm64,android-x64则是指定生成对应架构的release包；
最后的--split-per-abi则表示告知需要按照我们指定的类型分别打包，如果移除则直接构建包含所有CPU架构的Apk包。

方案三 混淆优化(减少0.4MB)
flutter build apk --obfuscate --split-debug-info=//
--obfuscate：开启混淆操作；
--split-debug-info=：将因混淆生成的map符号表缓存到此位置。

symbolize Flutter混淆调试神器

