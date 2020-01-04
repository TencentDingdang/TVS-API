# SpeechRecognizer接口

每个用户语音交互都会用到SpeechRecognizer。 它是TVS API的核心接口，提供了用于捕获用户语音的指令和事件，并在服务器端需要额外的语音输入时提示客户端。


## 1 状态图

下图说明了SpeechRecognizer状态流转图。框表示SpeechRecognizer状态，连线表示状态转换。
SpeechRecognizer具有以下状态：

**IDLE（空闲）**：在捕获用户语音之前，SpeechRecognizer应处于空闲状态。在与服务器端的语音交互结束后，SpeechRecognizer也应该返回到空闲状态。成功处理语音请求或ExpectSpeechTimedOut事件出现，可能会发生这种情况。
另外，SpeechRecognizer可以在多轮交互期间返回到空闲状态，此时，如果服务器端需要用户的语音输入，它应该从IDLE状态转换到期望语音状态而无需用户主动开始新的交互。
**RECOGNIZING（识别）**：当用户开始与您的客户端进行交互时，特别是当捕获的音频流式传输到服务器端时，SpeechRecognizer应该从空闲状态转换到识别状态。它应保持在识别状态，直到客户端停止录制语音（或流式传输完成），此时您的SpeechRecognizer应从识别状态转换为忙碌状态。
**BUSY（忙碌）**：处理语音请求时，SpeechRecognizer应处于忙碌状态。在组件转出忙碌状态之前，您无法启动另一个语音请求。从忙状态，如果请求被成功处理（完成），则SpeechRecognizer将转换到空闲状态，或者如果服务器端需要来自用户的额外语音输入，则SpeechRecognizer将转换到期望语音状态。
**EXPECTING SPEECH（期望语音）**：当用户需要额外的音频输入时，SpeechRecognizer应处于期望语音状态。从期望语音开始，SpeechRecognizer应当在用户交互发生时转换到识别状态，或者代表用户自动开始交互。如果在指定的时间内未检测到用户交互，则应转换为空闲状态。

![tvs_speechrecognizer_state.png](https://3gimg.qq.com/trom_s/dingdang/upload/20191127/facca44b9a131a6e181dfa1f5ee00362.png)

## 2 SpeechRecognizer 上下文

如果本次语音识别是语音唤醒发起的，客户端需要上报当前设置的唤醒词。

**示例**

```java
{
    "header": {
        "namespace": "SpeechRecognizer",
        "name": "RecognizerState"
    },
    "payload": {
        "wakeword": "DING1DANG1DING1DANG"
    }
}
```

**有效负载参数**

| 参数     | 描述                                                        | 类型   |
| -------- | ----------------------------------------------------------- | ------ |
| wakeword | 表示当前唤醒词，接受的值："DING1DANG1DING1DANG"。 | string |

## 3 Recognize 事件

Recognize事件用于将用户语音发送到服务器端并将该语音转换为一个或多个指令。 此事件必须作为 multipart消息发送：第一部分是JSON格式的对象，第二部分是二进制音频，整个请求需要用chunk编码传输。 每块语音流应包含100ms的音频。
在启动与服务器端的交互后，麦克风必须保持打开状态，直到：

- 收到StopCapture指令。
- 服务器端关闭了该流。
- 用户手动关闭麦克风。
- 客户端集成了本地vad，本地vad检测说话结束。

  

**示例**

```java
{
    "context": [
        ...  
    ],   
    "event": {
        "header": {
            "namespace": "SpeechRecognizer",
            "name": "Recognize",
            "messageId": "{{STRING}}",
            "dialogRequestId": "{{STRING}}"
        },
        "payload": {
            "format": "{{STRING}}",
            "initiator": {
                "type": "{{STRING}}",
                "payload": {
                    "wakeWordIndices": {
                        "startIndexInSamples": {{LONG}},
                        "endIndexInSamples": {{LONG}}
                    },
                    "token": "{{STRING}}"   
                }
            }
        }
    }
}
```

**二进制音频传输**

每个Recognize事件都需要相应的二进制音频附件作为multipart消息的一部分。 每个二进制音频附件都需要以下标头：
```java
Content-Disposition: form-data; name="audio"
Content-Type: application/octet-stream

{{BINARY AUDIO ATTACHMENT}}
```

**Context**

本事件需要设备携带Context结构。

**Header参数**

| 参数              | 描述                                       | 类型     |
| --------------- | ---------------------------------------- | ------ |
| messageId       | 用于表示特定消息的唯一ID。                           | string |
| dialogRequestId | 用于将response中的指令与特定的Recognize事件关联起来的唯一ID。 | string |



**payload参数**

| 参数                                                  | 描述                                                         | 类型   |
| ----------------------------------------------------- | ------------------------------------------------------------ | ------ |
| format                                                | 音频格式。可接受的值见后面。                                 | string |
| initiator                                             | 标识Recognize事件是如何启动的。当用户主动发起交互时（语音唤醒，点击开始），此对象是必需的。如果ExpectSpeech指令中存在initiator字段，则必须在随后的Recognize事件中带上它。如果ExpectSpeech指令中不存在initiator字段，则Recognize事件中也不应该存在这个字段。 | object |
| initiator.type                                        | 表示用户启动交互所采取的操作类型。 可接受的值：TAP(点击开始录音)和WAKEWORD(唤醒启动)。 | string |
| initiator.payload                                     | -                                                            | object |
| initiator.payload.wakeWordIndices                     | 当initiator.type设置为WAKEWORD时，此对象是必需的。           | object |
| initiator.payload.wakeWordIndices.startIndexInSamples | 表示唤醒词在音频流中开始位置（样本数）。 开始位置应精确到唤醒词检测开始的50ms以内。 | long   |
| initiator.payload.wakeWordIndices.endIndexInSamples   | 表示唤醒词在音频流中结束位置（样本数）。 结束位置应精确到唤醒词检测结束的的150ms以内。 | long   |
| initiator.payload.token                               | token。如果由ExpectSpeech指令触发，填入ExpectSpeech的`initiator.payload.token`。否则留空 | string |

**format**

format参数告诉服务器端，客户端的音频格式是怎样的，接受以下值

值|格式|采样率|位深度|声道数|字节序|备注
-|-|-|-|-|-|-
AUDIO_SPEEX_L16_RATE_16000_CHANNELS_1|Speex|16000|16位|单声道|小端模式|需要特定版本，不建议使用
AUDIO_SPEEX_L16_RATE_8000_CHANNELS_1|Speex|8000|16位|单声道|小端模式|需要特定版本，不建议使用
AUDIO_OPUS_L16_RATE_16000_CHANNELS_1|Opus|16000|16位|单声道|小端模式|
AUDIO_OPUS_L16_RATE_16000_CHANNELS_1|Opus|16000|16位|单声道|小端模式|
AUDIO_PCM_L16_RATE_8000_CHANNELS_1|PCM|8000|16位|单声道|小端模式|
AUDIO_PCM_L16_RATE_16000_CHANNELS_1|PCM|8000|16位|单声道|小端模式|

**Initiator**

initiator参数告诉服务器端交互是如何触发的，并确定两件事：
1. 如果在云端检测到语音结束后，StopCapture将被发送给您的客户端。
2. 如果要在语音流上执行基于云的唤醒词验证。initiator必须包含在每个SpeechRecognizer.Recognize事件的payload中。 接受以下值：

| 值   | 描述   | 是否支持StopCapture | 是否校验唤醒词| 是否要求唤醒词偏移 |
| -------- | ---- | ----- | ---------- | ----- |
| TAP    | 音频流由点击和释放按钮（物理或GUI）启动，并在收到StopCapture指令时终止。  | Y | N  | N    |
| WAKEWORD| 通过语音唤醒启动的音频流，并在收到StopCapture指令时终止。 | Y| Y  | Y  |


## 4 StopCapture 指令



该指令代表服务器端已经识别到说话结束。 收到此指令后，客户端必须立即关闭麦克风并停止录音。
**注意**：StopCapture在下行通道发送到客户端

**示例**

```java
{
    "directive": {
        "header": {
            "namespace": "SpeechRecognizer",
            "name": "StopCapture",
            "messageId": "{{STRING}}",
            "dialogRequestId": "{{STRING}}"
        },
        "payload": {
        }
    }
}
```


**Header参数**

| 参数              | 描述                                       | 类型     |
| --------------- | ---------------------------------------- | ------ |
| messageId       | 用于表示指令消息的唯一ID。                           | string |
| dialogRequestId | 用于将response中的指令与特定的Recognize事件关联起来的唯一ID。 | string |

**payload参数**

空

## 5 ExpectSpeech 指令

当服务器端需要额外信息来满足用户的请求时，会发送ExpectSpeech指令。 它指示客户端打开麦克风并开始流式传输用户语音。 如果未在指定的超时窗口内打开麦克风，则必须从客户端向服务器端发送ExpectSpeechTimedOut事件。
在与服务器端进行多轮交互期间，客户端将至少收到一条ExpectSpeech指令，指示客户端开始收音。 如果客户端收到`ExpectSpeech`，则必须将ExpectSpeech指令的`payload.initialtor`字段作为触发的Recognize事件中的`payload.initialtor`字段传递回服务器端。 如果将ExpectSpeech指令不存在`payload.initialtor`字段，则触发的Recognize事件也不应包含`payload.initialtor`字段。

**代码示例**

```java
{
    "directive": {
        "header": {
            "namespace": "SpeechRecognizer",
            "name": "ExpectSpeech",
            "messageId": "{{STRING}}",
            "dialogRequestId": "{{STRING}}"
        },
        "payload": {
            "timeoutInMilliseconds": {{LONG}},
            "initiator": {
                "type": "{{STRING}}",
                "payload": {
                    "token": "{{STRING}}"
                }
            }
        }
    }
}
```

**Header参数**

| 参数              | 描述                                       | 类型     |
| --------------- | ---------------------------------------- | ------ |
| messageId       | 用于表示特定消息的唯一ID。                           | string |
| dialogRequestId | 用于将response中的指令与特定的Recognize事件关联起来的唯一ID。 | string |



**payload参数**

| 参数                    | 描述                                                         | 类型   |
| ----------------------- | ------------------------------------------------------------ | ------ |
| timeoutInMilliseconds   | 指定客户端应等待麦克风打开并开始将用户语音流式传输到服务器端的时间（以毫秒为单位）。 如果未在指定的超时时间内打开麦克风，则必须发送ExpectSpeechTimedOut事件。 此行为的主要用例是PRESS_AND_HOLD方案。 | long   |
| initiator               | 包含有关交互的信息。 如果存在，必须在后续Recognize事件中将其发送回服务器端。 | object |
| initiator.type          | 不透明的字符串。 如果存在，必须在后续Recognize事件中将其发送回服务器端。 | string |
| initiator.payload       | 包含有关initiator的信息                                      | object |
| initiator.payload.token | 不透明的字符串。如果存在，必须在后续Recognize事件中将其发送回服务器端。 | string |

## 6 ExpectSpeechTimedOut 事件

如果收到ExpectSpeech指令，但在指定的超时时间内没有执行，则必须将此事件发送到服务器端。

**示例**

```json
{
    "event": {
        "header": {
            "namespace": "SpeechRecognizer",
            "name": "ExpectSpeechTimedOut",
            "messageId": "{{STRING}}",
        },
        "payload": {
        }
    }
}
```

**Header参数**

| 参数        | 描述             | 类型     |
| --------- | -------------- | ------ |
| messageId | 用于表示事件的唯一ID。 | string |

**payload参数**

空
