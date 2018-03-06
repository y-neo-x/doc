# Talk SDK for CocosCreator API接口手册

## 相关异步/同步处理方法介绍
游密语音引擎SDK提供的全部为Js接口,接口调用都会立即返回,凡是本身需要较长耗时的接口调用都会采用异步回调的方式,所有接口都可以在主线程中直接使用。回调在主线程。

## API调用说明
1.添加youmetalk.js的引用：
`var ymtalk = require("youmetalk");`

2.监听回调事件，事件名为youmetalk.js内的YMEvt枚举值，代码示例：
```
cc.eventManager.addListener({
	event: cc.EventListener.CUSTOM,    
	eventName: ymtalk.YMEvt.evt_common,
	callback: function (event) {
		var obj = event.getUserData();
		switch(obj.event){
			case 0:
			//todo:
			break;
			case 1:
			cc.log(obj.error);
			break;
		}
	}
}, 1);
```

3.初始化：
`ymtalk.init(appkey, appsecret, regoinid, exregionname);`

4.调用API相关功能函数：
`ymtalk.youmetalk.***`

* 接口使用的基本流程为：`实现回调事件监听`->`初始化`->`收到初始化成功回调通知`->`加入语音频道`->`收到加入频道成功回调通知`->`使用其它接口`->`离开语音频道`->`反初始化`，要确保严格按照上述的顺序使用接口。

* 文档内的接口除**初始化**外，均可用`ymtalk.youmetalk.***`进行调用，示例如下：
	*	初始化：`ymtalk.init(...);`
	*	加入房间：`ymtalk.youmetalk.talk_JoinChannelSingleMode(...);` 
	*	设置语音检测：`ymtalk.youmetalk.talk_SetVadCallbackEnabled(...);`

## 实现回调事件监听
youmetalk.js内已经实现了回调，并将回调通过事件广播了出来，所以使用时直接添加事件的listener即可监听回调事件，回调事件类型为youmetalk.js内的YMEvt枚举值。

**必须监听的回调事件**

* **ymtalk.YMEvt.evt_common**
	* *功能*
绝大部分异步函数调用后的回调，根据```event```参数来区分是什么回调

	* *参数说明*
```event```：当前是什么事件的回调，具体参见[YouMeEvent类型定义](#YouMeEvent类型定义)。
```error```：当前事件的错误码，具体参见[YouMeErrorCode类型定义](#youmeerrorcode类型定义)。
```channel```：频道ID
```param```：当前事件的参数，根据事件不同而不同，详情可见各个事件定义


**可选择监听的回调事件**

* **ymtalk.YMEvt.evt_memchange**
	* *功能*
设置了频道内成员通知的标识后，有成员变更就会收到此回调

	* *参数说明*
`channel`： 频道ID
`memberstr`：频道内成员列表（json字符串）
`isupdate`：是否是成员变更的标识

* **ymtalk.YMEvt.evt_restapi**
	* *功能*
RestApi请求的结果通知，调用了talk_RequestRestApi接口，结果返回后，就会收到此回调

	* *参数说明*
`cmd`： 请求的命令字符串，标识命令类型
`error`：错误码
`requestid`：回传ID
`result`：查询结果，json格式

* **ymtalk.YMEvt.evt_broadcast**
	* *功能*
语音频道内的广播通知，目前暂时只有在抢麦/连麦的活动时，就会收到此回调

	* *参数说明*
`channel`： 频道ID
`notifytype`： 广播的类型
`param1`：参数1，根据类型不同而不同
`param2`：参数2，根据类型不同而不同
`content`：调用抢连麦接口时用户传入的字符串

* **ymtalk.YMEvt.evt_tips**
	* *功能*
一些事件回调的提示信息，可用于显示

	* *参数说明*
`msg`： 提示信息
`error`：附带的错误码

## 初始化
* **语法**

```
init( strAppKey, strAPPSecret, serverRegionId, strExtServerRegionName );
```

* **功能**
初始化语音引擎，做APP验证和资源初始化。

* **参数说明**
`strAPPKey`：string，从游密申请到的 app key, 这个你们应用程序的唯一标识。
`strAPPSecret`：string， 对应 strAPPKey 的私钥, 这个需要妥善保存，不要暴露给其他人。
`serverRegionId`：int，设置首选连接服务器的区域码，参见状态码的YOUME_RTC_SERVER_REGION定义。如果在初始化时不能确定区域，可以填10001，后面确定时通过 talk_SetServerRegion 设置。如果YOUME_RTC_SERVER_REGION定义的区域码不能满足要求，可以把这个参数设为10000，然后通过后面的参数strExtServerRegionName 设置一个自定的区域值（如中国用 "cn" 或者 “ch"表示），然后把这个自定义的区域值同步给游密，我们将通过后台配置映射到最佳区域的服务器。
`strExtServerRegionName`：string,自定义的扩展的服务器区域名。可为空字符串“”。只有前一个参数serverRegionId设为10000时，此参数才有效（否则都将当空字符串“”处理）。

* **返回值**
int，返回0才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**

```
// 涉及到的主要回调事件有：
// OnEvent
// event: （0） - 表明初始化成功
// event: （1） - 表明初始化失败，最常见的失败原因是网络错误或者 AppKey-AppSecret 错误

```
## 语音频道管理

### 加入语音频道（单频道）

* **语法**

```
talk_JoinChannelSingleMode ( strUserID, strChannelID, roleType, bCheckRoomExist);
```
* **功能**
加入语音频道（单频道模式，每个时刻只能在一个语音频道里面）。

* **参数说明**
`strUserID`：string,全局唯一的用户标识，全局指在当前应用程序的范围内。
`strChannelID`：string, 全局唯一的频道标识，全局指在当前应用程序的范围内。
`roleType`：int,用户在语音频道里面的角色，见YouMeUserRole定义。
`bCheckRoomExist `：是否检查频道存在时才加入，默认为false: true 当频道存在时加入、不存在时返回错误（多用于观众角色），false 无论频道是否存在都加入频道

* **返回值**
int，返回0才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**

```
//涉及到的主要回调事件有：
//OnEvent
//event:2 - 成功进入语音频道
//event:3 - 进入语音频道失败，可能原因是网络或服务器有问题，或是bCheckRoomExist为true时频道还未创建
```

### 加入语音频道（多频道）

* **语法**

```
talk_JoinChannelMultiMode ( strUserID,strChannelID,bCheckRoomExist);
```
* **功能**
加入语音频道（多频道模式，可以同时听多个语音频道的内容，但每个时刻只能对着一个频道讲话）。

* **参数说明**
`strUserID`：string,全局唯一的用户标识，全局指在当前应用程序的范围内。
`strChannelID`：string,全局唯一的频道标识，全局指在当前应用程序的范围内。
`bCheckRoomExist `：是否检查频道存在时才加入，默认为false: true 当频道存在时加入、不存在时返回错误（多用于观众角色），false 无论频道是否存在都加入频道

* **返回值**
int，返回0才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**

```
// 涉及到的主要回调事件有：
// OnEvent
// event: 2 - 成功进入语音频道
// event: 3 - 进入语音频道失败，进入语音频道失败，可能原因是网络或服务器有问题，或是bCheckRoomExist为true时频道还未创建
```

### 指定讲话频道

* **语法**

```
talk_SpeakToChannel ( strChannelID);
```
* **功能**
多频道模式下，指定当前要讲话的频道。

* **参数说明**
`strChannelID`：string,全局唯一的频道标识，全局指在当前应用程序的范围内。

* **返回值**
int，返回0才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**

```
//涉及到的主要回调事件有：
// OnEvent
// event: 8 - 成功切入到指定语音频道
// event: 9 - 切入指定语音频道失败，可能原因是网络或服务器有问题

```

### 退出指定的语音频道
* **语法**

```
talk_LeaveChannelMultiMode (strChannelID);
```
* **功能**
多频道模式下，退出指定的语音频道。

* **参数说明**
`strChannelID`：string,全局唯一的频道标识，全局指在当前应用程序的范围内。

* **返回值**
int，返回0才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**

```
//涉及到的主要回调事件有：
// OnEvent
// event: 4 - 退出指定语音频道完成
```

### 退出所有语音频道

* **语法**

```
talk_LeaveChannelAll ();
```
* **功能**
退出所有的语音频道（单频道模式下直接调用此函数离开频道即可）。

* **返回值**
int，返回0才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**

```
//涉及到的主要回调事件有：
// OnEvent
// event: 5 - 退出所有语音频道完成
```

### 设置白名单用户

* **语法**

```
talk_SetWhiteUserList ( strChannelID, strWhiteUserList);
```
* **功能**
设置当前用户的语音消息接收白名单，其语音消息只会转发到白名单的用户，不设置该接口则默认转发至频道内所有人。

* **参数说明**
`strChannelID`：要设置的频道(兼容多频道模式，单频道模式下传入当前频道即可)。
`strWhiteUserList`：白名单用户列表, 以|分隔，如：User1|User2|User3；"all"表示转发至频道内所有人；""（空字符串）表示不转发至任何用户。

* **返回值**
int，返回0才会有异步回调通知。其它返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**

```
//涉及到的主要回调事件有：
// OnEvent
// event: 62 - 成功在指定语音频道设置白名单，有异常用户会返回错误码-501
// event: 63 - 在指定语音频道设置白名单失败，可能原因是网络或服务器有问题

```

## 设备状态管理

### 设置输出到扬声器/听筒

* **语法**

```
talk_SetOutputToSpeaker (bOutputToSpeaker);
```
* **功能**
是否输出到扬声器，同步接口。

* **参数说明**
`bOutputToSpeaker `：true——输出到扬声器，false——输出到听筒。

* **返回值**
int，如果成功返回0，否则返回错误码，请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

### 设置扬声器状态

* **语法**

```
talk_SetSpeakerMute (mute);
```
* **功能**
打开/关闭扬声器。**该状态值在加入房间成功后设置才有效。**

* **参数说明**
`mute`:true——关闭扬声器，false——开启扬声器。

* **相关回调**

```
// 涉及到的主要回调事件有：
// OnEvent
// event: 31 - 自己的扬声器打开
// event: 32 - 自己的扬声器关闭
```

### 获取扬声器状态

* **语法**

```
talk_GetSpeakerMute();
```

* **功能**
获取当前扬声器状态。

* **返回值**
true——扬声器关闭，false——扬声器开启。

### 设置麦克风状态

* **语法**

```
talk_SetMicrophoneMute (mute);
```

* **功能**
打开／关闭麦克风。**该状态值在加入房间成功后设置才有效。**

* **参数说明**
`mute`:true——关闭麦克风，false——开启麦克风。

* **相关回调**

```
// 涉及到的主要回调事件有：
// OnEvent
// event: 29 - 自己的麦克风打开
// event: 30 - 自己的麦克风关闭
```

### 获取麦克风状态

* **语法**

```
talk_GetMicrophoneMute ();
```

* **功能**
获取当前麦克风状态。

* **返回值**
true——麦克风关闭，false——麦克风开启。

### 设置是否通知别人麦克风和扬声器的开关

* **语法**

```
talk_SetAutoSendStatus( bAutoSend );
```

* **功能**
设置是否通知别人,自己麦克风和扬声器的开关状态

* **参数说明**
`bAutoSend`:true——通知，false——不通知。


* **相关回调**

```
// 涉及到的主要回调事件有(房间里的其他人会收到)：
// OnEvent
// event: 16 - 其他用户麦克风打开
// event: 17 - 其他用户麦克风关闭
// event: 18 - 其他用户扬声器打开
// event: 19 - 其他用户扬声器关闭
```

### 设置音量

* **语法**

```
talk_SetVolume (uiVolume);
```

* **功能**
设置当前程序输出音量大小。建议该状态值在加入房间成功后按需再重置一次。

* **参数说明**
`uiVolume`:unsigned int，当前音量大小，范围[0-100]。

### 获取音量

* **语法**

```
talk_GetVolume ();
```

* **功能**
获取当前程序输出音量大小。

* **返回值**
unsigned int ，当前音量大小，范围[0-100]。

## 设置网络

### 设置是否允许使用移动网络

* **语法**

```
talk_SetUseMobileNetworkEnabled (bEnabled);
```

* **功能**
设置是否允许使用移动网络。在WIFI和移动网络都可用的情况下会优先使用WIFI，在没有WIFI的情况下，如果设置允许使用移动网络，那么会使用移动网络进行语音通信，否则通信会失败。


* **参数说明**
`bEnabled`:true——允许使用移动网络，false——禁止使用移动网络。

### 获取是否允许使用移动网络

* **语法**

```
talk_GetUseMobileNetworkEnabled () ;
```

* **功能**
获取是否允许SDK在没有WIFI的情况使用移动网络进行语音通信。

* **返回值**
true——允许使用移动网络，false——禁止使用移动网络，默认情况下允许使用移动网络。

## 控制他人麦克风

* **语法**

```
talk_SetOtherMicMute ( strUserID, mute);
```

* **功能**
控制他人的麦克风状态

* **参数说明**
`strUserID`：string,要控制的用户ID
`mute`：是否静音。true:静音别人的麦克风，false：开启别人的麦克风

* **返回值**
int，如果成功返回0，否则返回错误码，请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **相关回调**
```
// 涉及到的主要回调事件有(被控制方收到)：
// OnEvent
// event: 23 - 麦克风被其他用户打开
// event: 24 - 麦克风被其他用户关闭
```

## 控制他人扬声器

* **语法**

```
talk_SetOtherSpeakerMute ( strUserID, mute);
```

* **功能**
控制他人的扬声器状态

* **参数说明**
`strUserID`：string,要控制的用户ID
`mute`：是否静音。true:静音别人的扬声器，false：开启别人的扬声器

* **返回值**
int，如果成功返回0，否则返回错误码，请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **相关回调**
```
// 涉及到的主要回调事件有(被控制方收到)：
// OnEvent
// event: 25 - 扬声器被其他用户打开
// event: 26 - 扬声器被其他用户关闭
```

## 设置是否听某人的语音

* **语法**

```
talk_SetListenOtherVoices (strUserID, on);
```

* **功能**
设置是否听某人的语音。

* **参数说明**
`strUserID`：string,要控制的用户ID。
`on`：true表示开启接收指定用户的语音，false表示屏蔽指定用户的语音。

* **返回值**
int，如果成功返回0，否则返回错误码，请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**
```
// 涉及到的主要回调事件有(被控制方收到)：
// OnEvent
// event: 27 - 取消屏蔽某人语音
// event: 28 - 屏蔽某人语音
```

## 通话管理

### 暂停通话

* **语法**

```
talk_PauseChannel();
```

* **功能**
暂停通话，释放对麦克风等设备资源的占用。当需要用第三方模块临时录音时，可调用这个接口。

* **返回值**
int，返回0才会有异步回调通知。其他返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**

```
//主要回调事件：
// OnEvent
// event: 6 - 暂停语音频道完成
```

### 恢复通话

* **语法**

```
talk_ResumeChannel();
```

* **功能**
恢复通话，调用PauseChannel暂停通话后，可调用这个接口恢复通话。

* **返回值**
int，返回0才会有异步回调通知。其他返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**

```
//主要回调事件：
// OnEvent
// event: 7 - 恢复语音频道完成
```

## 设置语音检测

* **语法**

```
talk_SetVadCallbackEnabled(bEnabled);
```

* **功能**
设置是否开启语音检测回调，开启后频道内有人正在讲话与结束讲话都会发起相应回调通知。**该状态值在加入房间成功后设置才有效。**

* **参数说明**
`bEnabled`:true——打开，false——关闭。

* **返回值**
int，如果成功则返回0，其他返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

## 背景音乐管理

### 播放背景音乐

* **语法**

```
talk_PlayBackgroundMusic ( strFilePath, bRepeat);
```

* **功能**
播放指定的音乐文件。播放的音乐将会通过扬声器输出，并和语音混合后发送给接收方。这个功能适合于主播/指挥等使用。

* **参数说明**
`strFilePath`：音乐文件的路径。
`bRepeat`：是否重复播放，true——重复播放，false——只播放一次就停止播放。

* **返回值**
int，返回0才会有异步回调通知。其他返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**

```
//主要回调事件：
// OnEvent
// event: 13 - 通知背景音乐播放结束
// event: 14 - 通知背景音乐播放失败
```

### 停止播放背景音乐
* **语法**

```
talk_StopBackgroundMusic();
```

* **功能**
停止播放当前正在播放的背景音乐。
这是一个同步调用接口，函数返回时，音乐播放也就停止了。

* **返回值**
int，如果成功返回0，表明成功停止了音乐播放流程；否则返回错误码，请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

### 设置背景音乐播放音量
* **语法**

```
talk_SetBackgroundMusicVolume(vol);
```

* **功能**
设定背景音乐的音量。这个接口用于调整背景音乐和语音之间的相对音量，使得背景音乐和语音混合听起来协调。
这是一个同步调用接口。

* **参数说明**
`vol`:int,背景音乐的音量，范围 [0-100]。

* **返回值**
int，如果成功（表明成功设置了背景音乐的音量）返回0，否则返回错误码，具体请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

## 设置监听
* **语法**

```
talk_SetHeadsetMonitorOn(enabled);
```

* **功能**
设置插耳机的情况下开启或关闭语音监听（即通过耳机听到自己说话的内容）。
这是一个同步调用接口。

* **参数说明**
`bEnabled`:true——打开，false——关闭监听。

* **返回值**
int，如果成功则返回0，其他返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

## 设置混响音效

* **语法**

```
talk_SetReverbEnabled( bEnabled);
```

* **功能**
设置是否开启混响音效，这个主要对主播/指挥有用。

* **参数说明**
`bEnabled`:true——打开，false——关闭。

* **返回值**
int，如果成功则返回0，其他返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

## 设置时间戳
### 设置录音时间戳

* **语法**

```
talk_SetRecordingTimeMs( timeMs);
```

* **功能**
设置当前录音的时间戳。当通过录游戏脚本进行直播时，要保证观众端音画同步，在主播端需要进行时间对齐。
这个接口设置的就是当前游戏画面录制已经进行到哪个时间点了。

* **参数说明**
`timeMs`:unsigned int, 当前游戏画面对应的时间点，单位为毫秒。

* **返回值**
无。

### 设置播放时间戳

* **语法**

```
talk_SetPlayingTimeMs( timeMs);
```

* **功能**
设置当前声音播放的时间戳。当通过录游戏脚本进行直播时，要保证观众端音画同步，游戏画面的播放需要和声音播放进行时间对齐。
这个接口设置的就是当前游戏画面播放已经进行到哪个时间点了。

* **参数说明**
`timeMs`:unsigned int,当前游戏画面播放对应的时间点，单位为毫秒。

* **返回值**
无。

## 设置服务器区域

* **语法**

```
talk_SetServerRegion(serverRegionId,strExtRegionName);
```

* **功能**
设置首选连接服务器的区域码.

* **参数说明**
`serverRegionId`：int,如果YOUME_RTC_SERVER_REGION定义的区域码不能满足要求，可以把这个参数设为 10000，然后通过后面的参数strExtServerRegionName 设置一个自定的区域值（如中国用 "cn" 或者 “ch"表示），然后把这个自定义的区域值同步给游密，我们将通过后台配置映射到最佳区域的服务器。
`strExtServerRegionName`：string,自定义的扩展的服务器区域名。可为空字符串“”。只有前一个参数serverRegionId设为10000时，此参数才有效（否则都将当空字符串“”处理）。


## RestApi——支持主播相关信息查询
* **语法**

```
talk_RequestRestApi( strCommand , strQueryBody  );
```
* **功能**
Rest API , 向服务器请求额外数据。支持主播信息，主播排班等功能查询。需要的话，请联系我们获取命令详情文档。

* **参数说明**
`strCommand`：请求的命令字符串，标识命令类型。
`strQueryBody`：请求需要的参数,json格式。

* **代码示例**

```
var body = "{\"ChannelID\":\"123456\",\"AreaID\":0}";
var errorcode = youmetalk.talk_RequestRestApi( "query_talk_channel_user_count", body);

```

* **返回值**
int，小于0，表示错误码，大于0，表示本次查询的requestID。

* **异步回调**

```
//requestID:int,回传ID
//iErrorCode:int,错误码
//query:string,回传查询请求，json格式，包含command（回传strCommand参数）和query（回传strQueryBody参数）字段。
//result:string,查询结果，json格式。
onRequestRestAPI( requestid, errcode, query, result )
```

* **回调示例**

```
query:{"command":"query_talk_channel_user_count","query":"{\"ChannelID\":\"123456\",\"AreaID\":0}"}
result:{"ActionStatus":"OK","ChannelID":"123456","ErrorCode":0,"ErrorInfo":"","UserCount":0}
```

## 安全验证码设置

* **语法**

```
talk_SetToken( strToken );
```

* **功能**
设置身份验证的token，需要配合后台接口。

* **参数说明**
`strToken`：string,身份验证用token，设置空字符串，清空token值，不进行身份验证。

## 查询频道用户列表

* **语法**

```
talk_GetChannelUserList( strChannelID,maxCount, notifyMemChange );
```

* **功能**
查询频道当前的用户列表， 并设置是否获取频道用户进出的通知。（必须自己在频道中）

* **参数说明**
`strChannelID`：string,频道ID。
`maxCount`：int,想要获取的最大人数。-1表示获取全部列表。
`notifyMemChange`：当有人进出频道时，是否获得通知。true，需要通知，false,不需要通知。

* **返回值**
int，返回0才会有异步回调通知。其他返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。

* **异步回调**

```
//strChannel:string,频道ID
//memchanges:对象数组，查询获得的用户列表，或变更列表。
//每一个数组的数据，包含userID和isJoin两个字段
onMemberChange(channel,  memberListJsonString, isUpdate)
```

## 广播消息
* **语法**

```
talk_SendMessage( channelID,  content );
```

* **功能**
在语音频道内，广播一个文本消息。


* **参数说明**
`channelID`：频道ID（自己需要进入这个频道）。
`content`：要广播的文本内容。

* **返回值**
int，小于0，表示错误码，大于0，表示本次查询的requestID。

* **异步回调**

```
// 涉及到的主要回调事件有：
// OnEvent
// event: （60） - 发送消息的结果，param为requestID的字符串
// event: （61） - 频道内其他人收到消息的通知。param为文本内容

```

## 把人踢出房间
* **语法**

```
talk_KickOtherFromChannel( userid, channelid, lastTime );
```

* **功能**
把人踢出房间。


* **参数说明**
`userid`：被踢的用户ID。
`channelid`：从哪个房间踢出（自己需要在房间）。
`lastTime`：踢出后，多长时间内不允许再次进入。


* **返回值**
int，返回0才会有异步回调通知。其他返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。


* **异步回调**

```
// 涉及到的主要回调事件有：
// OnEvent
// event: （64） - 踢人方收到，发送消息的结果回调，param为被踢者ID
// event: （65） - 被踢方收到，被踢通知，会自动退出所在频道。param: （踢人者ID，被踢原因，被禁时间）


```

## 设置日志等级
* **语法**

```
talk_SetLogLevel( level)
```

* **功能**
设置日志等级


* **参数说明**
`level`：日志等级。小于等于该等级的日志会打印。有效值参考YOUME_LOG_LEVEL类型定义。


## 反初始化

* **语法**

```
talk_UnInit ();
```

* **功能**
反初始化引擎，可在退出游戏时调用，以释放SDK所有资源。

* **返回值**
int，如果成功则返回0，其他返回值请参考[YouMeErrorCode类型定义](TalkCocosJsStatusCode.php#YouMeErrorCode定义)。


