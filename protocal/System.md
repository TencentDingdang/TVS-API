# System接口

System接口提供多个系统级事件、指令。

## 1 SynchronizeState 事件

建立新连接时，必须发送SynchronizeState事件到腾讯云小微以更新所有终端状态。

**代码示例**
```java
{
    "context": [
    ],      
    "event": {
        "header": {
            "namespace": "System",
            "name": "SynchronizeState",
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

| 参数      | 描述                       | 类型   |
| --------- | -------------------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |



**有效负载参数**

应发送空的有效负载。

 

## 2 ExceptionEncountered 事件

当客户端无法执行腾讯云小微下发的指令时，必须发送ExceptionEncountered事件。
**代码示例**
```java
{
    "context": [
        // This is an array of context objects that are used to communicate the
        // state of all client components to 腾讯云小微. See Context for details.      
    ],     
    "event": {
        "header": {
            "namespace": "System",
            "name": "ExceptionEncountered",
            "messageId": "{{STRING}}"
        },
        "payload": {
            "unparsedDirective": "{{STRING}}",
            "error": {
                "type": "{{STRING}}"
                "message": "{{STRING}}"
            }
        }
    }
}
```

**Context**

本事件要求客户端将所有状态发送到腾讯云小微。 有关其他信息，请参阅Context。

**Header参数**

| 参数      | 描述                       | 类型   |
| --------- | -------------------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |



**有效负载参数**

| 参数              | 描述                                                         | 类型   |
| ----------------- | ------------------------------------------------------------ | ------ |
| unparsedDirective | 当无法执行指令时，客户端必须将指令作为字符串返回给腾讯云小微。 | string |
| error             | 错误的键/值对。                                              | object |
| error.type        | 错误类型。错误类型见下面                                     | string |
| error.message     | 方便记录与问题定位的附加错误信息。                           | string |

错误类型

| Error类型                       | 描述                                                   |
| ------------------------------- | ------------------------------------------------------ |
| UNEXPECTED_INFORMATION_RECEIVED | 发送给客户端的指令格式不正确或有效负载不符合指令规范。 |
| INTERNAL_ERROR                  | 设备处理指令时发生错误，并且错误不属于指定的类别。     |