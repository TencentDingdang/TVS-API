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

 
