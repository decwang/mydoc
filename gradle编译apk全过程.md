gradle编译apk全过程

gradle是一个免费、开源、通用、灵活的项目构建工具，可用来构建Android项目、打包生成apk。通过gradle生成apk一般需要执行./gradlew clean和./gradlew assembleRelease两个命令，下面通过这两个命令所执行的每个task来分析生成apk的过程。

./gradlew clean
./gradlew assembleRelease


####clean
./gradlew clean命令会执行以下任务：

1. task ':clean'
清理，删除项目根目录下的build文件夹，/build

2. task ':app:clean'
清理，删除app目录下的build文件夹，/app/build

####assembleRelease
./gradlew assembleRelease命令会执行以下任务：

1. task ':app:preBuild'
预构建，准备构建，在项目根目录下生成build文件夹，在app目录下生成build文件夹

2. task ':app:preReleaseBuild'
预构建，生成/app/build/intermediates/prebuild/release目录，Release是构建类型，下同

3. task ':app:compileReleaseAidl'
编译aidl相关文件，通过aidl工具把.aidl文件编译成.java文件 ，生成的结果在/app/build/generated/source/aidl/release目录

4. task ':app:compileReleaseRenderscript'
编译渲染脚本，

5. task ':app:checkReleaseManifest'
检查清单AndroidManifest.xml，生成/app/build/intermediates/check-manifest/release目录

6. task ':app:generateReleaseBuildConfig'
在/app/build/generated/source/buildConfig/release目录生成BuildConfig.java，构建配置类中有应用ID、构建类型、版本名称等

7. task ':app:prepareLintJar'
准备检查.jar文件

8. task ':app:mainApkListPersistenceRelease'
主要的apk列表持久化

9. task ':app:generateReleaseResValues'
在/app/build/generated/res/resValues/release目录，生成/app/src/main/res/values目录相应的资源值

10. task ':app:generateReleaseResources'
生成资源，即生成color，string，style等resources

11. task ':app:mergeReleaseResources'
合并资源，即合并color，string，style等resources到values.xml

12. task ':app:createReleaseCompatibleScreenManifests'
创建兼容屏幕清单

13. task ':app:processReleaseManifest'
处理清单，生成处理后的/app/build/intermediates/manifests/full/release/AndroidManifest.xml

14. task ':app:splitsDiscoveryTaskRelease'
分裂发现任务

15. task ':app:processReleaseResources'
处理资源

16. task ':app:generateReleaseSources'
生成源码，在/app/build/generated/source/r/release目录生成R.java

17. task ':app:javaPreCompileRelease'
预编译.java文件

18. task ':app:compileReleaseJavaWithJavac'
使用javac编译.java文件，在/app/build/intermediates/classes/release目录生成R.class，BuildConfig.class，MainActiviy.class等.class文件

19. task ':app:compileReleaseNdk'
通过ndk编译.c或.cpp等相关文件

20. task ':app:compileReleaseSources'
编译源码

21. task ':app:lintVitalRelease'
进行至关重要的检查，检查有没有严重错误

22. task ':app:mergeReleaseShaders'
合并着色器，着色器是用来实现图像渲染的

23. task ':app:compileReleaseShaders'
编译着色器

24. task ':app:generateReleaseAssets'
生成资产assets

25. task ':app:mergeReleaseAssets'
合并资产assets

26. task ':app:transformClassesWithDexBuilderForRelease'
使用dex构建器转换.class文件，在/app/build/intermediates/transforms/dexBuilder/release目录生成R.dex，BuildConfig.dex，MainActivity.dex等.dex文件

27. task ':app:transformDexArchiveWithExternalLibsDexMergerForRelease'
使用外部库dex合并器转换.dex归档文件，在/app/build/intermediates/transforms/externalLibsDexMerger/release目录生成classes.dex文件

28. task ':app:transformDexArchiveWithDexMergerForRelease'
使用dex合并器转换.dex归档文件，
在/app/build/intermediates/transforms/dexMerger/release目录生成classes.dex文件

29. task ':app:mergeReleaseJniLibFolders'
合并jniLibs文件夹

30. task ':app:transformNativeLibsWithMergeJniLibsForRelease'
转换本地库文件，合并jni库文件

31. task ':app:transformNativeLibsWithStripDebugSymbolForRelease'
转换本地库文件，剥离调试符号

32. task ':app:processReleaseJavaRes'
处理Java资源

33. task ':app:transformResourcesWithMergeJavaResForRelease'
转换资源，合并Java资源，生成resources.arsc

34. task ':app:validateSigningRelease'
验证签名

35. task ':app:packageRelease'
打包，在/app/build/intermediates/incremental/packageRelease目录生成相关文件

36. task ':app:assembleRelease'
聚集，组合，最终生成/app/build/outputs/apk/release/app-release.apk




----------------
891527-20190314154758460-254525529.png
如图，

编译器将源代码（包括 Application Module 及其所依赖的所有 Library 源代码）转换成 DEX（Dalvik Executable）文件（其中包括运行在 Android 设备上的字节码），将所有其他内容转换成已编译资源。（
将源代码（包括 Application Module 和 Library Module）编译成 class 文件，再将所有的 class 文件（包括第三方库中）打包生成 dex 文件。
解压所有 aar 包中的资源文件，并和项目中所有资源文件合并到一个目录。
生成资源文件的索引文件。
）
APK 打包器将 DEX 文件和已编译资源打包生成单个 APK。不过，必须先对 APK 签名，才能将应用安装并部署到 Android 设备上。
APK 打包器使用 Debug 或 Release 密钥库对 APK 签名：
如果您构建的是 Debug 版本的应用（即专用于测试和分析的应用），打包器会使用 Debug 密钥库签名应用。Android Studio 自动使用 Debug 密钥库配置新项目。
如果您构建的是打算向外发布的 Release 版本应用，打包器会使用 Release 密钥库签名应用。
在生成最终 APK 之前，打包器会使用 zipalign 工具对应用进行优化，减少其在设备上运行时的内存占用。
 

2. Gradle 构建 tasks 说明
我们通过 Android Studio 的 build -> Build APK 功能来看下，build 过程中是怎样的。Build APK 会执行以下 Gradle task。 （注：这里是用 Debug 模式做例子，Release 模式时只需将 task 中的 Debug 替换成 Release 理解即可）

复制代码
Executing tasks: [:app:assembleDebug]

:app:preBuild UP-TO-DATE
:app:preDebugBuild UP-TO-DATE
:app:compileDebugAidl UP-TO-DATE
:app:compileDebugRenderscript UP-TO-DATE
:app:generateDebugResValues UP-TO-DATE
:app:generateDebugResources UP-TO-DATE
:app:mergeDebugResources UP-TO-DATE
:app:transformDataBindingBaseClassLogWithDataBindingMergeGenClassesForDebug UP-TO-DATE
:app:dataBindingGenBaseClassesDebug UP-TO-DATE
:app:checkDebugManifest UP-TO-DATE
:app:generateDebugBuildConfig UP-TO-DATE
:app:prepareLintJar UP-TO-DATE
:app:mainApkListPersistenceDebug UP-TO-DATE
:app:createDebugCompatibleScreenManifests UP-TO-DATE
:app:processDebugManifest
:app:splitsDiscoveryTaskDebug UP-TO-DATE
:app:processDebugResources
:app:generateDebugSources
:app:dataBindingExportBuildInfoDebug
:app:javaPreCompileDebug UP-TO-DATE
:app:transformDataBindingWithDataBindingMergeArtifactsForDebug UP-TO-DATE
:app:compileDebugJavaWithJavac
:app:compileDebugNdk NO-SOURCE
:app:compileDebugSources
:app:mergeDebugShaders UP-TO-DATE
:app:compileDebugShaders UP-TO-DATE
:app:generateDebugAssets UP-TO-DATE
:app:mergeDebugAssets UP-TO-DATE
:app:transformClassesWithDexBuilderForDebug
:app:transformDexArchiveWithExternalLibsDexMergerForDebug
:app:transformDexArchiveWithDexMergerForDebug
:app:mergeDebugJniLibFolders UP-TO-DATE
:app:transformNativeLibsWithMergeJniLibsForDebug UP-TO-DATE
:app:processDebugJavaRes NO-SOURCE
:app:transformResourcesWithMergeJavaResForDebug UP-TO-DATE
:app:validateSigningDebug UP-TO-DATE
:app:packageDebug
:app:assembleDebug


BUILD SUCCESSFUL in 20s
复制代码
1
特别说明：由于 Gradle 会尝试通过不重复执行输入未发生变化的 task 来节省时间（这些 task 被标记为 UP-TO-DATE，如上所示）<br>所以第一次编译运行会比较久，而后面就不会了。
　　

上面列出来的 Gradle 构建过程，大致可分为五个阶段：

Preparation of dependecies 在这个阶段 Gradle 检测 Module 依赖的所有 library 是否 ready。如果这个 Module 依赖于另一个 Module ，则另一个 Module 也要被编译。
Merging resources and processing Manifest 打包资源和 Manifest 文件。
Compiling 在这个阶段处理编译器的注解，源码被编译成字节码。
Postprocessing 所有带 transform 前缀的 task 都是这个阶段进行处理的。
Packaging and publishing 这个阶段 library 生成.aar文件，Application 生成.apk文件。
这五个阶段和上面的构建过程中 task 的时序有大致的对应关系，大家可以相互映照着理解。

为了便于大家理解，下面给出一些关键 task 的说明：

mergeDebugResources 解压所有的 aar 包输出到app/build/intermediates/exploded-aar，并且把所有的资源文件合并到app/build/intermediates/res/merged/debug目录里。

processDebugManifest 把所有 aar 包里的AndroidManifest.xml中的节点，合并到项目的AndroidManifest.xml中，并根据app/build.gradle中当前buildType 的 manifestPlaceholders配置内容替换 manifest 文件中的占位符，最后输出到app/build/intermediates/manifests/full/debug/AndroidManifest.xml。

processDebugResources

调用 aapt 生成项目和所有 aar 依赖的R.java，输出到app/build/generated/source/r/debug目录；
生成资源索引文件app/build/intermediates/res/debug/resources-debug.ap_；
把符号表输出到app/build/intermediates/symbols/debug/R.txt。
compileDebugJavaWithJavac 用来把 java 文件编译成 class 文件，输出的路径是app/build/intermediates/classes/debug

编译的输入目录有

项目源码目录，默认路径是app/src/main/java，可以通过 sourceSets 的 dsl 配置，允许有多个（打印project.android.sourceSets.main.java.srcDirs可以查看当前所有的源码路径,具体配置可以参考 android-doc，以及这里。
app/build/generated/source/aidl。
app/build/generated/source/buildConfig。
app/build/generated/source/apt(继承javax.annotation.processing.AbstractProcessor做动态代码生成的一些库，输出在这个目录)的代码。
transformClassesWithMultidexlistForDebug （这个任务在使用 Multidex 才会出现）它有两个作用

扫描项目的AndroidManifest.xml文件和分析类之间的依赖关系，计算出那些类必须放在第一个 dex 里面，最后把分析的结果写到app/build/intermediates/multi-dex/debug/maindexlist.txt文件里面
生成混淆配置项输出到app/build/intermediates/multi-dex/debug/manifest_keep.txt文件里
项目里的代码入口是 manifest 中 application 节点的属性 android.name 配置的继承自 Application 的类，在 Android 5.0 以前的版本系统只会加载一个 dex(classes.dex)，classes2.dex .......classesN.dex 一般是使用android.support.multidex.MultiDex加载的，所以如果入口的 Application 类不在 classes.dex 里 5.0 以下肯定会挂掉，另外当入口 Application 依赖的类不在 classes.dex 时初始化的时候也会因为类找不到而挂掉，还有如果混淆的时候类名变掉了也会因为对应不了而挂掉，综上所述就是这个任务的作用。