# RTC WEB SDK #



#### 添加依赖 ####

```
<script src="libs/md5.min.js"></script>
<script src="libs/int64.min.js"></script>
<script src="libs/msgpack.min.js"></script>
<script src="libs/fpnn.min.js"></script>
<script src="libs/rtm.min.js"></script>
<script src="libs/rtc.min.js"></script>
```



#### 初始化RTCClient

```
let rtcClient = new RTCClient({ 
	rtmEndpoint: 'rtm-nx-front.ilivedata.com:13322',  // 接入点请从云上曲率网站控制台获取
    rtcEndpoint: 'rtc-web.ilivedata.com:13706',
    connectionTimeout: 10 * 1000,
    pid: 80000071,  // 项目ID请从云上曲率控制台获取
});
```



#### 设置事件监听

```
// RTM连接关闭事件
rtcClient.on('SessionClosed', function(errorCode) {
    console.log('SessionClosed: ' + errorCode);
});

// 内部错误信息收集
rtcClient.on('ErrorRecorder', function(err) {
    console.log(err);
});

/* 在进行自动重连时，每次重连完成都会触发ReloginCompleted事件。
     successful: 重连是否成功
     retryAgain: 是否会进行下一次重连尝试
     errorCode: 本次重连收到的错误码
     retriedCount: 已经尝试重连了多少次
    */
rtcClient.on('ReloginCompleted', function(successful ,retryAgain, errorCode, retriedCount) 

});


/* RTC房间成员变动时触发事件

		memberChangedMap结构: {"enter": [RTCRoomMember], "leave": [RTCRoomMember], "change": [RTCStatusChangedInfo], "all": {uid => RTCRoomMember} }
				enter: 新加入的成员
				leave: 新离开的成员
				all: 当前全量成员
				
*/
rtcClient.on('RoomMembersChanged', function(memberChangedMap) {
    console.log(memberChangedMap);
});
```



#### 事件相关结构

```
RTCRoomMember结构:
      RTCRoomMember.roomID: 当前房间ID
      RTCRoomMember.userID: 用户ID
      RTCRoomMember.getStream(): 获取对应MediaStream
      RTCRoomMember.enableCamera(isEnable): 打开/关闭该成员视频
      RTCRoomMember.enableMicrophone(isEnable): 打开/关闭该成员麦克风
                
RTCStatusChangedInfo结构:
      RTCRoomMember.userID: 用户ID
      RTCRoomMember.changedEvent: 变化类型： 
          RTC_EV_CAMERA_CLOSE: 3,
          RTC_EV_CAMERA_OPEN: 4,
          RTC_EV_MICROPHONE_CLOSE: 5,
          RTC_EV_MICROPHONE_OPEN: 6,
```



####  登录登出RTC

```
// 登录RTC，token为RTC的登录验证凭证，需由业务服务端请求RTM服务器获取
rtcClient.login(new rtm.RTMConfig.Int64(uid), token, function(ok, errorCode) {
    if (errorCode == fpnn.FPConfig.ERROR_CODE.FPNN_EC_OK) {
        if (!ok) {
            // token error, need to get a new token
            return;
        } else {
            // login successfully
        }
    } else {
        // login error
    }
}, 60 * 1000);

// 退出RTC
rtcClient.logout();

// 判断是否已登录
rtcClient.isLogin();
```



#### 加入退出RTC房间

```
/* 加入视频房间
    enableCamera: 是否开启摄像头
    enableMicrophone: 是否开启麦克风
    返回:
		publishStream: 推流的MediaStream
		memberMap:  {uid => RTCRoomMember}
*/
rtcClient.enterVideoRoom(new rtm.RTMConfig.Int64(rid), enableCamera, enableMicrophone, timeoutInMilliseconds, function(errorCode, publishStream, memberMap) {
    if (errorCode == fpnn.FPConfig.ERROR_CODE.FPNN_EC_OK) {
				// enter room successfully
    } else {
        // enter room fail
    }
});

/* 加入音频房间
    enableMicrophone: 是否开启麦克风
    返回:
		publishStream: 推流的MediaStream
		memberMap:  {uid => RTCRoomMember}
*/
rtcClient.enterAudioRoom(new rtm.RTMConfig.Int64(rid), enableMicrophone, timeoutInMilliseconds, function(errorCode, publishStream, memberMap) {
    if (errorCode == fpnn.FPConfig.ERROR_CODE.FPNN_EC_OK) {
				// enter room successfully
    } else {
        // enter room fail
    }
});

// 离开当前RTC房间
rtcClient.leaveRoom();
```



#### RTC房间成员管理

```
/*
    返回： {uid => RTCRoomMember}
*/
rtcClient.getRTCRoomMembers();

// 打开/关闭某成员视频
RTCRoomMember.enableCamera(isEnable);

// 打开/关闭某成员声音
RTCRoomMember.enableMicrophone(isEnable);
```



#### 设备管理

```
// 开启/关闭自己的摄像头
rtcClient.enableCamera(isEnable);

// 开启/关闭自己麦克风
rtcClient.enableMicrophone(isEnable);
```



#### IM/信令传输相关功能

通过rtcClient.rtmClient 可获取内置的信令传输服务实例，可实现IM及信令传输相关能力，详细文档可参考： [rtm-client-sdk-websocket](https://github.com/highras/rtm-client-sdk-websocket)