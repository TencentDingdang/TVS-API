# TvsPushInterface接口

本接口提供了TVS Push相关能力

## Context
无

## TransparentMessage指令

触发时机：

- 手机解绑账户
- 手机与音箱进行多端互动，探测设备在线状态时下发。
- 其他情况~

```json
{
    "directive": {
        "header": {
            "namespace": "TvsPushInterface",
            "name": " TransparentMessage",
            "messageId": "{{STRING}}"
         },
        "payload": {
            "messages":[{
                    "type":"{{STRING}}",
                    "text":"{{STRING}}",
                    "token":"{{STRING}}"
                }
            ]
        }
}

```

***Header Paramters***

|    Parameter            |    Type        |    必选    |    描述                                |
|    :-------------------    |    :--------    |    :-----    |    :--------------------------------    |
|    messageId            |    string    |    Yes    |    指令ID                            |
|    dialogRequestId    |    string    |    Yes    |    对话ID                            |

***Payload Paramters** 

| Parameter      | Type   | 必选 | 描述             |
| :------------- | :----- | :--- | :--------------- |
| messages       | list   | Yes  | 多条消息         |
| messages.type  | string | Yes  | 消息类型         |
| messages.text  | string | No   | 消息内容         |
| messages.token | string | Yes  | Push消息唯一标识 |

## Acknowledgement事件

当设备侧收到服务器端下发的TransparentMessage指令时，应当立即通过本事件对下发的消息进行确认，以表示设备收到TransparentMessage指令。

```json
{
    "event": {
        "header": {
            "namespace": "TvsPushInterface",
            "name": " Acknowledgement",
            "messageId": "{{STRING}}"
         },
        "payload": {
            "tokens":[ {{LIST}} ]
        }
}

```

***Header Paramters***

|    Parameter            |    Type        |    必选    |    描述                                |
|    :-------------------    |    :--------    |    :-----    |    :--------------------------------    |
|    messageId            |    string    |    Yes    |    消息ID                            |

***Payload Paramters** 

| Parameter | Type | 必选 | 描述                                   |
| :-------- | :--- | :--- | :------------------------------------- |
| tokens    | list | Yes  | TransparentMessage的messages.token数组 |