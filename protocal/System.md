# System接口

System接口提供多个系统级事件、指令。

## 1 SynchronizeState 事件

建立新连接时，必须发送SynchronizeState事件到腾讯云小微以更新所有终端状态。

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

## 2 UserInactivityReport 事件

此事件必须在一小时不活动后发送，并在此之后每小时发送一次，直到执行用户操作。 腾讯云小微以此检测上次用户活动以来的持续时间。 用户活动被定义为确认用户存在产品的动作，例如与产品上的按钮交互，与腾讯云小微交谈或使用GUI 检测到用户活动后，用于跟踪不活动的计时器必须重置为0。

**提示**：为inactiveTimeInSeconds提供的值应始终为3600（1小时）的倍数。 例如，在4小时不活动后，该值将为14400。

**代码示例**
```java
{
   "event": {
        "header": {
            "namespace": "System",
            "name": "UserInactivityReport",
            "messageId": "{{STRING}}"
        },
        "payload": {
            "inactiveTimeInSeconds": {{LONG}}
        }

    }

}
```


**Header参数**

| 参数      | 描述                       | 类型   |
| --------- | -------------------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |



**有效负载参数**

| 参数                  | 描述                       | 类型 |
| --------------------- | -------------------------- | ---- |
| inactiveTimeInSeconds | 自上次用户交互以来的秒数。 | long |

## 3 ResetUserInactivity 指令


ResetUserInactivity指令发送客户端可以重置UserInactivityReport使用的不活动计时器。 例如，腾讯DingDang应用程序上的用户交互将触发此指令。
**代码示例**
```java
{
    "directive": {
        "header": {
            "namespace": "System",
            "name": "ResetUserInactivity",
            "messageId": "{{STRING}}"
        },
        "payload": {
        }
    }
}
```


**Header参数**

| 参数      | 描述                       | 类型   |
| --------- | -------------------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |

## 4 SetEndpoint 指令

SetEndpoint指令指示客户端在满足以下条件时更改接入点：
- 客户端连接到的接入点不支持用户的国家/地区设置。 例如，如果用户在管理您的内容和设备中把当前国家/地区设置为的英国（UK），但客户端连接到美国（US）接入点，腾讯云小微将发送SetEndpoint指令，指示客户端连接到支持英国的接入点。
- 用户更改其国家/地区设置（或地址）。 例如，如果连接到US接入点的用户将其当前国家/地区从美国更改为英国，腾讯云小微将发送SetEndpoint指令，指示客户端连接到支持UK的端点。
  **重要**：未能切换接入点可能导致用户无法访问自定义首选项以及国家或地区特定内容。
  **代码示例**
```java
{
    "directive": {
        "header": {
            "namespace": "System",
            "name": "SetEndpoint",
            "messageId": "{{STRING}}"
        },
        "payload": {
            "endpoint": "{{STRING}}"
         }
    }
}
```


**Header参数**

| 参数      | 描述                       | 类型   |
| --------- | -------------------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |



**有效负载参数**

| 参数     | 描述                                                         | 类型   |
| -------- | ------------------------------------------------------------ | ------ |
| endpoint | 支持用户的国家/地区设置的腾讯云小微接入点URL，URL可能包括协议和端口。例如： https://tvs.html5.qq.com | string |



## 5 SoftwareInfo 事件

此事件将您产品的软件信息传达给腾讯云小微，例如固件版本。 它必须在以下场景中发送：
- 对于具有持久存储的产品，必须在产品的初始引导和固件版本更新时发送事件。
- 对于没有持久存储的产品，必须在每次启动/重新启动时发送事件。
- 收到ReportSoftwareInfo指令时。
  如果事件处理成功，产品将收到204 HTTP状态码，返回体为空。 如果未处理该事件，则产品将收到500 HTTP状态码和异常消息。

**代码示例**
```java
{    
    "event": {
        "header": {
            "namespace": "System",
            "name": "SoftwareInfo",
            "messageId": "{{STRING}}"
        },
        "payload": {
            "firmwareVersion": "{{STRING}}"
        }
    }
}
```

**Header Parameter**

| 参数      | 描述                       | 类型   |
| --------- | -------------------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |



**有效负载参数**

| 参数            | 描述                                                         | 类型   |
| --------------- | ------------------------------------------------------------ | ------ |
| firmwareVersion | 32位正整数的字符串形式。 如果向腾讯云小微发送了无效值，腾讯云小微会向您的客户端返回HTTP 400状态代码。 重要信息："0"不是有效的固件版本。 | string |

| 有效       | 无效                    |
| ---------- | ----------------------- |
| "123"      | "50.3"                  |
| "8701"     | "tvs-123.4x"            |
| "20170207" | "tsk.201-(1.23.4-test)" |

## 6 ReportSoftwareInfo 指令

该指令收到时，产品必须使用SoftwareInfo事件向腾讯云小微报告当前软件信息。

**代码示例**
```java
{
    "directive": {
        "header": {
            "namespace": "System",
            "name": "ReportSoftwareInfo",
            "messageId": "{{STRING}}"
        },
        "payload": {
        }
    }
}
```

**Header Parameter**

| 参数      | 描述                       | 类型   |
| --------- | -------------------------- | ------ |
| messageId | 用于表示特定消息的唯一ID。 | string |

## 7 ExceptionEncountered 事件

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