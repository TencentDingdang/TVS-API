# SpeechSynthesizer接口

SpeechSynthesizer接口用于返回服务器端的语音响应。 例如，当用户问“上海的天气怎么样？”  时，服务器端将向客户端返回Speak指令，并带有二进制音频，客户端应该处理、播放该音频。

## 1 状态

SpeechSynthesizer具有以下状态：

**PLAYING(播放)**：当客户端在播放音频时，SpeechSynthesizer应该处于播放状态。 当客户端播放完成时，SpeechSynthesizer应转换到完成状态。
**FINISHED(完成)**：当客户端播放完音频时，SpeechSynthesizer应该上报SpeechFinished事件之后转换到FINISHED状态。

## 2 SpeechSynthesizer 上下文

如果客户端接入SpeechSynthesizer相关功能，需要在Context中上报播放器状态（状态），以及当前正在播放的TTS的offsetInMilliseconds。

**示例**
```java
{
    "header": {
        "namespace": "SpeechSynthesizer",
        "name": "SpeechState"
    },
    "payload": {
        "token": "{{STRING}}",
        "offsetInMilliseconds": {{LONG}},
        "playerActivity": "{{STRING}}"
    }
}
```


**payload参数**

| 参数                 | 描述                                                         | 类型   |
| -------------------- | ------------------------------------------------------------ | ------ |
| token                | 代表Speak指令的令牌。Speak指令会下发| string |
| offsetInMilliseconds | 标识TTS的当前偏移量（以毫秒为单位）| long   |
| playerActivity       | 标识SpeechSynthesizer的状态。 接受的值：PLAYING或FINISHED。  | string |

| playerActivity | 描述                  |
| --------------- | ---------------------------- |
| PLAYING         | 语音正在播放        |
| FINISHED        | 语音已经播放完毕 |

## 3 Speak 指令

在大多数情况下，Speak指令是为响应用户请求而发送的，例如Recognize事件。 

该指令作为multipart消息发送给您的客户端：一部分是JSON格式的指令和一部分二进制音频附件。
**代码示例**
```java
{
    "directive": {
        "header": {
            "namespace": "SpeechSynthesizer",
            "name": "Speak",
            "messageId": "{{STRING}}",
            "dialogRequestId": "{{STRING}}"
        },
        "payload": {
            "url": "{{STRING}}",
            "format": "{{STRING}}",
            "token": "{{STRING}}"
        }
    }
}
```

**Binary Audio Attachment**

每个Speak指令将具有相应的二进制音频附件作为multipart回包的一部分。 以下multipart头位于二进制音频附件之前：
```java
Content-Type: application/octet-stream
Content-ID: {{Audio Item CID}}

{{BINARY AUDIO ATTACHMENT}}
```


**Header参数**

| 参数            | 描述                                                         | 类型   |
| --------------- | ------------------------------------------------------------ | ------ |
| messageId       | 用于表示特定消息的唯一ID。                                   | string |
| dialogRequestId | 用于将response中的指令与特定的Recognize事件关联起来的唯一ID。 | string |



**payload参数**

| 参数   | 描述                                                         | 类型   |
| ------ | ------------------------------------------------------------ | ------ |
| url    | 音频内容的唯一标识符。 URL始终遵循前缀cid：。例如：CID：{{STRING}}。音频内容存放在以CID为Content-ID的multipart数据中。 | string |
| format | 返回音频的格式。 可接受的值：“AUDIO_MPEG”（mp3）。            |string |
| token  | 当前的Speak指令的令牌    | string |

## 4 SpeechStarted 事件

在客户端处理完Speak指令并开始播放语音时，应将SpeechStarted事件发送到腾讯云小微。

**代码示例**
```java
{
    "event": {
        "header": {
            "namespace": "SpeechSynthesizer",
            "name": "SpeechStarted",
            "messageId": "{{STRING}}"
        },
        "payload": {
            "token": "{{STRING}}"
        }
    }
}
```


**Header参数**

| 参数      | 描述                       | 类型   |
| --------- | -------------------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |



**payload参数**

| 参数  | 描述                                 | 类型   |
| ----- | ------------------------------------ | ------ |
| token | 代表Speak指令的令牌。Speak指令会下发 | string |

## 5 SpeechFinished 事件

客户端处理Speak指令并且音频完全播放完后发送SpeechFinished事件。 如果播放未完成，例如用户唤醒打断了播报，则不发送SpeechFinished。

**代码示例**
```java
{
    "event": {
        "header": {
            "namespace": "SpeechSynthesizer",
            "name": "SpeechFinished",
            "messageId": "{{STRING}}"
        },
        "payload": {
            "token": "{{STRING}}"
        }
    }
}
```


**Header参数**

| 参数      | 描述                       | 类型   |
| --------- | -------------------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |



**payload参数**

| 参数  | 描述                            | 类型   |
| ----- | ------------------------------- | ------ |
| token | 代表Speak指令的令牌。Speak指令会下发 | string |
