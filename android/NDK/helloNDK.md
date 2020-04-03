



0. 准备

   SDK TOOL : ndk \ cmake勾选

   project配置：选择NDK路径 sdk\ndk\...



1.  写Java native class

   ```java
   package my.package;
   
   public clsss HelloNDK {
   	public static native void helloNDK();
   }
   ```

2. make project

3. 生成h文件：进入 build/intermediates/javac/debug/classes运行

   ```
   javah -jni my.package.HelloNDK
   ```

   在 build/intermediates/javac/debug/classes目录下生成my_package_HelloNDK.h, 将其复制到main/jni/目录下

4. h对应实现：在h目录下编写h的实现my_package_HelloNDK.cpp

   ```c++
   #include "my_package_HelloNDK.h"
   #include <jni.h>
   #include <stdio.h>
   #include <filesystem>
   #include <android/log.h>
   
   // LOGD宏定义
   #ifndef LOG_TAG
   #define LOG_TAG "HELLO_JNI"
   #define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG, LOG_TAG, __VA_ARGS_)
   
   extern "C" JNIEXPORT jint JNICALL Java_my_package_HelloNDK_helloNDK
     (JNIEnv *env, jclass obj) {
      LOGD("=======================HELLO DDK========================");
     }
   ```

5. cmake文件：

   ```cmake
   add_library(
   	hellojni //库名字
   	SHARED
   	my_package_HelloNDK.cpp // c/cpp文件
   )
   
   // 添加link的库，比如my_package_HelloNDK.cpp 中使用了android/log，就要link到androd log
   target_link_libraries(hellojni // link自己
     android
     log
   )
   ```

6. 添加gralde cmake配置

   ```groovy
   android {
       compileSdkVersion 29
       buildToolsVersion "29.0.2"
   
   
       defaultConfig {
           applicationId "com.paomian.app2"
           minSdkVersion 16
           targetSdkVersion 29
           versionCode 1
           versionName "1.0"
   
           testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
   
           externalNativeBuild {
               cmake {
                 // c构建参数
                   arguments "-DCMAKE_BUILD_TYPE=DEBUG"
               }
           }
       }
   
       externalNativeBuild {
           cmake {
             // 指定cmake文件
               path "src/main/jni/CMakeLists.txt"
           }
       }
   
       buildTypes {
           release {
               minifyEnabled false
               proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
           }
       }
   
   }
   ```

7. 构建后/build/intermediates/merged_native_libs可看到对应的so文件

8. 使用：

   ```java
   package my.package;
   
   public clsss HelloNDK {
     static {
       System.loadLibrary("hellojni")
     }
   	public static native void helloNDK();
   }
   
   ......
     
     HelloNDK.helloNDK();
   
   .......
     
   输出：
   =======================HELLO DDK========================
   ```

   





参考：

https://www.jianshu.com/p/87ce6f565d37