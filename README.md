# hangtiansuo_server

接口说明
--------

### `GET http://[host]:[port]/` ------ 申请一个新的会话

##### 所需参数
* `user_id`: 必需，用户的唯一ID。
* `expire_time`: 可选，最后一次回复之后的有效时间（单位：秒），超过这个时间后本次会话将被释放，默认600（十分钟）。
* `mode`: 可选，对应main.py中的`mode`参数（0：规则；1：模型；2：规则+模型），默认0。

##### 返回结果（若成功调用）
一个JSON对象，其中包含：
* `status`: 本次调用的结果（code）以及说明信息（message）。
* `session_id`: 此次会话分配到的UID (string类型)。之后使用POST方式交互均需要提供此UID作为参数。此次获取到的`session_id`将在对话结束或最后一次回复一定时间（可通过`expire_time`指定）后失效。

##### 成功调用示例
```
GET http://127.0.0.1:80/?user_id=123456&expire_time=600&mode=0
```
返回结果:
```json
{
    "session_id": "0b9f54ae-1d15-11e9-9a75-0f9226be1a37",
    "status": {
        "code": 200,
        "message": "Successfully created a session for user 123456 with expire_time 600 seconds"
    }
}
```

##### 失败调用可能原因
* 必需的参数`user_id`未指定
* 所传参数为空或格式不符合要求
* 其他服务器内部原因导致的失败


### `POST http://[host]:[port]/` ------ 与之前通过`GET http://[host]:[port]/`申请的会话进行交互

##### 所需参数
* `session_id`: 此前通过`GET http://[host]:[port]/`获取到的会话ID
* `user_input`: 用户输入的对话文本

##### 返回结果（若成功调用）
一个JSON对象，其中包含：
* `status`: 本次调用的结果（code）以及说明信息（message）。
* `output`: 系统返回的对于此次输入`user_input`的回复结果。
* `ended`: 系统返回的关于此轮对话是否结束的标记（`1`表示结束，`0`表示未结束）。**注意：在对话结束后再调用此API将会失败并得到一个错误提示。**
* `action_info`: **为测试/调试预留。** 一个JSON对象，包含字段`user_action`和`response_action`。其中`user_action`为对用户输入的意图识别结果（字符串）；`response_action`为回复意图（字符串）。
* `inner_error`: **为测试/调试预留。** 若成功调用，为一个空字符串；否则，当内部系统出现错误（抛出异常）时，会将异常的traceback信息放到此字段中。

##### 成功调用示例
```
POST http://127.0.0.1:80/
```
request.body:
```json
{
    "session_id": "0b9f54ae-1d15-11e9-9a75-0f9226be1a37",
    "user_input": "打印机坏了"
}
```
返回结果:
```json
{
    "output": "请问您的设备发生了什么故障",
    "ended": "0",
    "action_info": {
        "user_action": "{\"dia_act\": \"inform\", \"request_slots\": {}, \"inform_slots\": {\"goal\": \"printer\"}, \"nl\": \"打印机坏了\"}",
        "response_action": "{\"dia_act\": \"request\", \"request_slots\": {\"mafunction\": \"UNK\"}, \"inform_slots\": {}}"
    },
    "inner_error": "",
    "status": {
        "code": 200,
        "message": "Successfully produced the response"
    }
}
```

##### 失败调用可能原因
* 请求体JSON格式错误
* 必需的参数`session_id`或`user_input`未指定
* 所传参数为空或格式不符合要求
* `session_id`非法（未找到该`session_id`、`session_id`对应的会话已经结束或过期）
* 对话系统内部错误（抛出异常）
* 其他服务器内部原因导致的失败
