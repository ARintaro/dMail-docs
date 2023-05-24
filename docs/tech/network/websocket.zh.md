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

- 本API文档之后的[协议名称]部分，均指明文协议中command字段需要的字符串，[协议参数]部分，表格的第一行均指data的类型，其后各行指明文协议中data部分需要的字段（如果data存在且类型为JSON）。

- 由于通信通常是客户端向服务器端发送数据，服务器端在接受到相应的数据后向客户端发送对应的返回数据，故本API文档大部分协议采取C2S-S2C的形式编写。其中C2S(Client2Server)表示客户端向服务器端发送，S2C类似。

- 协议内容在加密后，均为```base64```编码的字符串。

### 握手加密协议(C2S)*

#### 协议名称

```SetConnectionSymKey```

用于和服务器交换AES密钥对，实现握手加密。

#### 协议参数

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

### 握手加密协议(S2C)**

#### 加密方法

使用[握手加密协议(C2S)*]提供的RSA公钥进行加密

#### 协议名称

```SetConnectionPubKeyResponse```

用于设置客户端的AES对称密钥

#### 协议参数

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

### 登录协议(C2S)

#### 协议名称

```Login```

用于用户登录

#### 协议参数

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
        "password": "testtesttest123"
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

### 登录协议(S2C)

#### 协议名称

```LoginResponse```

用于服务器端响应用户登录请求

#### 协议参数

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

