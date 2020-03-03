# TvsCustomData接口

TvsCustomData接口用于传递数据

 

## 1 Data指令

用于将云小微特有的数据下发给终端解析、执行

**示例**

```java
{
    "header": {
        "namespace": "TvsCustomData",
        "name": "Data",
        "messageId":"{{STRING}}",
        "dialogRequestId":"{{STRING}}"
    },
    "payload": {
        "data": [
        {
            "type":"{{STRING}}",
            "value":"{{STRING}}"
        }
        ],
    }
}
```
**header参数**

| 参数            | 描述                                                         | 类型   |
| --------------- | ------------------------------------------------------------ | ------ |
| messageId       | 用于表示本指令的唯一ID。                                   | string |
| dialogRequestId | 用于将response中的指令与特定的Recognize事件关联起来的唯一ID。 | string |

**payload参数**

| 参数                 | 描述                                                         | 类型   |
| -------------------- | ------------------------------------------------------------ | ------ |
| data                | 数据集合 | list |
| data.type | 数据标识，现有的数据标识见1.1 data.type | string |
| data.value       | 数据的值，字符串封装，但内部的数据格式根据data.type的不同而不同 | string |

 

### 1.1 data.type

data.type根据功能进行划分。可以分为"情绪系统相关"等。

#### 1.1.1 情绪系统相关数据

#### 1.1.1.1 表情数据



`type`: emotion



`value`: value格式是一个json的字符串形式

```
{
    "description": "{{STRING}}",
    "tag": "{{STRING}}"
}

```

字段含义为

| 字段名称            | 字段含义 | 字段类型 |
| ------------------- | -------- | -------- |
| tag         | 情绪标签 | string   |
| description | 情绪描述 | string   |



**字段取值说明：**

tag和description的取值成对出现，具体如下：

| tag      | description |
| -------- | ----------- |
| like     | 喜爱        |
| surprise | 惊喜        |
| neutral  | 中性        |
| sad      | 哀          |
| fear     | 惧          |
| angry    | 怒          |

**使用时请以tag标签字段为准**，其他字段作为描述和补充信息，不保证取值有效性，不建议使用来做逻辑。



表情数据示例

```
{
	"header": {
		"namespace": "TvsCustomData",
		"name": "Data",
		"messageId": "{{STRING}}",
		"dialogRequestId": "{{STRING}}"
	},
	"payload": {
		"data": [{
			"type": "emotion",
			"value": "{\"description\": \"喜欢\",\"tag\": \"like\"}"
		}]
	}
}
```





#### 1.1.1.2 动作数据



`type`: action



`value`: value格式是一个json的字符串形式

```
{
    "description": "{{STRING}}",
    "tag": "{{STRING}}"
}

```

字段含义为

| 字段名称            | 字段含义 | 字段类型 |
| ------------------- | -------- | -------- |
| tag         | 动作标签 | string   |
| description | 动作描述 | string   |



**字段取值说明：**

tag和description的取值成对出现，具体如下：


| tag         |description |
| ------------------ | ------------------ |
| welcome            | 欢迎               |
| show_love          | 示爱               |
| shy_state          | 害羞态             |
| hands_on_hips      | 不满               |
| sad_face           | 沮丧态             |
| compliment_state   | 称赞               |
| longing            | 期待               |
| doubt_chin         | 质疑态             |
| speaking_circle    | 循环播报           |
| speaking_explain   | 讲解播报           |
| speaking_introduce | 介绍播报           |
| speaking_point     | 指点播报           |

**使用时请以tag标签字段为准**，其他字段作为描述和补充信息，不保证取值有效性，不建议使用来做逻辑。



动作数据示例

```
{
	"header": {
		"namespace": "TvsCustomData",
		"name": "Data",
		"messageId": "{{STRING}}",
		"dialogRequestId": "{{STRING}}"
	},
	"payload": {
		"data": [{
			"type": "action",
			"value": "{\"description\": \"欢迎\",\"tag\": \"welcome\"}"
		}]
	}
}
```



#### 1.1.1.3 终端如何根据数据做逻辑

情绪的动作数据与表情数据经常同时下发。

当终端收到动作数据或者表情数据时，可以根据动作表情和表情标签自定定义终端处理行为。

例如对于有屏设备，当表情数据tag为angry（愤怒）时，屏幕上可以展示一个愤怒的表情。

对于机器人设备，当收到的动作tag为welcome（欢迎）时，机器人可以做一个欢迎的动作。

