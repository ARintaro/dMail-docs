# Websocket Communication Protocol

## Introduction

- To ensure the security and privacy of communication, all protocol contents, except for those marked with special symbols (with an asterisk (\*) or double asterisks (\*\*) ), are encrypted using the `AES` symmetric key encryption. The asterisk (\*) indicates that the protocol is unencrypted, while the double asterisks (\*\*) indicate that the protocol is encrypted using ```RSA``` public key encryption.

- All protocol contents, before encryption (after decryption), have the following format:

  ```json
  {
    "command": "***",
    "data": [data] // JSON or string or undefined
  }
  ```

- In the [Protocol Name] section following this API documentation, the required string for the ```command``` field in plain text protocols is specified. In the [Parameters] section, the first row of the table indicates the type of ```data```, and subsequent rows specify the fields required for the ```data``` section in plain text protocols (if ```data``` exists and is of type JSON).

- Since communication typically involves the client sending data to the server, and the server responding with corresponding return data to the client, most of the protocols in this API documentation are written in the C2S-S2C format. C2S (Client2Server) indicates communication from the client to the server, while S2C (Server2Client) indicates communication from the server to the client.

- After encryption, protocol contents are represented as strings encoded in ```base64```.

## Handshake Encryption Protocol

### SetConnectionSymKey(C2S)*

#### Description

Used to exchange AES key pairs with the server to achieve handshake encryption.

#### Parameters

| Parameter | Type   | Description             |
| --------- | ------ | ----------------------- |
| data      | string | 1024-bit RSA public key |

#### Example

```json
{
	"command": "SetConnectionSymKey",
	"data": "MIGJAoGBAKMxsngBaWseSIR+84kzKaG31WEgePkC3AxgeOmQN3zHPs2tBHgsH8eqP8BTOVOzX6Llqo2HwO8FBMWetNRYMytzIYzSNL728ANyBeoItoyhqd2vlZYqj9eJ5ImmYcRMgTwc8CVcDuonjFF8bR2Y+sULLprrzFLjFSgj6K2Te3wZAgMBAAE="
}
```

### SetConnectionPubKeyResponse(S2C)**

#### Encryption Method

Encrypt using the RSA public key provided by the [Handshake Encryption Protocol (C2S)*].

#### Description

Used to set the AES symmetric key for the client.

#### Parameters

| Parameter Name | Type        | Description                    |
| -------------- | ----------- | ------------------------------ |
| data           | JSON        | None                           |
| state          | string      | Response status from server    |
| pubKey         | string(opt) | AES key if response is correct |

| state         | Description                                     |      |
| ------------- | ----------------------------------------------- | ---- |
| NeedSetPubKey | Server has no RSA public key                    |      |
| PubKeyError   | RSA public key encryption failure on the server |      |
| HasApproved   | Handshake encryption already approved           |      |
| Success       | AES key issuance successful on the server       |      |

#### Example

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

## Login Protocol

### Login (C2S)

#### Description

Used for user login.

#### Parameters

| Parameter Name | Type        | Description             |
| -------------- | ----------- | ----------------------- |
| data           | JSON        | None                    |
| email          | string      | User's email            |
| password       | string(opt) | Salted hashed password  |
| emailCode      | number(opt) | Email verification code |
| token          | string(opt) | Server-issued token     |

#### Example

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

### LoginResponse (S2C)

#### Description

Used for server response to user login request.

#### Parameters

| Parameter Name | Type        | Description                          |
| -------------- | ----------- | ------------------------------------ |
| data           | JSON        | None                                 |
| state          | string      | User login response state            |
| userId         | number(opt) | User's userId after successful login |

| state          | Description                         |
| -------------- | ----------------------------------- |
| Success        | Successful login                    |
| EmailInvalid   | Invalid email format                |
| UserNotFound   | User does not exist in the database |
| UserLogged     | User is already logged in           |
| PasswordError  | Incorrect password                  |
| EmailCodeError | Incorrect verification code         |
| TokenError     | Invalid token                       |
| TokenExpired   | Token has expired                   |

#### Example

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

## Registration Protocol

### Register (C2S)

#### Description

Used for the client to send a registration account request to the server.

#### Parameters

| Parameter Name | Type   | Description             |
| -------------- | ------ | ----------------------- |
| data           | JSON   | None                    |
| userName       | string | User's username         |
| password       | string | Salted hashed password  |
| email          | string | User's email            |
| emailCode      | number | Email verification code |

#### Example

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

### RegisterResponse (S2C)

#### Description

Used for the server to respond to the client's registration request.

#### Parameters

| Parameter Name | Type        | Description                                 |
| -------------- | ----------- | ------------------------------------------- |
| data           | JSON        | None                                        |
| state          | string      | Response state of the registration request  |
| userId         | number(opt) | User's userId after successful registration |

| state               | Description                 |
| ------------------- | --------------------------- |
| Success             | Registration successful     |
| UserNameFormatError | Invalid username format     |
| PasswordFormatError | Invalid password format     |
| EmailRegistered     | Email already registered    |
| EmailCodeError      | Incorrect verification code |
| EmailInvalid        | Invalid email format        |

#### Example

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

## Initial Data Pull Protocol

### Pull (C2S)

#### Description

After successful login, the client automatically sends a request to the server to pull data for initializing chat information.

#### Parameters

| Parameter Name  | Type   | Description                                                  |
| --------------- | ------ | ------------------------------------------------------------ |
| data            | JSON   | None                                                         |
| lastRequestId   | number | Pull requests after the latest RequestId, 0 for full data pull |
| noticeTimestamp | number | Pull all notices after this timestamp                        |

#### Example

```json
{
    "command": "Pull",
    "data": {
        "lastRequestId": 0,
        "noticeTimestamp": 1685018341911
    }
}
```

### ReadCursors (S2C)

#### Description

The server sends the client the latest read positions for corresponding chats.

#### Parameters

| Parameter Name | Type  | Description                                |
| -------------- | ----- | ------------------------------------------ |
| data           | Array | None                                       |
| data[index]    | Array | Tuple [chatId, readCursor] for each chatId |

#### Example

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

### Messages (S2C)

#### Description

The server sends the client the latest partial messages for each chat.

#### Parameters

| Parameter Name | Type   | Description             |
| -------------- | ------ | ----------------------- |
| data           | Array  | None                    |
| data[index]    | string | Serialized message data |

#### Example

```json
{
    "command": "Messages",
    "data": [
        "{"type":"Text", "inChatId":64, "chatId":1, "senderId":7, "serializedContent":"\"\\\"XXX\\\"\"", "timestamp":1684332121757}",
		"{"type":"Text", "inChatId":65, "chatId":1, "senderId":3, "serializedContent":"\"www.hao.com www.hao.com www.hao.com\"", "timestamp":1684332445808}"
    ]
}
```

### Requests (S2C)

#### Description

The server sends the client the latest partial requests for each chat.

#### Parameters

| Parameter Name | Type   | Description             |
| -------------- | ------ | ----------------------- |
| data           | Array  | None                    |
| data[index]    | string | Serialized Request data |

#### Example

```json
{
    "command": "Requests",
    "data": [
        "{"info":{"reqId":1,"senderId":2,"message":"123","content":{"type":"JoinGroup","chatId":1}},"state":"Approved"}",
        "{"info":{"reqId":2,"senderId":1,"message":"test","content":{"type":"MakeFriend","receiverId":2}},"state":"Approved"}"
    ]
}
```

### UserSetting (S2C)

#### Description

The server sends user configuration data to the client.

#### Parameters

| Parameter Name | Type   | Description                        |
| -------------- | ------ | ---------------------------------- |
| data           | string | Serialized user configuration data |

#### Example

```json
{
    "command": "UserSetting",
    "data": "{"secondaryCheckChats":[],"userNickname":[],"notification":{"slient":true,"show":true,"muteChatIds":[6]},"topChatIds":[4,3]}"
}
```

### Notices (S2C)

#### Description

The server sends a list of partially fetched Notice data to the client.

#### Parameters

| Parameter Name | Type   | Description            |
| -------------- | ------ | ---------------------- |
| data           | Array  | None                   |
| data[index]    | string | Serialized Notice data |

#### Example

```json
{
    "command": "Notices",
    "data": []
}
```

### PullResponse (S2C)

#### Description

The server responds to the Pull request status from the client.

#### Parameters

| Parameter Name | Type   | Description                     |
| -------------- | ------ | ------------------------------- |
| data           | JSON   | None                            |
| state          | string | Response status of Pull request |

| state         | Description                         |
| ------------- | ----------------------------------- |
| Success       | Successful response to Pull request |
| DatabaseError | Database exception                  |

#### Example

```json
{
    "command": "PullResponse",
    "data": {
        "state": "Success"
    }
}
```

## User Information Retrieval Protocol

### GetUserInfo (C2S)

#### Description

The client requests user information from the server.

#### Parameters

| Parameter Name | Type   | Description |
| -------------- | ------ | ----------- |
| data           | number | User ID     |

#### Example

```json
{
    "command": "GetUserInfo",
    "data": 1
}
```

### GetUserInfoResponse (S2C)

#### Description

The server sends user information data to the client.

#### Parameters

| Parameter Name | Type        | Description                      |
| -------------- | ----------- | -------------------------------- |
| data           | JSON        | None                             |
| state          | string      | Response status                  |
| userId         | number(opt) | User ID (if successful)          |
| userName       | string(opt) | User name (if successful)        |
| avatarHash     | string(opt) | Avatar file hash (if successful) |

| state        | Description         |
| ------------ | ------------------- |
| Success      | Successful response |
| UserNotFound | User does not exist |
| ServerError  | Server error        |

#### Example

```json
{
    "command": "GetUserInfoResponse",
    "data": {
        "state": "Success",
        "userId": 8,
        "userName": "Rintaro",
        "avatarHash": "b9c9a45da34b134a835c0ec673ea4ce4"
    }
}
```

## Chat Data Retrieval Protocol

### GetChatInfo (C2S)

#### Description

The client requests chat information data from the server.

#### Parameters

| Parameter Name | Type   | Description |
| -------------- | ------ | ----------- |
| data           | number | Chat ID     |

#### Example

```json
{
    "command": "GetChatInfo",
    "data": 9
}
```

### Chat (S2C)

#### Description

The server sends chat information data to the client.

#### Parameters

| Parameter Name | Type   | Description               |
| -------------- | ------ | ------------------------- |
| data           | string | Serialized chat info data |

#### Example

```json
{
    "command": "Chat",
    "data": "{\"id\":9,\"name\":\"Create Group Chat Test 2\",\"avatarHash\":\"\"}"
}
```

## Group Chat Permission Related Protocols

### GetGroupUsers, GetGroupAdmin, GetGroupOwner (C2S)

#### Description

The client requests the user ID list of a group chat, the ID list of administrators, and the ID of the group owner from the server.

#### Parameters

| Parameter Name | Type   | Description |
| -------------- | ------ | ----------- |
| data           | number | Chat ID     |

#### Example

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

### GetGroupUsersResponse (S2C)

#### Description

The server sends the list of group chat member IDs to the client.

#### Parameters

| Parameter Name | Type         | Description      |
| -------------- | ------------ | ---------------- |
| data           | JSON         | None             |
| state          | string       | Response status  |
| chatId         | number (opt) | Chat ID          |
| userIds        | Array (opt)  | List of user IDs |
| userIds[index] | number       | User ID          |

| state        | Description                  |
| ------------ | ---------------------------- |
| Success      | Successful response          |
| NotGroupChat | The chat is not a group chat |
| ServerError  | Server error                 |

#### Example	

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

### GetGroupOwnerResponse (S2C)

#### Description

The server sends the ID of the group owner to the client.

#### Parameters

| Parameter Name | Type         | Description           |
| -------------- | ------------ | --------------------- |
| data           | JSON         | None                  |
| chatId         | number (opt) | Chat ID               |
| userId         | number (opt) | ID of the group owner |

| state         | Description                 |
| ------------- | --------------------------- |
| Success       | Successful response         |
| UserNotInChat | The user is not in the chat |
| ServerError   | Server error                |

#### Example

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

### GetGroupAdminResponse (S2C)

#### Description

The server sends the list of administrator IDs to the client.

#### Parameters

| Parameter Name | Type         | Description               |
| -------------- | ------------ | ------------------------- |
| data           | JSON         | None                      |
| chatId         | number (opt) | Chat ID                   |
| userIds        | Array (opt)  | List of administrator IDs |
| userIds[index] | number       | Administrator ID          |

| state         | Description                 |
| ------------- | --------------------------- |
| Success       | Successful response         |
| UserNotInChat | The user is not in the chat |
| ServerError   | Server error                |

#### Example

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

### GroupOwnerTransfer (C2S)

#### Description

Transfer the ownership of a group chat.

#### Parameters

| Parameter Name | Type   | Description                      |
| -------------- | ------ | -------------------------------- |
| data           | JSON   | None                             |
| chatId         | number | Chat ID                          |
| userId         | number | User ID to transfer ownership to |

#### Example

```json
{
    "command": "GroupOwnerTransfer",
    "data": {
        "chatId": 5,
        "userId": 2
    }
}
```

### GroupOwnerTransferResponse (S2C)

#### Description

The server responds to the request to transfer ownership of a group chat.

#### Parameters

| Parameter Name | Type   | Description     |
| -------------- | ------ | --------------- |
| data           | JSON   | None            |
| state          | string | Response status |
| chatId         | number | Chat ID         |
| userId         | number | User ID         |

| State         | Description                          |
| ------------- | ------------------------------------ |
| Success       | Successful response                  |
| NotOwner      | The requesting user is not the owner |
| UserNotInChat | User is not in the chat              |
| ServerError   | Server error                         |

#### Example

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

### RemoveGroupMember (C2S)

#### Description

Remove a group member from the chat.

#### Parameters

| Parameter Name | Type   | Description |
| -------------- | ------ | ----------- |
| chatId         | number | Chat ID     |
| userId         | number | User ID     |

#### Example

```json
{
    "command": "RemoveGroupMember",
    "data": {
		"chatId": 5,
        "userId": 2
    }
}
```

### RemoveGroupMemberResponse (S2C)

#### Description

The server responds to the request to remove a group member from the chat.

#### Parameters

| Parameter Name | Type         | Description                                       |
| -------------- | ------------ | ------------------------------------------------- |
| data           | JSON         | None                                              |
| state          | string       | Response status                                   |
| chatId         | number (opt) | Chat ID                                           |
| userId         | number (opt) | User ID of the removed member                  \| |

| state         | Description                                     |
| ------------- | ----------------------------------------------- |
| Success       | Successful response                             |
| SameUser      | Requesting user is the same as the removed user |
| NoPermission  | No permission to remove the member              |
| UserNotInChat | User is not in the chat                         |
| ServerError   | Server error                                    |

#### Example

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

## Private Chat Read Status Query Protocol

### GetUserReadInPrivate (C2S)

#### Description

The client requests the latest read message ID from the other user in a private chat.

#### Parameters

| Parameter Name | Type   | Description |
| -------------- | ------ | ----------- |
| data           | number | Chat ID     |

#### Example

```json
{
    "command": "GetUserReadInPrivate",
    "data": 72
}
```

### GetUserReadInPrivateResponse (S2C)

#### Description

The server sends the latest read message ID from the other user in a private chat to the client.

#### Parameters

| Parameter Name | Type         | Description            |
| -------------- | ------------ | ---------------------- |
| data           | JSON         | None                   |
| state          | string       | Response status        |
| chatId         | number (opt) | Chat ID                |
| inChatId       | number (opt) | Latest read message ID |

| state          | Description                    |
| -------------- | ------------------------------ |
| Success        | Successful response            |
| NotPrivateChat | The chat is not a private chat |
| UserNotInChat  | User is not in the chat        |
| ServerError    | Server error                   |

#### Example

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

## Group Chat Read Status Query Protocol

### GetUserReadInGroup (C2S)

#### Description

The client requests the list of member IDs who have read a specific message in a group chat.

#### Parameters

| Parameter Name | Type   | Description |
| -------------- | ------ | ----------- |
| data           | JSON   | None        |
| chatId         | number | Chat ID     |
| inChatId       | number | Message ID  |

#### Example

```json
{
    "command": "GetUserReadInGroup",
    "data": {
        "chatId": 4,
        "inChatId": 49
    }
}
```

### GetUserReadInGroupResponse (S2C)

#### Description

The server sends the list of member IDs who have read the specified message in a group chat to the client.

#### Parameters

| Parameter Name | Type         | Description             |
| -------------- | ------------ | ----------------------- |
| data           | JSON         | None                    |
| state          | string       | Response status         |
| chatId         | number (opt) | Chat ID                 |
| inChatId       | number (opt) | Message ID              |
| userIds        | Array (opt)  | List of read member IDs |
| userIds[index] | number       | User ID                 |

| state         | Description                  |
| ------------- | ---------------------------- |
| Success       | Successful response          |
| NotGroupChat  | The chat is not a group chat |
| UserNotInChat | User is not in the chat      |

#### Example

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

## Group Notice Protocol

### PullGroupNotice (C2S)

#### Description

The client requests incremental retrieval of group notice data from the server.

#### Parameters

| Parameter Name | Type   | Description                                                  |
| -------------- | ------ | ------------------------------------------------------------ |
| data           | JSON   | No data                                                      |
| chatId         | number | Chat ID                                                      |
| lastNoticeId   | number | The latest notice ID the client has, or 0 for full retrieval |

#### Example

```json
{
    "command": "PullGroupNotice",
    "data": {
        "chatId": 4,
        "lastNoticeId": 0
    }
}
```

### PullGroupNoticeResponse (S2C)

#### Description

The server sends the serialized group notice data list to the client.

#### Parameters

| Parameter Name     | Type        | Description                  |
| ------------------ | ----------- | ---------------------------- |
| data               | JSON        | None                         |
| state              | string      | Response status              |
| chatId             | number      | Chat ID                      |
| groupNotice        | Array (opt) | List of group notice data    |
| groupNotice[index] | string      | Serialized group notice data |

| state         | Description             |
| ------------- | ----------------------- |
| Success       | Successful response     |
| UserNotInChat | User is not in the chat |
| ServerError   | Server error            |

#### Example

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

### SendGroupNotice (C2S)

#### Description

Send a group notice.

#### Parameters

| Parameter Name | Type   | Description       |
| -------------- | ------ | ----------------- |
| data           | JSON   | None              |
| chatId         | number | Chat ID           |
| clientId       | number | Client message ID |
| notice         | string | Notice content    |

#### Example

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

### GroupNoticeResponse (S2C)

#### Description

The server responds to the client's group notice request.

#### Parameters

| Parameter Name | Type   | Description      |
| -------------- | ------ | ---------------- |
| data           | JSON   | None             |
| state          | string | Response status  |
| chatId         | number | Chat ID          |
| clientId       | number | Client chat ID   |
| noticeId       | number | Notice ID        |
| timestamp      | number | Notice timestamp |

| state               | Description                  |
| ------------------- | ---------------------------- |
| Success             | Successful response          |
| NoPermission        | No permission to send notice |
| LengthLimitExceeded | Group notice length exceeded |
| ServerError         | Server error                 |

#### Example

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

## Token Issuance Protocol

### ApplyForToken (C2S)

#### Description

The client requests token issuance from the server.

#### Parameters

| Parameter Name | Type      | Description |
| -------------- | --------- | ----------- |
| data           | undefined | None        |

#### Example

```json
{
    "command": "ApplyForToken"
}
```

### ApplyForTokenResponse (S2C)

#### Description

The server issues a token to the client.

#### Parameters

| Parameter Name | Type         | Description     |
| -------------- | ------------ | --------------- |
| data           | JSON         | None            |
| state          | string       | Response status |
| token          | string (opt) | Token           |
| timestamp      | number (opt) | Timestamp       |

| state       | Description         |
| ----------- | ------------------- |
| Success     | Successful response |
| ServerError | Server error        |

#### Example

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

## Message Sending and Operation Protocol

### SendMessage (C2S)

#### Description

Send a message.

#### Parameters

| Parameter Name    | Type   | Description             |
| ----------------- | ------ | ----------------------- |
| data              | JSON   | None                    |
| type              | string | Message type            |
| clientId          | number | Client message ID       |
| serializedContent | string | Serialized message body |
| timestamp         | number | Timestamp               |

| type        | Description     |
| ----------- | --------------- |
| Text        | Text message    |
| File        | File message    |
| Image       | Image message   |
| Voice       | Voice message   |
| Transfer    | Forward message |
| Revoked     | Recall message  |
| MentionText | Mention message |
| ReplyText   | Reply message   |

#### Example

```json
{
    "command": "SendMessage",
    "data": {
        "chatId": 4,
        "clientId": 1,
        "serializedContent": "\"Hello\"",
        "timestamp": 1685192622888,
        "type": "Text"
    }
}
```

### SendMessageResponse (S2C)

#### Description

Server response to client's message sending request.

#### Parameters

| Parameter Name | Type   | Description       |
| -------------- | ------ | ----------------- |
| data           | JSON   | None              |
| state          | string | Response status   |
| clientId       | number | Client message ID |
| chatId         | number | Chat ID           |
| inChatId       | number | Message ID        |
| timestamp      | number | Timestamp         |

| state               | Description         |
| ------------------- | ------------------- |
| Success             | Successful response |
| LengthLimitExceeded | Message too long    |
| UserNotInChat       | User not in chat    |
| UserNotLoggedIn     | User not logged in  |
| ServerError         | Server error        |

#### Example

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

### RevokeMessage (C2S)

#### Description

Recall a message.

#### Parameters

| Parameter Name | Type   | Description   |
| -------------- | ------ | ------------- |
| data           | JSON   | None          |
| chatId         | number | Chat ID       |
| inChatId       | number | Message ID    |
| method         | string | Recall method |

| method     | Description                      |
| ---------- | -------------------------------- |
| Sender     | Sender recalls their own message |
| GroupAdmin | Group chat admin recalls message |
| GroupOwner | Group owner recalls message      |

#### Example

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

### RevokeMessageResponse (S2C)

#### Description

Server response to recall request.

#### Parameters

| Parameter Name | Type   | Description     |
| -------------- | ------ | --------------- |
| data           | JSON   | None            |
| state          | string | Response status |
| chatId         | number | Chat ID         |
| inChatId       | number | Message ID      |

| state             | Description                          |
| ----------------- | ------------------------------------ |
| Success           | Successful response                  |
| TimeLimitExceeded | Unable to recall due to timeout      |
| PermissionsDenied | No permission to recall this message |
| MessageNotExisted | Message does not exist               |

#### Example

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

## Request Sending and Handling Protocol

### SendRequest (C2S)

#### Description

Send requests such as adding friends or applying to join a group.

#### Parameters

| Parameter Name     | Type        | Description                  |
| ------------------ | ----------- | ---------------------------- |
| data               | JSON        | None                         |
| clientId           | number      | Client message ID            |
| content            | JSON        | Request body                 |
| content.type       | string      | Request type                 |
| content.receiverId | number(opt) | Receiver ID for the request  |
| content.chatId     | number(opt) | Chat ID for the request      |
| message            | string      | Request verification message |

| content.type     | Description             |
| ---------------- | ----------------------- |
| MakeFriend       | Add friend              |
| JoinGroup        | Apply to join group     |
| GroupInvitation  | Group invitation        |
| InvitedJoinGroup | Group invitation review |

#### Example

```json
{
    "command": "SendRequest",
    "data": {
        "message": "Add me as a friend",
        "clientId": 1,
        "content": {
            "type": "MakeFriend",
            "receiverId": 15
        }
    }
}
```

### SendRequestResponse (S2C)

#### Description

Server response to the request sending request.

#### Parameters

| Parameter Name | Type        | Description                |
| -------------- | ----------- | -------------------------- |
| data           | JSON        | None                       |
| state          | string      | Response status            |
| clientId       | number      | Client message ID          |
| reqId          | number(opt) | Request ID (if successful) |

| state       | Description         |
| ----------- | ------------------- |
| Success     | Successful response |
| ServerError | Server error        |

#### Example

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

### SolveRequest (C2S)

#### Description

Handle a request.

#### Parameters

| Parameter Name | Type   | Description    |
| -------------- | ------ | -------------- |
| data           | JSON   | None           |
| reqId          | number | Request ID     |
| answer         | string | Action to take |

| answer   | Description         |
| -------- | ------------------- |
| Refused  | Refuse the request  |
| Approved | Approve the request |

#### Example

```json
{
    "command": "SolveRequest",
    "data": {
        "reqId": 15,
        "answer": "Refused"
    }
}
```

### SolveRequestResponse (S2C)

#### Description

Server response to user's request handling.

#### Parameters

| Parameter Name | Type   | Description     |
| -------------- | ------ | --------------- |
| data           | JSON   | None            |
| state          | string | Response status |
| reqId          | number | Request ID      |

| state     | Description                                  |
| --------- | -------------------------------------------- |
| Success   | Successful response                          |
| Nohandler | User has no permission to handle the request |

#### Example

```json
{
    "command": "SolveRequestResponse",
    "data": {
        "state": "Success",
        "reqId": 10
    }
}
```

### RequestStateUpdate (S2C)

#### Description

Server sends an instruction to the client to update the request state.

#### Parameters

| Parameter Name | Type   | Description   |
| -------------- | ------ | ------------- |
| data           | JSON   | N/A           |
| state          | string | Updated state |
| reqId          | number | Request ID    |

#### Example

```json
{
    "command": "RequestStateUpdate",
    "data": {
        "state": "Solved",
        "reqId": 10
    }
}
```

## Unfriend Protocol

### Unfriend (C2S)

#### Description

Unfriend a user.

#### Parameters

| Parameter Name | Type   | Description                    |
| -------------- | ------ | ------------------------------ |
| data           | number | ID of the friend to be removed |

#### Example

```json
{
    "command": "Unfriend",
    "data": 15
}
```

### UnfriendResponse (S2C)

#### Description

Server response to the client's unfriend request.

#### Parameters

| Parameter Name | Type        | Description             |
| -------------- | ----------- | ----------------------- |
| state          | string      | Response status         |
| chatId         | number(opt) | Chat ID (if successful) |

| state     | Description          |
| --------- | -------------------- |
| Success   | Successful response  |
| NotFriend | They are not friends |

#### Example

```json
{
    "command": "UnfriendResponse",
    "data": {
        "state": "Success",
        "chatId": 5
    }
}
```