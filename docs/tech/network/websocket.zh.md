# Websocket 通信协议

## 写在前面

- 为保证通信的安全与隐私，除特殊标记（带```*```号或```**```号）的通信协议外，所有协议内容均经过```AES```对称密钥加密。其中```*```标记表示该协议无加密，```**```标记表示该协议通过```RSA```公钥加密。

- 所有的协议内容，在加密前（解密后）格式均为：

    ```json
    {
    	"command": "***",
    	"data": [data] // JSON或string或undefined
    }
    ```

- 本API文档之后的[协议名称]部分，均指明文协议中command字段需要的字符串，[参数]部分，表格的第一行均指data的类型，其后各行指明文协议中data部分需要的字段（如果data存在且类型为JSON）。

- 由于通信通常是客户端向服务器端发送数据，服务器端在接受到相应的数据后向客户端发送对应的返回数据，故本API文档大部分协议采取C2S-S2C的形式编写。其中C2S(Client2Server)表示客户端向服务器端发送，S2C类似。

- 协议内容在加密后，均为```base64```编码的字符串。

## 握手加密协议

### SetConnectionSymKey(C2S)*

#### 描述

用于和服务器交换AES密钥对，实现握手加密。

#### 参数

| 参数名称 | 类型   | 说明            |
| -------- | ------ | --------------- |
| data     | string | 1024位的RSA公钥 |

#### 示例

```json
{
	"command": "SetConnectionSymKey",
	"data": "MIGJAoGBAKMxsngBaWseSIR+84kzKaG31WEgePkC3AxgeOmQN3zHPs2tBHgsH8eqP8BTOVOzX6Llqo2HwO8FBMWetNRYMytzIYzSNL728ANyBeoItoyhqd2vlZYqj9eJ5ImmYcRMgTwc8CVcDuonjFF8bR2Y+sULLprrzFLjFSgj6K2Te3wZAgMBAAE="
}
```

### SetConnectionPubKeyResponse(S2C)**

#### 加密方法

使用[握手加密协议(C2S)*]提供的RSA公钥进行加密

#### 描述

用于设置客户端的AES对称密钥

#### 参数

| 参数名称 | 类型         | 说明                      |
| -------- | ------------ | ------------------------- |
| data     | JSON         | 无                        |
| state    | string       | 服务器端的响应状态        |
| pubKey   | string(可选) | 如果正确响应，则为AES密钥 |

| state状态     | 说明                    |      |
| ------------- | ----------------------- | ---- |
| NeedSetPubKey | 服务器端尚未获得RSA公钥 |      |
| PubKeyError   | 服务器端RSA公钥加密失败 |      |
| HasApproved   | 已经通过握手加密        |      |
| Success       | 服务器端签发AES密钥成功 |      |

#### 示例

```json
{
	"command": "SetConnectionPubKeyResponse",
	"data": {
		"state": "Success",
		"pubKey": "hfqtj+f9vPN3CXDPgN6qQJgIlQph2dvPe99fsZJUK2vud6Iwyw…0FyXp/blfQaE3Lf/n5CWFl+TppHN5x6q7oCGuUHGp10EZKMQ=" 
	}
}
```

```json
{
	"command": "SetConnectionPubKeyResponse",
    "data": {
        "state": "HasApproved"
    }
}
```

## 登录协议

### Login(C2S)

#### 描述

用于用户登录

#### 参数

| 参数名称  | 类型         | 说明                |
| --------- | ------------ | ------------------- |
| data      | JSON         | 无                  |
| email     | string       | 用户邮箱            |
| password  | string(可选) | 加盐hash后的密码    |
| emailCode | number(可选) | 邮箱验证码          |
| token     | string(可选) | 服务器端签发的token |

#### 示例

```json
{
    "command": "Login",
    "data": {
        "email": "rintaro@foxmail.com",
        "password": "fd750cd28387c49d2f5b951005e6ac2d5af2e7e0a3a3dd821827c12ae527f52d"
    }
}
```

```json
{
    "command": "Login",
    "data": {
        "email": "rintaro@foxmail.com",
        "emailCode": 2145621540
    }
}
```

### LoginResponse(S2C)

#### 描述

用于服务器端响应用户登录请求

#### 参数

| 参数名称 | 类型         | 说明                       |
| -------- | ------------ | -------------------------- |
| data     | JSON         | 无                         |
| state    | string       | 用户登录请求响应状态       |
| userId   | number(可选) | 成功登录后，为用户的用户Id |

| state状态      | 说明               |
| -------------- | ------------------ |
| Success        | 成功登录           |
| EmailInvalid   | 邮箱格式错误       |
| UserNotFound   | 数据库不存在该用户 |
| UserLogged     | 该用户已登录       |
| PasswordError  | 密码错误           |
| EmailCodeError | 验证码错误         |
| TokenError     | token错误          |
| TokenExpired   | token过期          |

#### 示例

```json
{
    "command": "LoginResponse",
    "data": {
        "state": "Success",
        "userId": 1
    }
}
```

```json
{
	"command": "LoginResponse",
    "data": {
        "state": "UserLogged"
    }
}
```

## 注册协议

### Register(C2S)

#### 描述

用于客户端向服务器端发送注册账户请求

#### 参数

| 参数名称  | 类型   | 说明             |
| --------- | ------ | ---------------- |
| data      | JSON   | 无               |
| userName  | string | 用户名           |
| password  | string | 加盐hash后的密码 |
| email     | string | 用户邮箱         |
| emailCode | number | 邮箱验证码       |

#### 示例

```json
{
    "command": "Register",
    "data": {
        "userName": "ARintaro",
        "email": "rintaro@foxmail.com",
        "password": "fd750cd28387c49d2f5b951005e6ac2d5af2e7e0a3a3dd821827c12ae527f52d",
        "emailCode": 1234567890
    }
}
```

### RegisterResponse(S2C)

#### 描述

用于服务器端向客户端响应注册请求

#### 参数

| 参数名称 | 类型         | 说明                           |
| -------- | ------------ | ------------------------------ |
| data     | JSON         | 无                             |
| state    | string       | 注册请求的响应状态             |
| userId   | number(可选) | 注册成功后，为注册用户的用户ID |

| state状态           | 说明           |
| ------------------- | -------------- |
| Success             | 注册成功       |
| UserNameFormatError | 用户名格式错误 |
| PasswordFormatError | 密码格式错误   |
| EmailRegistered     | 该邮箱已被注册 |
| EmailCodeError      | 验证码错误     |
| EmailInvalid        | 验证码格式错误 |

#### 示例

```json
{
    "command": "RegisterResponse",
    "data": {
		"state": "Success",
        "userId": 15
    }
}
```

```json
{
    "command": "RegisterResponse",
    "data": {
		"state": "EmailCodeError"
    }
}
```

## 初始数据拉取协议

### Pull(C2S)

#### 描述

在登陆成功后，客户端自动向服务器端发送，请求拉取用于初始化聊天信息的数据

#### 参数

| 参数名称        | 类型   | 说明                                                  |
| --------------- | ------ | ----------------------------------------------------- |
| data            | JSON   | 无                                                    |
| lastRequestId   | number | 增量拉取最新的RequestId后的Request，为0时进行全量拉取 |
| noticeTimestamp | number | 增量拉取该时间戳后的全部notice                        |

#### 示例

```json
{
    "command": "Pull",
    "data": {
        "lastRequestId": 0,
        "noticeTimestamp": 1685018341911
    }
}
```

### ReadCursors(S2C)

#### 描述

服务器端向客户端发送对应聊天最新已读消息的位置

#### 参数

| 参数名称    | 类型  | 说明                                     |
| ----------- | ----- | ---------------------------------------- |
| data        | Array | 无                                       |
| data[index] | Array | 以元组的形式存储的[chatId, readCursor]对 |

#### 示例

```json
{
    "command": "ReadCursors",
    "data": [
        [1, 69],
        [3, 52],
        [8, 5],
        [2, 11]
    ]
}
```

### Messages(S2C)

#### 描述

服务器端向客户端为每个聊天发送最新的部分消息

#### 参数

| 参数名称    | 类型   | 说明               |
| ----------- | ------ | ------------------ |
| data        | Array  | 无                 |
| data[index] | string | 序列化后的消息数据 |

#### 示例

```json
{
    "command": "Messages",
    "data": [
        "{"type":"Text", "inChatId":64, "chatId":1, "senderId":7, "serializedContent":"\"\\\"XXX\\\"\"", "timestamp":1684332121757}",
		"{"type":"Text", "inChatId":65, "chatId":1, "senderId":3, "serializedContent":"\"www.hao.com www.hao.com www.hao.com\"", "timestamp":1684332445808}"
    ]
}
```

### Requests(S2C)

#### 描述

服务器端向客户端为每个聊天发送最新的部分消息

#### 参数

| 参数名称    | 类型   | 说明                  |
| ----------- | ------ | --------------------- |
| data        | Array  | 无                    |
| data[index] | string | 序列化后的Request数据 |

#### 示例

```json
{
    "command": "Requests",
    "data": [
        "{"info":{"reqId":1,"senderId":2,"message":"123","content":{"type":"JoinGroup","chatId":1}},"state":"Approved"}",
        "{"info":{"reqId":2,"senderId":1,"message":"test","content":{"type":"MakeFriend","receiverId":2}},"state":"Approved"}"
    ]
}
```

### UserSetting(S2C)

#### 描述

服务器端向客户端发送用户配置数据

#### 参数

| 参数名称 | 类型   | 说明                   |
| -------- | ------ | ---------------------- |
| data     | string | 序列化后的用户配置数据 |

#### 示例

```json
{
    "command": "UserSetting",
    "data": "{"secondaryCheckChats":[],"userNickname":[],"notification":{"slient":true,"show":true,"muteChatIds":[6]},"topChatIds":[4,3]}"
}
```

### Notices(S2C)

#### 描述

服务器端向客户端发送客户端拉取的部分Notice数据列表

#### 参数

| 参数名称    | 类型   | 说明                 |
| ----------- | ------ | -------------------- |
| data        | Array  | 无                   |
| data[index] | string | 序列化后的Notice数据 |

#### 示例

```json
{
    "command": "Notices",
    "data": []
}
```

### PullResponse(S2C)

#### 描述

服务器端向客户端响应Pull请求状态

#### 参数

| 参数名称 | 类型   | 说明               |
| -------- | ------ | ------------------ |
| data     | JSON   | 无                 |
| state    | string | Pull请求的响应状态 |

| state状态     | 说明             |
| ------------- | ---------------- |
| Success       | 成功响应Pull请求 |
| DatabaseError | 数据库异常       |

#### 示例

```json
{
    "command": "PullResponse",
    "data": {
        "state": "Success"
    }
}
```

## 用户信息拉取协议

### GetUserInfo(C2S)

#### 描述

客户端向服务器端拉取用户信息

#### 参数

| 参数名称 | 类型   | 说明   |
| -------- | ------ | ------ |
| data     | number | 用户ID |

#### 示例

```json
{
    "command": "GetUserInfo",
    "data": 1
}
```

