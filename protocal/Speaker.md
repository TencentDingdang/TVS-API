# Speaker接口

Speaker接口提供了用于调整音量、静音/取消静音的指令和事件。 腾讯云小微支持两种音量调节方法：

1. 通过SetVolume指令;
2. 通过AdjustVolume指令。

## 1 上下文

腾讯云小微希望客户端在Context中上报Speaker接口的音量和静音状态信息。有关上报Context的详细信息，请参阅Context概述。
**协议**

```java
{
    "header": {
        "namespace": "Speaker",
        "name": "VolumeState"
    },
    "payload": {
        "volume": {{LONG}},
        "muted": {{BOOLEAN}}
    }
}
```


**有效负载参数**

| 参数   | 描述                                  | 类型    |
| ------ | ------------------------------------- | ------- |
| volume | 当前扬声器的音量。可接受的值：0-100。 | long    |
| muted  | 当前扬声器是否静音。                  | boolean |

## 2 SetVolume指令

该指令指示终端进行绝对音量调整。 音量值将介于0（最小）和100（最大）之间。
**协议**

```java
{
    "directive": {
        "header": {
            "namespace": "Speaker",
            "name": "SetVolume",
            "messageId": "{{STRING}}",
            "dialogRequestId": "{{STRING}}"
        },
        "payload": {
            "volume": {{LONG}}
        }
    }
}
```


**Header参数**

| 参数            | 描述                                                         | 类型   |
| --------------- | ------------------------------------------------------------ | ------ |
| messageId       | 本条指令唯一ID。                                             | string |
| dialogRequestId | 用于将response中的指令与特定的Recognize事件关联起来的唯一ID。 | string |



**有效负载参数**

| 参数   | 描述                                                         | 类型 |
| ------ | ------------------------------------------------------------ | ---- |
| volume | 绝对音量值，从0（分钟）到100（最大）。 可接受的值：0到100之间的任何值，包括0或者100。 | long |

## 3 AdjustVolume指令

该指令指示终端进行相对音量调整。 音量值介于-100和100之间。
AdjustVolume指令始终相对于当前音量设置，增加音量为正数，减小音量为负数。
**代码示例**

```java
{
    "directive": {
        "header": {
            "namespace": "Speaker",
            "name": "AdjustVolume",
            "messageId": "{{STRING}}",
            "dialogRequestId": "{{STRING}}"
        },
        "payload": {
            "volume": {{LONG}}
        }
    }
}
```


**Header参数**

| 参数            | 描述                                                         | 类型   |
| --------------- | ------------------------------------------------------------ | ------ |
| messageId       | 本条指令唯一ID。                                   | string |
| dialogRequestId | 用于将response中的指令与特定的Recognize事件关联起来的唯一ID。 | string |



**有效负载参数**

| 参数   | 描述                                                         | 类型 |
| ------ | ------------------------------------------------------------ | ---- |
| volume | 调整的相对音量。正数、负数分别用来增加、减小当前音量。可接受的值：介于-100和100之间的任何值，包括-100/100。 | long |

## 4 VolumeChanged事件
在以下情况下，必须将VolumeChanged事件发送到腾讯云小微：
- 接收并处理SetVolume或AdjustVolume指令，以扬声器音量已被调整/更改。
- 音量在本地调整，以指示的扬声器音量已经调整/更改。
  **重要**：音量必须是介于0（分钟）和100（最大）之间的值。 如果您的产品本地支持从0到10的音量调节，当用户将音量增加到8时，发送给腾讯云小微的音量值应为80。

**代码示例**
```java
{
    "event": {
        "header": {
            "namespace": "Speaker",
            "name": "VolumeChanged",
            "messageId": "{{STRING}}"
        },
        "payload": {
            "volume": {{LONG}},
            "muted": {{BOOLEAN}}
        }
    }
}
```


**Header参数**

| 参数      | 描述                       | 类型   |
| --------- | -------------------------- | ------ |
| messageId | 本条事件唯一ID。 | string |



**有效负载参数**

| 参数   | 描述                                                         | 类型    |
| ------ | ------------------------------------------------------------ | ------- |
| volume | 从0（分钟）到100（最大）的绝对音量。 可接受的值：0到100之间的任何整数值 | long    |
| mute   | 布尔值表示扬声器是否静音。 扬声器静音时该值为true，取消静音时为false。 | boolean |

## 5 SetMute 指令

此指令从腾讯云小微发送到客户端，以使扬声器静音。
**代码示例**
```java
{
    "directive": {
        "header": {
            "namespace": "Speaker",
            "name": "SetMute",
            "messageId": "{{STRING}}",
            "dialogRequestId": "{{STRING}}"
        },
        "payload": {
            "mute": {{BOOLEAN}}
        }
    }
}
```


**Header参数**

| 参数            | 描述                                                         | 类型   |
| --------------- | ------------------------------------------------------------ | ------ |
| messageId       | 本条指令唯一ID。                                   | string |
| dialogRequestId | 用于将response中的指令与特定的Recognize事件关联起来的唯一ID。 | string |



**有效负载参数**

| 参数 | 描述                                                         | 类型    |
| ---- | ------------------------------------------------------------ | ------- |
| mute | 布尔值用于设置扬声器是否需要静音。 扬声器静音时该值为true，取消静音时为false。 | boolean |

## 6 MuteChanged 事件


在下列情况下，必须将MuteChanged事件发送到腾讯云小微：

- 接收并执行SetMute指令，表示产品扬声器的静音状态已更改。
- 设备在本地执行静音/取消静音，表示产品扬声器的静音状态已更改。

**代码示例**
```java
{
    "event": {
        "header": {
            "namespace": "Speaker",
            "name": "MuteChanged",
            "messageId": "{{STRING}}"
        },
        "payload": {
            "volume": {{LONG}},
            "muted": {{BOOLEAN}}
        }
    }
}
```


**Header参数**

| 参数      | 描述                       | 类型   |
| --------- | -------------------------- | ------ |
| messageId | 本条事件唯一ID。 | string |



**有效负载参数**

| 参数   | 描述                                                         | 类型    |
| ------ | ------------------------------------------------------------ | ------- |
| volume | 从0（分钟）到100（最大）的绝对音量。 可接受的值：0到100之间的任何整数值。 | long    |
| mute   | 布尔值表示扬声器是否静音。 扬声器静音时该值为true，取消静音时为false。 | boolean |