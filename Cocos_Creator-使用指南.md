# Talk SDK for CocosCreator 使用指南

## 适用范围

本规范仅适用于使用游密实时语音引擎Cocos2d-js接口开发多人实时语音功能的开发者。

## SDK目录概述

语音SDK中有两个子文件夹：lib、include,下面依次介绍下这两个子文件夹。

1. `include`：SDK的头文件。
详细介绍下inlude，所有接口都在这个文件夹中。
    * `IYouMeVoiceEngine.h`封装了语音SDK的全部功能接口，集成方可通过IYouMeVoiceEngine::getInstance ()->…来调用语音SDK接口。
    * `IYouMeEventCallback.h`包含语音SDK的所有回调事件接口，例如初始化结果，频道加入成功等，都将通过此接口通知集成方
    * `YouMeConstDefine.h`包含语音SDK的所有枚举类型定义，如错误码等。
2. `lib`：库文件，分为Android平台和iOS平台。Android平台下包括ARMv5、ARMv7和X86三种CPU架构下的libyoume_voice_engine.so文件，还包括youme_voice_engine.jar。iOS平台下包含libyoume_voice_engine.a文件。
3. jsb_youmetalk.h, jsb_youmetalk.cpp：JSB的封装文件
4. youmetalk.js：js的封装文件，供开发者调用

## 开发环境集成
### CocosCreator项目集成

将youmetalk.js放置到`项目名\assets\Script`目录中

### 构建后的SDK集成
* 构建：Cocos Creator菜单选择：`项目->构建发布`，在弹出的面板上选择`发布平台->***`，`模板->link`，然后点击构建。
* 目录拷贝：构建成功后，打开生成的目录：`项目名\build\jsb-link\frameworks\runtime-src`，目录结构与下图类似，将引擎SDK目录更名为youme_voice_engine（内含“include”和“lib”两个子文件夹），并复制到该目录下，这个目录下包含了Android、iOS、windows三个平台所需的所有C++头文件和库文件。
* ![](https://www.youme.im/doc/images/talk_cocos_project_directory.png)
* 文件拷贝：把jsb_youmetalk.h，jsb_youmetalk.cpp放入`...\frameworks\runtime-src\Classes`目录
* 接口注册：修改Classes目录内的AppDelegate.cpp文件：
	* 添加引用：`#include "jsb/jsb_youmetalk.hpp"`
	* 添加代码：在`AppDelegate::applicationDidFinishLaunching()`方法内的`se->start();`之前，加上：`se->addRegisterCallback(register_all_youmetalk);`

### Android系统Android-Studio开发环境配置

1. 修改proj.android-studio/app/jni/Android.mk文件，对应位置增加指定内容，分别对游密实时语音SDK的动态库进行预编译处理、添加头文件路径、链接动态库。

  ``` Shell
    LOCAL_PATH := $(call my-dir)
    include $(CLEAR_VARS)

    #＝＝＝＝＝＝Youme 添加＝＝＝＝＝＝＝＝＝
    LOCAL_MODULE := youme_voice_engine
    LOCAL_SRC_FILES := ../$(LOCAL_PATH)/../../../youme_voice_engine/lib/android/$(TARGET_ARCH_ABI)/libyoume_voice_engine.so
    include $(PREBUILT_SHARED_LIBRARY)
    #＝＝＝＝＝＝结束 Youme 添加＝＝＝＝＝＝＝

    LOCAL_MODULE := cocos2dcpp_shared
    LOCAL_MODULE_FILENAME := libcocos2djs

	ifeq ($(USE_ARM_MODE),1)
	LOCAL_ARM_MODE := arm
	endif

    LOCAL_SRC_FILES := hellojavascript/main.cpp \
				   ../../../Classes/AppDelegate.cpp \
				   ../../../Classes/jsb_module_register.cpp \
    #＝＝＝＝＝＝==Youme修改,不要拷贝这一行＝＝＝＝＝＝＝＝＝
                   ../../../Classes/jsb_youmetalk.cpp
    #＝＝＝＝＝＝结束 Youme 修改＝＝＝＝＝＝＝

    #＝＝＝＝＝＝==Youme修改＝＝＝＝＝＝＝＝＝
    LOCAL_C_INCLUDES := $(LOCAL_PATH)/../../../Classes \
                        $(LOCAL_PATH)/../../../youme_voice_engine/include
    #＝＝＝＝＝＝结束 Youme 修改＝＝＝＝＝＝＝

	ifeq ($(USE_ANY_SDK),1)
	LOCAL_SRC_FILES += ../../../Classes/anysdk/SDKManager.cpp \
				   ../../../Classes/anysdk/jsb_anysdk_basic_conversions.cpp \
				   ../../../Classes/anysdk/manualanysdkbindings.cpp \
				   ../../../Classes/anysdk/jsb_anysdk_protocols_auto.cpp

	LOCAL_C_INCLUDES += $(LOCAL_PATH)/../../../Classes/anysdk

	LOCAL_WHOLE_STATIC_LIBRARIES := PluginProtocolStatic
	endif

    LOCAL_STATIC_LIBRARIES := cocos2d_js_static

	LOCAL_EXPORT_CFLAGS := -DCOCOS2D_DEBUG=2 -DCOCOS2D_JAVASCRIPT

    #＝＝＝＝＝＝Youme 添加＝＝＝＝＝＝＝＝＝
    LOCAL_SHARED_LIBRARIES := youme_voice_engine
    #＝＝＝＝＝＝结束 Youme 添加＝＝＝＝＝＝＝

    include $(BUILD_SHARED_LIBRARY)
	$(call import-module, scripting/js-bindings/proj.android)

  ```

2. 如果需要显示指定CPU架构则修改proj.android/jni/Application.mk文件，增加指定部分的内容(v5版本为APP_ABI := armeabi)；如果不需要指定CPU架构Application.mk文件, 则不用修改。

  ``` Shell
  #＝＝＝＝＝＝修改＝＝＝＝＝＝＝＝＝＝＝
  APP_ABI := armeabi-v7a
  #＝＝＝＝＝＝结束修改＝＝＝＝＝＝＝＝＝

  APP_CPPFLAGS := -frtti -DCC_ENABLE_CHIPMUNK_INTEGRATION=1 -std=c++11 -fsigned-char
  APP_LDFLAGS := -latomic

  ifeq ($(USE_ANY_SDK),1)
    APP_CPPFLAGS += -DPACKAGE_AS
  endif

  ifeq ($(NDK_DEBUG),1)
  	APP_CPPFLAGS += -DCOCOS2D_DEBUG=1
  	APP_CFLAGS += -DCOCOS2D_DEBUG=1
  	APP_OPTIM := debug
  else
  	APP_CPPFLAGS += -DNDEBUG
  	APP_CFLAGS += -DNDEBUG
  	APP_OPTIM := release
  endif

  ```

3. 复制youme_voice_engine/lib/android/youme_voice_engine.jar到proj.android-studio/app/libs/youme_voice_engine.jar。

4. 修改proj.android/AndroidManifest.xml文件，确保声明了如下的权限：

    ```
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.BLUETOOTH" />
    ```

5. 修改proj.android/AndroidManifest.xml文件，声明服务：

    ```
    <service android:name ="com.youme.voiceengine.VoiceEngineService">
          <intent-filter>
              <action android:name="com.youme.voiceengine.VoiceEngineService"/>
              <category android:name="android.intent.category.default"/>
          </intent-filter>
     </service>
    ```

6. 用android-studio打开项目，在项目的第一个启动的AppActivity（找到AppActivity.java文件）中导入package:

    ```
    import  com.youme.voiceengine.mgr.YouMeManager;
    import  com.youme.voiceengine.*;
    ```
    然后在onCreate方法里添加如下代码(没有此方法的话需要自己补上)：

    ```
    @Override
    public void onCreate(Bundle savedInstanceState)
    {
        //以下两个函数调用顺序不能错
        YouMeManager.Init(this);
        super.onCreate(savedInstanceState);
        //启动服务用来监控网络的变化
        Intent intent = new Intent(this,VoiceEngineService.class);
        startService(intent);

    }
    ```

### iOS系统XCode开发环境配置

添加头文件和依赖库:
1. 添加头文件路径：在`Build Settings` -> `Search Paths` -> `Header Search Paths`中添加:
    `../../../youme_voice_engine/include`  （建议直接将此include文件夹拖到xcode需要填入的位置，然后路径会自动生成)，
    `../../cocos2d-x/cocos/scripting/js-bindings/manual`;
2. 添加库文件路径：在`Build Settings` -> `Search Paths` -> `Library Search Paths`中添加`../../../youme_voice_engine/lib/ios` （建议直接将此ios文件夹拖到xcode需要填入的位置，然后路径会自动生成);
3. 把jsb_youmetalk.h,js_youmetalk.cpp加入项目。
4. 添加依赖库：在`Build Phases`  -> `Link Binary With Libraries`下添加：`libc++.tbd`、`libsqlite3.0.tbd`、`libyoume_voice_engine.a`、`libz.dylib`、`libz.1.2.5.tbd`、`libresolv.9.tbd`、`SystemConfiguration.framework`、`CoreTelephony.framework`、`AVFoundation.framework`、`AudioToolBox.framework`、`CFNetwork.framework`。

## 接口注册
* 引入C++接口文件 #include "jsb_youmetalk.hpp"
* 注册js方法：在AppDelegate::applicationDidFinishLaunching() 中，添加注册函数。

    ```
    ///添加这句
    se->addRegisterCallback(register_jsb_youmetalk);
    ///添加结束

    se->start();

    ```


## API接口调用流程
API调用的基本流程如下图所示，具体接口说明参见API接口手册。
![](https://www.youme.im/doc/images/talk_shixutu.png)




