### last_id

#### Data Type

Key-Value

#### Description

Maintains the last chat ID to generate new IDs for chats.

#### Example

```bash
redis-cli
set chat:last_id 10
```

### {chat_id}:last_id

#### Data Type

Key-Value

#### Description

Maintains the last in_chat_id for messages in a chat to generate new IDs for chat messages.

#### Example

```bash
redis-cli
set chat:1:last_id 10
```

### {chat_id}:info

#### Data Type

Key-Value

#### Description

Stores the serialized information of the corresponding chat.

#### Example

```bash
redis-cli
set chat:1:info info
```

### {chat_id}:owner

#### Data Type

Key-Value

#### Description

Only applicable to group chats. Stores the ID of the owner of the group chat, which can be used to verify if a chat is a private chat or a group chat.

#### Example

```bash
redis-cli
set chat:1:owner 10
```

### {chat_id}:admins

#### Data Type

Set

#### Description

Only applicable to group chats. Stores the IDs of all the administrators of the group chat.

#### Example

```bash
redis-cli
sadd chat:1:admins 10
```

### {chat_id}:notices

#### Data Type

Sorted Set

#### Description

Only applicable to group chats. Stores the group notices of the group chat. The sorting criterion (score) is the notice_id, and the stored content (member) is the serialized content of the group notice.

#### Example

```bash
redis-cli
zadd chat:1:notices 2 notices
```

### {chat_id}:last_notice_id

#### Data Type

Key-Value

#### Description

Only applicable to group chats. Stores the last ID of the group notice, used to generate notice_id for new group announcements.

#### Example

```bash
redis-cli
set chat:1:last_notice_id 10
```

### {chat_id}:users

#### Data Type

Set

#### Description

Stores the user IDs in the chat.

#### Example

```bash
redis-cli
sadd chat:1:users 1
```

### {chat_id}:msgs

#### Data Type

Sorted Set

#### Description

Stores the messages in the chat in sequential order. The sorting criterion (score) is the in_chat_id, and the stored content (member) is the serialized message record.

#### Example

```bash
redis-cli
zadd chat:1:msgs 2 msg
```

### {chat_id}:0

#### Data Type

Key-Value

#### Description

Only applicable to private chats. Stores the smaller ID of the user in the private chat.

#### Example

```bash
redis-cli
set chat:1:0 1
```

### {chat_id}:1

#### Data Type

Key-Value

#### Description

Only applicable to private chats. Stores the larger ID of the user in the private chat.

#### Example

```bash
redis-cli
set chat:1:1 2
```

### Invitations

#### Data Type

Hash Table

#### Description

When a user receives an invitation to join a group chat, a key in the format "inviter:invitee:chat id" is added to this hash table. The key is removed after the invitation is processed.

#### Example

```bash
redis-cli
hset Invitations 2:3:3 1
```