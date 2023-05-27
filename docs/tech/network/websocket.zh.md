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

### GetUserInfoResponse(S2C)

#### 描述

服务器端向客户端发送用户信息数据

#### 参数

| 参数名称   | 类型         | 说明                         |
| ---------- | ------------ | ---------------------------- |
| data       | JSON         | 无                           |
| state      | string       | 响应状态                     |
| userId     | number(可选) | 成功响应后，为用户Id         |
| userName   | string(可选) | 成功响应后，为用户名         |
| avaterHash | string(可选) | 成功响应后，为头像文件hash值 |

| state状态    | 说明         |
| ------------ | ------------ |
| Success      | 正确响应请求 |
| UserNotFound | 该用户不存在 |
| ServerError  | 服务器错误   |

#### 示例

```json
{
    "command": "GetUserInfoResponse",
    "data": {
		"state": "Success",
        "userId": 8,
        "userName": "Rintaro",
        "avaterHash": "b9c9a45da34b134a835c0ec673ea4ce4"
    }
}
```

## 聊天数据拉取协议

### GetChatInfo(C2S)

#### 描述

客户端向服务器端申请聊天信息数据

#### 参数

| 参数名称 | 类型   | 说明   |
| -------- | ------ | ------ |
| data     | number | 聊天ID |

#### 示例

```json
{
    "command": "GetChatInfo",
    "data": 9
}
```

### Chat(S2C)

#### 描述

服务器端向客户端发送聊天信息数据

#### 参数

| 参数名称 | 类型   | 说明                   |
| -------- | ------ | ---------------------- |
| data     | string | 序列化后的聊天信息数据 |

#### 示例

```json
{
    "command": "Chat",
    "data": "{\"id\":9,\"name\":\"创建群聊测试2\",\"avaterHash\":\"\"}"
}
```

## 群聊权限相关协议

### GetGroupUsers、GetGroupAdmin、GetGroupOwner(C2S)

#### 描述

客户端向服务器端请求群聊用户ID列表、请求管理员ID列表、请求群主ID

#### 参数

| 参数名称 | 类型   | 说明   |
| -------- | ------ | ------ |
| data     | number | 聊天ID |

#### 示例

```json
{
    "command": "GetGroupUsers",
    "data": 9
}
```

```json
{
    "command": "GetGroupAdmin",
    "data": 9
}
```

```json
{
    "command": "GetGroupOwner",
    "data": 9
}
```

### GetGroupUsersResponse(S2C)

#### 描述

服务器端向客户端发送群聊成员ID列表

#### 参数

| 参数名称       | 类型         | 说明           |
| -------------- | ------------ | -------------- |
| data           | JSON         | 无             |
| state          | string       | 响应状态       |
| chatId         | number(可选) | 聊天ID         |
| userIds        | Array(可选)  | 群聊用户ID列表 |
| userIds[index] | number       | 用户ID         |

| state状态    | 说明           |
| ------------ | -------------- |
| Success      | 成功响应       |
| NotGroupChat | 该聊天不是群聊 |
| ServerError  | 服务器异常     |

#### 示例	

```json
{
    "command": "GetGroupUsersResponse",
    "data": {
        "state": "Success",
        "chatId": 8,
        "userIds": [
            1,
            5
        ]
    }
}
```

```json
{
    "command": "GetGroupUsersResponse",
    "data": {
        "state": "NotGroupChat"
    }
}
```

### GetGroupOwnerResponse(S2C)

#### 描述

服务器端向客户端发送群主ID

#### 参数

| 参数名称 | 类型         | 说明                 |
| -------- | ------------ | -------------------- |
| data     | JSON         | 无                   |
| chatId   | number(可选) | 成功响应后，为聊天ID |
| userId   | number(可选) | 成功响应后，为群主ID |

| state状态     | 说明                     |
| ------------- | ------------------------ |
| Success       | 成功响应                 |
| UserNotInChat | 发出请求的用户不在聊天中 |
| ServerError   | 服务器错误               |

#### 示例

```json
{
    "command": "GetGroupOwnerResponse",
    "data": {
        "state": "Success",
        "chatId": 8,
        "userId": 10
    }
}
```

```json
{
    "command": "GetGroupOwnerResponse",
    "data": {
        "state": "UserNotInChat"
    }
}
```

### GetGroupAdminResponse(S2C)

#### 描述

服务器端向客户端发送管理员ID列表

#### 参数

| 参数名称       | 类型         | 说明                       |
| -------------- | ------------ | -------------------------- |
| data           | JSON         | 无                         |
| chatId         | number(可选) | 成功响应后，为聊天ID       |
| userIds        | Array(可选)  | 成功响应后，为管理员ID列表 |
| userIds[index] | number       | 管理员ID                   |

| state状态     | 说明                     |
| ------------- | ------------------------ |
| Success       | 成功响应                 |
| UserNotInChat | 发出请求的用户不在聊天中 |
| ServerError   | 服务器错误               |

#### 示例

```json
{
    "command": "GetGroupAdminResponse",
    "data": {
        "state": "Success",
        "chatId": 8,
        "userIds": [
            10,
            5
        ]
    }
}
```

```json
{
    "command": "GetGroupAdminResponse",
    "data": {
        "state": "UserNotInChat"
    }
}
```

### GroupOwnerTransfer(C2S)

#### 描述

转移群主身份

#### 参数

| 参数名称 | 类型   | 说明           |
| -------- | ------ | -------------- |
| data     | JSON   | 无             |
| chatId   | number | 聊天ID         |
| userId   | number | 要移交的用户ID |

#### 示例

```json
{
    "command": "GroupOwnerTransfer",
    "data": {
        "chatId": 5,
        "userId": 2
    }
}
```

### GroupOwnerTransferResponse(S2C)

#### 描述

服务器端向客户端响应移交群主请求

#### 参数

| 参数名称 | 类型   | 说明     |
| -------- | ------ | -------- |
| data     | JSON   | 无       |
| state    | string | 响应状态 |
| chatId   | number | 聊天ID   |
| userId   | number | 用户ID   |

| state状态     | 说明             |
| ------------- | ---------------- |
| Success       | 成功响应         |
| NotOwner      | 发出请求的用户ID |
| UserNotInChat | 用户不在群聊中   |
| ServerError   | 服务器错误       |

#### 示例

```json
{
    "command": "GroupOwnerTransferResponse",
    "data": {
        "state": "Success",
        "chatId": 4,
        "userId": 2
    }
}
```

### RemoveGroupMember(C2S)

#### 描述

踢出群成员

#### 参数

| 参数名称 | 类型   | 说明   |
| -------- | ------ | ------ |
| chatId   | number | 聊天ID |
| userId   | number | 用户ID |

#### 示例

```json
{
    "command": "RemoveGroupMember",
    "data": {
		"chatId": 5,
        "userId": 2
    }
}
```

### RemoveGroupMemberResponse(S2C)

#### 描述

服务器端向客户端响应踢出群成员请求

#### 参数

| 参数名称 | 类型         | 说明                         |
| -------- | ------------ | ---------------------------- |
| data     | JSON         | 无                           |
| state    | string       | 响应状态                     |
| chatId   | number(可选) | 成功响应后，为聊天ID         |
| userId   | number(可选) | 成功响应后，为被踢出的用户ID |

| state状态     | 说明                       |
| ------------- | -------------------------- |
| Success       | 成功响应                   |
| SameUser      | 发出请求用户与踢出用户相同 |
| NoPermission  | 无权限踢出该成员           |
| UserNotInChat | 用户不在群聊中             |
| ServerError   | 服务器错误                 |

#### 示例

```json
{
    "command": "RemoveGroupMemberResponse",
    "data": {
        "state": "Success",
        "chatId": 5,
        "userId": 15
    }
}
```

## 私聊已读查询协议

### GetUserReadInPrivate(C2S)

#### 描述

客户端向服务器端请求私聊对方已读到的最新消息ID

#### 参数

| 参数名称 | 类型   | 说明   |
| -------- | ------ | ------ |
| data     | number | 聊天ID |

#### 示例iLike

```json
{
    "command": "GetUserReadInPrivate",
    "data": 72
}
```

### GetUserReadInPrivateResponse(S2C)

#### 描述

服务器端向客户端发送私聊对方最新的已读消息ID

#### 参数

| 参数名称 | 类型         | 说明               |
| -------- | ------------ | ------------------ |
| data     | JSON         | 无                 |
| state    | string       | 响应状态           |
| chatId   | number(可选) | 成功响应后，聊天ID |
| inChatId | number(可选) | 成功响应后，消息ID |

| state状态      | 说明                       |
| -------------- | -------------------------- |
| Success        | 成功响应                   |
| NotPrivateChat | 该聊天不是私聊             |
| UserNotInChat  | 发出请求的用户不在该群聊中 |
| ServerError    | 服务器错误                 |

#### 示例

```json
{
    "command": "GetUserReadInPrivateResponse",
    "data": {
        "state": "Success",
        "chatId": 72,
        "inChatId": 5
    }
}
```

```json
{
    "command": "GetUserReadInPrivateResponse",
    "data": {
        "state": "NotPrivateChat"
    }
}
```

## 群聊已读查询协议

### GetUserReadInGroup(C2S)

#### 描述

客户端向服务器端请求已读该消息的群成员ID列表

#### 参数

| 参数名称 | 类型   | 说明   |
| -------- | ------ | ------ |
| data     | JSON   | 无     |
| chatId   | number | 群聊ID |
| inChatId | number | 消息ID |

#### 示例

```json
{
    "command": "GetUserReadInGroup",
    "data": {
		"chatId": 4,
        "inChatId": 49
    }
}
```

### GetUserReadInGroupResponse(S2C)

#### 描述

服务器端向客户端发送对应消息的已读成员ID列表

#### 参数

| 参数名称       | 类型         | 说明                       |
| -------------- | ------------ | -------------------------- |
| data           | JSON         | 无                         |
| state          | string       | 响应状态                   |
| chatId         | number(可选) | 成功响应后，聊天ID         |
| inChatId       | number(可选) | 成功响应后，消息ID         |
| userIds        | Array(可选)  | 成功响应后，已读成员ID列表 |
| userIds[index] | number       | 用户ID                     |

| state状态     | 说明                     |
| ------------- | ------------------------ |
| Success       | 成功响应                 |
| NotGroupChat  | 该群聊不是群聊           |
| UserNotInChat | 发出请求的用户不在群聊中 |

#### 示例

```json
{
    "command": "GetUserReadInGroupResponse",
    "data": {
        "state": "Success",
        "chatId": 4,
        "inChatId": 15,
        "userIds": [
            10,
            2
        ]
    }
}
```

```json
{
    "command": "GetUserReadInGroupResponse",
    "data": {
        "state": "UserNotInChat"
    }
}
```

## 群聊公告协议

### PullGroupNotice(C2S)

#### 描述

客户端向服务器端增量拉取群聊公告数据

#### 参数

| 参数名称     | 类型   | 说明                                    |
| ------------ | ------ | --------------------------------------- |
| data         | JSON   | 无                                      |
| chatId       | number | 聊天ID                                  |
| lastNoticeId | number | 客户端已有的最新公告ID，若为0则全量拉取 |

#### 示例

```json
{
    "command": "PullGroupNotice",
    "data": {
		"chatId": 4,
        "lastNoticeId": 0
    }
}
```

### PullGroupNoticeResponse(S2C)

#### 描述

服务器端向客户端发送序列化后的群公告数据列表

#### 参数

| 参数名称           | 类型        | 说明                         |
| ------------------ | ----------- | ---------------------------- |
| data               | JSON        | 无                           |
| state              | string      | 响应状态                     |
| chatId             | number      | 聊天ID                       |
| groupNotice        | Array(可选) | 成功响应后，为群公告数据列表 |
| groupNotice[index] | string      | 序列化后的群公告数据         |

| state状态     | 说明                       |
| ------------- | -------------------------- |
| Success       | 成功响应                   |
| UserNotInChat | 发出请求的用户不在该群聊中 |
| ServerError   | 服务器错误                 |

#### 示例

```json
{
    "command": "PullGroupNoticeResponse",
    "data": {
		"state": "Success",
        "chatId": 4,
        "groupNotice": []
    }
}
```

### SendGroupNotice(C2S)

#### 描述

发送群公告

#### 参数

| 参数名称 | 类型   | 说明         |
| -------- | ------ | ------------ |
| data     | JSON   | 无           |
| chatId   | number | 聊天ID       |
| clientId | number | 客户端消息ID |
| notice   | string | 公告内容     |

#### 示例

```json
{
    "command": "SendGroupNotice",
    "data": {
        "chatId": 4,
        "clientId": 1,
        "notice": "test"
    }
}
```

### GroupNoticeResponse(S2C)

#### 描述

服务器端响应客户端的发送群公告请求

#### 参数

| 参数名称  | 类型   | 说明         |
| --------- | ------ | ------------ |
| data      | JSON   | 无           |
| state     | string | 响应状态     |
| chatId    | number | 聊天ID       |
| clientId  | number | 客户端聊天ID |
| noticeId  | number | 公告ID       |
| timestamp | number | 公告时间戳   |

| state状态          | 说明           |
| ------------------ | -------------- |
| Success            | 成功响应       |
| NoPermission       | 无权发送群公告 |
| LenthLimitExceeded | 群公告长度过长 |
| ServerError        | 服务器错误     |

#### 示例

```json
{
    "command": "GroupNoticeResponse",
    "data": {
		"state": "Success",
        "chatId": 4,
        "clientId": 1,
        "noticeId": 1,
        "timestamp": 16851863077
    }
}
```

## token签发协议

### ApplyForToken(C2S)

#### 描述

客户端向服务器端请求签发token

#### 参数

| 参数列表 | 类型       | 说明 |
| -------- | ---------- | ---- |
| data     | undefinend | 无   |

#### 示例

```json
{
	"command": "ApplyForToken"
}
```

### ApplyForTokenResponse(S2C)

#### 描述

服务器端向客户端签发token

#### 参数

| 参数名称  | 类型         | 说明     |
| --------- | ------------ | -------- |
| data      | JSON         | 无       |
| state     | string       | 响应状态 |
| token     | string(可选) | token    |
| timestamp | number(可选) | 时间戳   |

| state状态   | 说明       |
| ----------- | ---------- |
| Success     | 成功响应   |
| ServerError | 服务器错误 |

#### 示例

```json
{
    "command": "ApplyForTokenResponse",
    "data": {
        "state": "ServerError"
    }
}
```

```json
{
    "command": "ApplyForTokenResponse",
    "data": {
        "state": "Success",
        "token": "b309358883384c45b321340a127cacd6",
        "timestamp": 1685192367997
    }
}
```

## 消息发送及操作协议

### SendMessage(C2S)

#### 描述

发送消息

#### 参数

| 参数名称          | 类型   | 说明               |
| ----------------- | ------ | ------------------ |
| data              | JSON   | 无                 |
| type              | string | 消息类型           |
| clientId          | number | 客户端消息ID       |
| serializedContent | string | 序列化后的消息主体 |
| timestamp         | number | 时间戳             |

| type类型    | 说明     |
| ----------- | -------- |
| Text        | 文本消息 |
| File        | 文件消息 |
| Image       | 图片消息 |
| Voice       | 语音消息 |
| Transfer    | 转发消息 |
| Revoked     | 撤回消息 |
| MentionText | 提及消息 |
| ReplyText   | 回复消息 |

#### 示例

```json
{
    "command": "SendMessage",
    "data": {
		"chatId": 4,
        "clientId": 1,
        "serializedContent": "\"你好\"",
        "timestamp": 1685192622888,
        "type": "Text"
    }
}
```

### SendMessageResponse(S2C)

#### 描述

服务器端响应客户端发送消息请求

#### 参数

| 参数名称  | 类型   | 说明         |
| --------- | ------ | ------------ |
| data      | JSON   | 无           |
| state     | string | 响应状态     |
| clientId  | number | 客户端消息ID |
| chatId    | number | 聊天ID       |
| inChatId  | number | 消息ID       |
| timestamp | number | 时间戳       |

| state状态          | 说明           |
| ------------------ | -------------- |
| Success            | 成功响应       |
| LenthLimitExceeded | 发送消息过长   |
| UserNotInChat      | 用户不在聊天中 |
| UserNotLoggedIn    | 用户尚未登录   |
| ServerError        | 服务器错误     |

#### 示例

```json
{
    "command": "SendMessageResponse",
    "data": {
		"state": "Success",
        "clientId": 1,
        "chatId": 4,
        "inChatId": 52,
        "timestamp": 1685192616
    }
}
```

```json
{
    "command": "SendMessageResponse",
    "data": {
		"state": "UserNotInChat"
    }
}
```

### RevokeMessage(C2S)

#### 描述

撤回消息

#### 参数

| 参数名称 | 类型   | 说明     |
| -------- | ------ | -------- |
| data     | JSON   | 无       |
| chatId   | number | 聊天ID   |
| inChatId | number | 消息ID   |
| method   | string | 撤回权限 |

| method类型 | 说明                     |
| ---------- | ------------------------ |
| Sender     | 发送者撤回自己发送的消息 |
| GroupAdmin | 群聊管理员撤回消息       |
| GroupOwner | 群主撤回消息             |

#### 示例

```json
{
    "command": "RevokeMessage",
    "data": {
		"chatId": 4,
        "inChatId": 52,
        "method": "Sender"
    }
}
```

### RevokeMessageResponse(S2C)

#### 描述

服务器端响应撤回请求

#### 参数

| 参数名称 | 类型   | 说明     |
| -------- | ------ | -------- |
| data     | JSON   | 无       |
| state    | string | 响应状态 |
| chatId   | number | 聊天ID   |
| inChatId | number | 消息ID   |

| state状态         | 说明           |
| ----------------- | -------------- |
| Success           | 成功响应       |
| TimeLimitExceeded | 超时无法撤回   |
| PermissionsDenied | 无权撤回该消息 |
| MessageNotExisted | 消息不存在     |

#### 示例

```json
{
    "command": "RevokeMessageResponse",
    "data": {
		"state": "Success",
        "chatId": 4,
        "inChatId": 50
    }
}
```

## 请求发送及处理协议

### SendRequest(C2S)

#### 描述

发送诸如添加好友，申请入群的请求

#### 参数

| 参数名称           | 类型         | 说明             |
| ------------------ | ------------ | ---------------- |
| data               | JSON         | 无               |
| clientId           | number       | 客户端消息ID     |
| content            | JSON         | 请求体           |
| content.type       | string       | 请求类型         |
| content.receiverId | number(可选) | 请求接收者ID     |
| content.chatId     | number(可选) | 接收请求的群聊ID |
| message            | string       | 请求验证消息     |

| content.type类型 | 说明         |
| ---------------- | ------------ |
| MakeFriend       | 添加好友     |
| JoinGroup        | 申请入群     |
| GroupInvitation  | 邀请入群     |
| InvitedJoinGroup | 邀请入群审核 |

#### 示例

```json
{
    "command": "SendRequest",
    "data": {
        "messgae": "加个好友",
        "clientId": 1,
        "content": {
			"type": "MakeFriend",
            "receiverId": 15
        }
    }
}
```

### SendRequestResponse(S2C)

#### 描述

服务器端响应发送请求的请求

#### 参数

| 参数名称 | 类型         | 说明                 |
| -------- | ------------ | -------------------- |
| data     | JSON         | 无                   |
| state    | string       | 响应状态             |
| clientId | number       | 客户端消息ID         |
| reqId    | number(可选) | 成功响应后，为请求ID |

| state状态   | 说明       |
| ----------- | ---------- |
| Success     | 成功响应   |
| ServerError | 服务器错误 |

#### 示例

```json
{
    "command": "SendRequestResponse",
    "data": {
        "state": "Success",
        "clientId": 1,
        "reqId": 15
    }
}
```

### SolveRequest(C2S)

#### 描述

处理请求

#### 参数

| 参数名称 | 类型   | 说明     |
| -------- | ------ | -------- |
| data     | JSON   | 无       |
| reqId    | number | 请求ID   |
| answer   | string | 处理操作 |

| answer类型 | 说明     |
| ---------- | -------- |
| Refused    | 拒绝请求 |
| Approved   | 同样请求 |

#### 示例

```json
{
    "command": "SolveRequest",
    "data": {
		"reqId": 15,
        "answer": "Refused"
    }
}
```

### SolveRequestResponse(S2C)

#### 描述

服务器端响应用户处理请求

#### 参数

| 参数名称 | 类型   | 说明     |
| -------- | ------ | -------- |
| data     | JSON   | 无       |
| state    | string | 响应状态 |
| reqId    | number | 请求ID   |

| state状态 | 说明             |
| --------- | ---------------- |
| Success   | 成功响应         |
| Nohandler | 该用户无处理权限 |

#### 示例

```json
{
    "command": "SolveRequestResponse",
    "data": {
        "state": "Success",
        "reqId": 10
    }
}
```

### RequestStateUpdate(S2C)

#### 描述

服务器端向客户端发送请求状态更改指令

#### 参数

| 参数名称 | 类型   | 说明         |
| -------- | ------ | ------------ |
| data     | JSON   | 无           |
| state    | string | 将更新的状态 |
| reqId    | number | 请求ID       |

#### 示例

```json
{
    "command": "RequestStateUpdate",
    "data": {
		"state": "Solved",
        "reqId": 10
    }
}
```

## 删除好友协议

### Unfriend(C2S)

#### 描述

删除好友

#### 参数

| 参数名称 | 类型   | 说明         |
| -------- | ------ | ------------ |
| data     | number | 删除的好友ID |

#### 示例

```json
{
    "command": "Unfriend",
    "data": 15
}
```

### UnfriendResponse(S2C)

#### 描述

服务器端响应客户端删除好友请求

#### 参数

| 参数名称 | 类型         | 说明                 |
| -------- | ------------ | -------------------- |
| state    | string       | 响应状态             |
| chatId   | number(可选) | 成功响应后，为聊天ID |

| state状态 | 说明             |
| --------- | ---------------- |
| Success   | 成功响应         |
| NotFriend | 二者目前不是好友 |

#### 示例

```json
{
    "command": "UnfriendResponse",
    "data": {
        "state": "Success",
        "chatId": 5
    }
}
```

