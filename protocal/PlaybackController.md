# PlaybackController接口

PlaybackController接口用于通过按钮或者GUI来导航播放队列，如点击上一首按钮。

## 1 PlayCommandIssued事件

当用户使用客户端启动/恢复按钮或GUI启动/恢复媒体播放时，必须发送PlayCommandIssued事件。
**代码示例**

```java
{
    "context": [
    ],   
    "event": {
        "header": {
            "namespace": "PlaybackController",
            "name": "PlayCommandIssued",
            "messageId": "{{STRING}}"
        },
        "payload": {
        }
    }
}
```

**Context**

本事件要求客户端将所有状态发送到腾讯云小微。 有关其他信息，请参阅Context。



**Header参数**

| 参数        | 描述             | 类型     |
| --------- | -------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |



**有效负载参数**

应发送空的有效负载。

## 2 PauseCommandIssued事件

当用户使用按钮或GUI暂停正在播放的媒体时，必须发送PauseCommandIssued事件。
**代码示例**

```java
{
    "context": [
    ],    
    "event": {
        "header": {
            "namespace": "PlaybackController",
            "name": "PauseCommandIssued",
            "messageId": "{{STRING}}"
        },
        "payload": {
        }
    }
}
```

**Context**

本事件要求客户端将所有状态发送到腾讯云小微。 有关其他信息，请参阅Context。


**Header参数**

| 参数        | 描述             | 类型     |
| --------- | -------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |


**有效负载参数**

应发送空的有效负载。

## 3 NextCommandIssued事件

当用户使用按钮或GUI跳到其播放队列中的下一个媒体时，必须发送NextCommandIssued事件。
**代码示例**
```java
{
    "context": [
    ],     
    "event": {
        "header": {
            "namespace": "PlaybackController",
            "name": "NextCommandIssued",
            "messageId": "{{STRING}}"
        },
        "payload": {
        }
    }
}
```

**Context**
本事件要求客户端将所有状态发送到腾讯云小微。 有关其他信息，请参阅Context。



**Header参数**

| 参数        | 描述             | 类型     |
| --------- | -------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |



**有效负载参数**

应发送空的有效负载。

## 4 PreviousCommandIssued事件

当用户使用按钮或GUI时跳到其播放队列中的上一个媒体时，必须发送PreviousCommandIssued事件。
**代码示例**
```java
{
    "context": [
    ],     
    "event": {
        "header": {
            "namespace": "PlaybackController",
            "name": "PreviousCommandIssued",
            "messageId": "{{STRING}}"
        },
        "payload": {
        }
    }
}
```

**Context**

本事件要求客户端将所有状态发送到腾讯云小微。 有关其他信息，请参阅Context。



**Header参数**

| 参数        | 描述             | 类型     |
| --------- | -------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |

**有效负载参数**

应发送空的有效负载。

## 5 ButtonCommandIssued事件

此事件用于向腾讯云小微通知客户端的按钮或GUI被触发，例如向前或向后按钮被按下。 跳过持续时间由提供者/技能确定，并且每个事件都是附加的。 例如，如果用户连续三次按下向前按钮，结果三个ButtonCommandIssued事件被发送到腾讯云小微，则加起来（如果跳过时间为30秒）将是90秒。

**代码示例**
```java
{
    "context": [
    ],
    "event": {
        "header": {
            "namespace": "PlaybackController",
            "name": "ButtonCommandIssued",
            "messageId": "{{STRING}}"
        },
        "payload": {
            "name": "{{STRING}}"
        }
    }
}
```

**Context**

本事件要求客户端将所有状态发送到腾讯云小微。 有关其他信息，请参阅Context。



**Header参数**

| 参数        | 描述             | 类型     |
| --------- | -------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |



**有效负载参数**

| 参数   | 描述                                       | 类型     |
| ---- | ---------------------------------------- | ------ |
| name | 指定按钮/GUI触发的命令。可接受的值：SKIPFORWARD，SKIPBACKWARD。 | string |

## 6 ToggleCommandIssued事件

此事件用于通知腾讯云小微客户端已使用按钮或GUI选择或取消选项或功能。 支持的选项包括：shuffle，loop，repeat，thumbs up和thumbs down。
**代码示例**
```java
{
    "context": [
    ],
    "event": {
        "header": {
            "namespace": "PlaybackController",
            "name": "ToggleCommandIssued",
            "messageId": "{{STRING}}"
        },
        "payload": {
            "name": "{{STRING}}"
            "action": "{{STRING}}"
         }
    }
}
```

**Context**

本事件要求客户端将所有终端的状态发送到腾讯云小微。 有关其他信息，请参阅Context。


**Header参数**

| 参数        | 描述             | 类型     |
| --------- | -------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |



**有效负载参数**

| 参数     | 描述                                       | 类型     |
| ------ | ---------------------------------------- | ------ |
| name   | 指定由客户端按下按钮或GUI切换的选项/功能。 接受的值：SHUFFLE，LOOP，REPEAT，THUMBSUP，THUMBSDOWN。 | string |
| action | 指示是否已选择或取消选择。 接受的值：SELECT，DESELECT。      | string |

