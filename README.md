

<h1>TVS API接入指南</h1>
[TOC]
# 一. 接入前置要求

## 1.1 平台要求
- 在云小微开放平台申请应用并发布。接入流程见[这里](https://dingdang.qq.com/doc/page/330 )
- 接入[账号授权流程](https://dingdang.qq.com/doc/page/365)
- 设备应**直接**对接TVS API，不能通过服务器中转请求。否则会被限频。

## 1.2 设备要求

### 1.2.1 硬件要求
#### 1.2.1.1 声学前端

语音识别对声学前端无特定要求，但是需要录音尽量清晰、少噪音。


### 1.2.2 软件要求

能力|是否必须|要求
-|-|-
媒体播放|非必须|如果开通了云小微音乐、FM等需要设备播放媒体的技能。设备的媒体播放器需要支持[这些格式](https://github.com/TencentDingdang/tvs-tools/tree/master/evaluate#32-%E5%AA%92%E4%BD%93%E6%92%AD%E6%94%BE%E8%83%BD%E5%8A%9B%E8%AF%84%E6%B5%8B)，并且设备需要测试这些格式音频是否能正常播放。
TTS播放|必须|需要支持mp3音频流的播放。

# 二. 接入概述

TVS API基于HTTP/2协议提供服务，如果设备使用普通HTTPS/HTTP请求，返回数据很可能不正常，常见的HTTP/2库见[这里](https://www.baidu.com)。如果是RTOS设备，可以接入云小微的RTOS SDK。

## 2.1 接口地址

TVS API主要涉及三个接口：

接口|说明
-|-
下行通道接口| HTTP长连接，用来接收从服务器端下发的`StopCapture`或者其他指令。 
 事件上报接口 |上报事件，并接收从服务器端下发的指令。
Ping接口|定时请求云端，用来保活HTTP/2物理连接。

接口地址分为正式环境、体验环境、测试环境三套环境。
厂商接入和发布产品时，终端 **一定** 要使用正式环境。
当需要和云小微联调新功能时，可能需要用体验环境、测试环境来联调。

### 正式环境地址

接口|地址
-|-
下行通道接口|https://tvs.html5.qq.com/v20160207/directives
 事件上报接口 |https://tvs.html5.qq.com/v20160207/events
Ping接口|https://tvs.html5.qq.com/ping

### 体验环境地址

接口|地址
-|-
下行通道接口|https://tvsexp.html5.qq.com/v20160207/directives
事件上报接口|https://tvsexp.html5.qq.com/v20160207/events
Ping接口|https://tvsexp.html5.qq.com/ping

### 测试环境地址

接口|地址
-|-
下行通道接口|https://tvstest.html5.qq.com/v20160207/directives
 事件上报接口 |https://tvstest.html5.qq.com/v20160207/events
Ping接口|https://tvstest.html5.qq.com/ping

## 2.2 协议概述

TVS API协议分为HTTP数据协议、事件指令协议两层。

事件、指令在HTTP数据协议中传输。HTTP数据协议定义

### 2.2.1 HTTP数据协议

HTTP数据协议，定义了：

- 设备通过什么样的HTTP数据协议请求服务端。

- 服务端通过什么样的HTTP数据协议回包

云小微对请求的HTTP Header有特殊要求，其字段名、含义、及其拼接方式见[TVS API Http Header文档](<https://dingdang.qq.com/doc/page/384>)

#### 2.2.1.1 事件上报接口

`POST /v20160207/events`

##### 请求

###### 普通事件上报协议

普通事件上报请求，基于[HTTP multipart](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)协议。事件协议数据在一个name为`metadata`的part中传递。

缺少图



HTTP Header

```
authorization： Bearer {{ACCESS_TOKEN}}
tvssettings: {{TVS_SETTINGS}}
q-ua: {{Q-UA}}
content-type: multipart/form-data; boundary=this-is-a-boundary
Content-Length: {{HTTP Body长度}}
```

HTTP Body

```
--this-is-a-boundary
Content-Disposition: form-data; name="metadata"
Content-Type: application/json; charset=UTF-8
Content-Length: {{事件协议字节长度}}

{{事件协议}}
--this-is-a-boundary--

```

###### `SpeechRecognizer.Recognize`事件传输协议(语音识别)

`SpeechRecognizer.Recognize`采用[HTTP chunk](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding)来传输请求数据。

`SpeechRecognizer.Recognize`事件上报时，事件协议数据在一个name为`metadata`的part中传递，音频二进制数据放在一个name为`audio`的part中。设备在录音时，应当边录音变上传，建议100ms上传一次，每次上传的音频，应放在一个单独的chunk中。

缺少图

HTTP Header

```
authorization： Bearer {{ACCESS_TOKEN}}
tvssettings: {{TVS_SETTINGS}}
q-ua: {{Q-UA}}
content-type: multipart/form-data; boundary=this-is-a-boundary
Transfer-Encoding: chunked
```

HTTP Body
```
{{CHUNK_SIZE}}
--this-is-a-boundary
Content-Disposition: form-data; name="metadata"
Content-Type: application/json; charset=UTF-8
Content-Length: {{事件协议字节长度}}

{{SpeechRecognizer.Recognize事件协议}}
--this-is-a-boundary
Content-Disposition: form-data; name="audio"
Content-Type: application/octet-stream

{{CHUNK_SIZE}}
{{音频数据}}
{{CHUNK_SIZE}}
{{音频数据}}
{{CHUNK_SIZE}}
{{音频数据}}
--this-is-a-boundary

{{CHUNK结束符}}

```

##### 回包

回包有四种形式: chunk回包、multipart回包、HTTP 500回包、HTTP 204回包

###### chunk回包

** 不带语音音频 **

HTTP Header

```
SessionId: {{SessionId}}
Content-Type: multipart/related; boundary=this-is-a-boundary; type="application/json"
Transfer-Encoding: chunked
```

HTTP Body
```
{{CHUNK_SIZE}}
--this-is-a-boundary
Content-Type: application/json; charset=UTF-8

{{指令协议}}
{{CHUNK_SIZE}}
--this-is-a-boundary
Content-Type: application/json; charset=UTF-8

{{指令协议}}
--this-is-a-boundary
Content-Type: application/json; charset=UTF-8

{{指令协议}}
--this-is-a-boundary--
{{CHUNK结束符}}

```

缺少图

** 带语音音频 **



予以你音频会在`Content-Type: application/octet-stream`的part中，并且有一个`Content-ID`字段，`SpeechSynthesizer.Speak`指令会标识该指令使用哪个Content-ID的音频数据。

HTTP Header

```
SessionId:{{SessionId}}
Content-Type: multipart/related; boundary=this-is-a-boundary; type="application/json"
Transfer-Encoding: chunked
```

HTTP Body
```
{{CHUNK_SIZE}}
--this-is-a-boundary
Content-Type: application/json; charset=UTF-8

{{指令协议}}
{{CHUNK_SIZE}}
--this-is-a-boundary
Content-Type: application/json; charset=UTF-8

{{指令协议}}
--this-is-a-boundary
Content-Type: application/octet-stream
Content-ID: <{{ID}}> 

{{CHUNK_SIZE}}
{{音频数据}}
{{CHUNK_SIZE}}
{{音频数据}}
{{CHUNK_SIZE}}
{{音频数据}}
{{CHUNK_SIZE}}
--this-is-a-boundary
Content-Type: application/json; charset=UTF-8

{{指令协议}}
--this-is-a-boundary--
{{CHUNK结束符}}

```

缺少图

###### multipart回包

HTTP Header

```
SessionId:{{SessionId}}
Content-Type: multipart/related; boundary=this-is-a-boundary; type="application/json"
```

HTTP Body
```
--this-is-a-boundary
Content-Type: application/json; charset=UTF-8

{{指令协议}}
--this-is-a-boundary
Content-Type: application/json; charset=UTF-8

{{指令协议}}
--this-is-a-boundary
Content-Type: application/json; charset=UTF-8

{{指令协议}}
--this-is-a-boundary--
0

```

###### HTTP 500回包

当HTTP Status为500时，服务器端会在HTTP Body中直接下发System.Exception指令，该指令用来标识服务器端发生的错误。

HTTP Header

```
SessionId: {{SessionId}}
```
HTTP Body
```
{{System.Exception指令协议}}
```

缺少图

###### HTTP 204回包

当事件上报后，没有其他指令返回时，服务器端将会返回HTTP Status为204，HTTP body内没有有效数据。如上报AudioPlayer.PlayStarted，服务器会返回HTTP 204.



##### 异常处理

当返回HTTP Status不为200，204时，设备应该当异常处理，并打印详细的出错日志。

#### 2.2.1.2 下行通道接口

`GET /v20160207/directives`

下行通道接口用来接收从服务器端下发的指令，设备应设置这个接口请求的超时时间至少为60分钟以上，服务器可能会在回包中不定时下发指令。设备应解析chunk回包或multipart回包中的指令处理。

##### 请求

HTTP Header

```
authorization： Bearer {{ACCESS_TOKEN}}
tvssettings: {{TVS_SETTINGS}}
q-ua: {{Q-UA}}
Content-Length: 0
```

##### 回包

回包格式为chunk回包或者multipart回包，与事件上报接口回包数据一致。

##### 异常处理

当返回HTTP Status不为200，或者设备遇到HTTP流被断掉、超时等异常情况时，设备应断掉当前HTTP/2物理连接，并建立新的HTTP/2物理连接，并在新建立的物理上请求下行通道接口。





#### 2.2.1.3 Ping接口

`GET /ping`

本接口用来保活HTTP/2物理连接。因运营商限制，HTTP/2物理连接在一段时间不活跃后，就会被运营商断掉。因此设备应在HTTP/2物理连接上，定时请求`/ping`接口。建议一分钟请求一次。

##### 请求

HTTP Header

```
authorization： Bearer {{ACCESS_TOKEN}}
tvssettings: {{TVS_SETTINGS}}
q-ua: {{Q-UA}}
Content-Length: 0
```

HTTP Body

空

##### 回包

回包格式与事件上报接口一致。

##### 异常处理

当返回HTTP Status不为200、204，或者设备超时等异常情况时，设备应断掉当前HTTP/2物理连接，并建立新的HTTP/2物理连接，并在新建立的物理上请求下行通道接口，。

### 2.2.2 事件、指令协议

 事件、指令是在HTTP中传输的数据应用层协议。

 __指令（directive）__ 是服务端下发给设备端，设备端需要执行的操作。比如播放一个语音（`SpeechSynthesizer.Speak`指令），设置一个闹钟（`Alerts.SetAlert`指令），播放一个音乐（`AudioPlayer.Play`指令）等等。

 __事件（event）__ 是设备端上报给服务端，通知服务端在设备端发生的事情。比如音乐播放开始了（`AudioPlayer.PlaybackStarted`事件），音乐播放结束了（`AudioPlayer.PlaybackFinished`事件），闹铃开始响了（`Alert.AlertStarted`事件），设备被唤醒并开始接受用户语音请求（`SpeechRecognizer.Recognize`事件）等等。



指令和事件是TVS API协议最基本的要素，设备端上发生的变化都通过上报相应的事件来通知服务端，服务端通过下发指令给设备端，对用户请求进行响应。

设备端在上报**某些**事件时，需要带上设备端的 __端状态（Context）__ 信息。比如当前是否有音乐正在播放，播放到哪里了（AudioPlayer.PlaybackState），设备端是否有设置闹铃，闹铃状态（Alerts.AlertsState）等等。对用户的请求，服务端结合端目前所处的状态，决定合理的响应，下发相应的指令。


指令协议一般是这样的：

```json
{
    "directive":{
        "header":{
             "namespace":"{{STRING}}",
             "name":"{{STRING}}",
             "messageId":"{{STRING}}",
             "diaglogRequestId":"{{STRING}}"
         },
         "payload":{
         }    
	}
}
```

字段|类型|是否必须|说明
-|-|-|-
directive|object|必须| -  
directive.header|object|是|-
directive.header.namespace|string|是|指令命名空间，如Speaker
directive.header.name|string|是|指令名，如SetVolume，namespace和name组合可以唯一确定一种指令。
directive.header.messageId|string|是|消息唯一标识
directive.header.diaglogRequestId|string|否|设备会话标识，此标识由设备发起某些事件(如SpeechRecognizer.Recognize)时带给服务器端，服务器端原样带回。每个设备应该保证diaglogRequestId唯一性，以免产生会话混乱问题，建议diaglogRequestId至少以20位随机字符串组成。
directive.payload|object|是|指令的自定义数据，每种指令都有自己的数据格式定义。具体详见指令协议



事件协议一般是这样的：

```json
{
    "context":[
        {
             "header":{
                 "namespace":"{{STRING}}",
                 "name":"{{STRING}}",
             },
             "payload":{
             }  
        }
    ],
    "event":{
        "header":{
             "messageId":"{{STRING}}",
             "namespace":"{{STRING}}",
             "name":"{{STRING}}",
             "diaglogRequestId":"{{STRING}}"
         },
         "payload":{
         }    
	}
}
```

字段|类型|是否必须|说明
-|-|-|-
context|array|否|设备状态集，只有一部分事件需要携带这个结构。当事件要求传递Context时，设备应把所有状态都放到Context中，不能仅选择一部分放入。 
context.header|object|是|-
context.header.namespace|string|是|状态命名空间，如AudioPlayer
context.header.name|string|是|发送的状态名，如PlayBackState，namespace和name组合可以唯一确定一种状态
context.payload|object|是|状态的自定义数据，每种状态都有自己的数据格式定义。具体详见状态协议
event|object|必须| -  
event.header|object|是|-
event.header.namespace|string|是|事件命名空间，如Speaker
event.header.name|string|是|发送的事件名，如VolumeChanged，namespace和name组合可以唯一确定一种事件
event.header.messageId|string|是|消息唯一标识，设备端应尽量保证每个事件都不一样。
directive.header.diaglogRequestId|string|否|设备会话标识，此标识由设备发起某些事件(如SpeechRecognizer.Recognize)时带给服务器端，服务器端在下发某些指令时会原样带回。每个设备应该保证diaglogRequestId唯一性，以免产生会话混乱问题，建议diaglogRequestId至少以20位随机字符串组成。
event.payload|object|是|事件的自定义数据，每种事件都有自己的数据格式定义。具体详见事件协议

TVS API定义了一系列directive与event、context，除了与AVS兼容外，还定义了扩展的能力。具体协议见[协议文档](https://github.com/TencentDingdang/TVS-API/blob/master/README.md)

## 2.3 约定规则

### 2.3.1 HTTP/2连接与下行通道

设备与TVS API建立的HTTP/2物理连接只能有一个。

设备开机，进行票据刷票后，按照下面流程进行初始化工作：

- 初始化HTTP/2物理连接
- 请求下行通道接口
- 上报System.SynchronizeState事件，向服务器端同步终端的状态。

当遇到以下问题时，设备应该关闭之前的HTTP/2物理连接，并重新建立新的HTTP/2物理连接，进行初始化工作

- 调用Ping接口超时
- 下行通道接口连接超时或者抛异常。
- HTTP/2物理连接抛异常

当设备上报语音识别事件已经收到对应的语音播放事件后，长时间（10s)无法收到从下行通道收到的`StopCapture`指令，应该重新建立下行通道。

###  2.3.2 设备账户切换

当设备账户发生切换时，应该进行以下下步骤：

- 更新Authorization
- 关闭之前的下行通道接口请求stream，重新请求下行通道接口
- 重置context，清空闹钟列表
- 上报System.SynchronizeState事件，向服务器端同步设备的状态。


###  2.3.3 diaglogRequestId
每个设备应该保证diaglogRequestId唯一性，以免产生会话混乱问题，建议diaglogRequestId至少以20位随机字符串组成。

###  2.3.4 Context

有些事件上报时，需要设备状态，设备应该将context中的所有状态都带着，不能仅选择一部分状态。

另，在设备开机后，如果进行过语音播报，必须将播报状态放在context中。

### 2.3.5 无法识别的字段
为了不断增强云小微能力，服务端在必要时会进行协议升级。为了保持对已有设备的向后兼容，服务端在进行协议升级时，会新增加字段，但不会改变已有字段。

设备端收到指令，如果指令的header参数或者payload参数多出了不能识别的字段，设备端应该忽略所有不能识别的字段，正常执行指令。

### 2.3.6 无法识别的指令

服务端协议升级也会增加新指令，但正常情况下服务端不应该对现有设备下发无法识别的新指令。如果设备端收到了无法识别的指令，应该上报`System.ExceptionEncountered`事件，但不应该终止设备端应用。其他指令应该正常执行，应用也应该继续正常运行。

###   2.3.7 指令执行顺序

设备端收到的带有dialogRequestId的指令，应该按收到的顺序一个一个执行。对于Speak指令，把播报播完才是指令执行结束，再执行下一条指令。对于Play指令，只要把音频放到队列或者开始进行播放，即指令执行结束，可以继续执行下一条指令。

大部分指令定义中都是带dialogRequestId的，但dialogRequestId字段是可选的，服务端可以根据情况下发不带dialogRequestId的指令，设备端收到不带有dialogRequestId的指令，应该立即执行（与其他指令独立并行执行）。

### 2.3.8 多通道音频播放

#### 音频播放通道

具有扬声器的设备端，应该支持多个音频播放通道，对应不同的端能力。TVS定义了以下三种音频播放通道：

1. **对话通道**：对应语音输入（Voice Input）和语音输出（Voice Output）端能力；用户在语音请求时，或者设备在执行Speak指令进行播报时，对话通道进入活跃状态
2. **闹钟通道**：对应闹钟（Alerts）端能力；在闹铃响起时，闹钟通道进入活跃状态
3. **音频通道**：对应音频播放器（Audio Player）端能力；在播放音频（音乐、新闻、有声资源）时，音频通道进入活跃状态

#### 播放优先级

如果同一时刻有多个播放通道同时处在活跃状态（如正在播放音乐时闹铃响起），优先级最高的通道应该在前景（foreground），其他低优先级通道应该移到背景（background）。在最高优先级的播放通道播放完毕并退出活跃状态后，下一优先级的播放通道从背景移到前景。

移到背景的播放通道，可以暂停播放，也可以把声音减弱播放。

播放通道优先级：对话通道 > 闹钟通道 > 音频通道




# 三、接入TVS API流程

## 3.1 获取账号登录信息

设备访问TVS API，需要先 [获取腾讯云小微帐号体系的访问票据](https://dingdang.qq.com/doc/page/365) 。该票据授权设备代表用户，向腾讯云小微发起调用。
腾讯云小微帐号地址:https://tvs.html5.qq.com/auth/o2/token
除了账号方面的接口，每个请求都应当携带访问票据。携带方法：
在HTTP Header中，添加Authorization头承载访问票据。
格式如下

```
Authorization: Bearer 访问票据
```
如

```
Authorization: Bearer yourAccessToken
```
基于Android或iOS厂商App方式进行帐号认证的产品接入流程，请参考： [腾讯云小微厂商App账号接入流程](https://dingdang.qq.com/doc/page/365)

## 3.2 选择功能进行接入

TVS API提供一系列功能，设备可以选择性的接入一部分能力。

建议按照如下顺序接入：

- [账号授权与刷票](https://github.com/TencentDingdang/TVS-API/blob/master/README.md)（必须）
- [语音识别能力和语音朗读](https://github.com/TencentDingdang/TVS-API/blob/master/语音识别和语音播报接入指南.md)（必须）
- [公共能力](https://github.com/TencentDingdang/TVS-API/blob/master/README.md)（必须）
- [扬声器管理能力](https://github.com/TencentDingdang/TVS-API/blob/master/扬声器管理.md)（可选）
- [自定义技能](https://github.com/TencentDingdang/TVS-API/blob/master/自定义技能.md)（可选）
- [媒体播放控制-语音控制能力](https://github.com/TencentDingdang/TVS-API/blob/master/媒体播放控制-语音控制能力.md)（可选）
- [媒体播放控制-按钮控制能力](https://github.com/TencentDingdang/TVS-API/blob/master/媒体播放控制-按钮控制能力.md)（可选）
- [闹钟](https://github.com/TencentDingdang/TVS-API/blob/master/README.md)（可选）
- [原子能力](https://github.com/TencentDingdang/TVS-API/blob/master/README.md)（可选）
- [媒体播放控制-远程点播控制](https://github.com/TencentDingdang/TVS-API/blob/master/README.md)（可选）
- [Push能力](https://github.com/TencentDingdang/TVS-API/blob/master/README.md)（可选）
- [语音识别-oneshoot](https://github.com/TencentDingdang/TVS-API/blob/master/README.md)（可选）
- [全双工能力](https://github.com/TencentDingdang/TVS-API/blob/master/README.md)（可选）
- [UI能力](https://github.com/TencentDingdang/TVS-API/blob/master/README.md)（可选）
- [情绪系统](https://github.com/TencentDingdang/TVS-API/blob/master/%E6%83%85%E7%BB%AA%E7%B3%BB%E7%BB%9F%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97.md)（可选）













