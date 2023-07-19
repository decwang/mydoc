gradle版本 kotlin版本

diff --git a/android/app/build.gradle b/android/app/build.gradle
index 0caf7bb..557a1ec 100644
--- a/android/app/build.gradle
+++ b/android/app/build.gradle

 android {
-    compileSdkVersion 30
+    compileSdkVersion 31

     sourceSets {
         main.java.srcDirs += 'src/main/kotlin'
@@ -35,7 +35,7 @@ android {
     defaultConfig {
-        minSdkVersion 16
+        minSdkVersion 24

diff --git a/android/build.gradle b/android/build.gradle
index 9b6ed06..5e8d6cb 100644
--- a/android/build.gradle
+++ b/android/build.gradle
@@ -1,12 +1,12 @@
 buildscript {
-    ext.kotlin_version = '1.3.50'
+    ext.kotlin_version = '1.7.10'

     dependencies {
-        classpath 'com.android.tools.build:gradle:4.1.0'
+        classpath 'com.android.tools.build:gradle:7.2.0'

diff --git a/android/gradle/wrapper/gradle-wrapper.properties b/android/gradle/wrapper/gradle-wrapper.properties
-distributionUrl=https\://services.gradle.org/distributions/gradle-6.7-all.zip
+distributionUrl=https\://services.gradle.org/distributions/gradle-7.5-all.zip


从网上下载的代码往往运行时存在问题,各种版本类的
有一个较为简单的解法,用flutter create 创建一个demo工程,运行下,一般都可以.然后对比两个项目的gradle配置,不一样的参照demo改下.