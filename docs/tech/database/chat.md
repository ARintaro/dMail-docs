### last_id

#### 数据类型

键值

#### 描述

维护最后一个聊天ID，用于产生新的聊天时分配id

#### 示例

```bash
redis-cli
set chat:last_id 10
```

### {chat_id}:last_id

#### 数据类型

键值

#### 描述

维护聊天中消息的最后一个in_chat_id，用于产生新的聊天消息时分配in_chat_id

#### 示例

```bash
redis-cli
set chat:1:last_id 10
```

### {chat_id}:info

#### 数据类型

键值

#### 描述

存储对应聊天的序列化后的信息

#### 示例

```bash
redis-cli
set chat:1:info info
```

### {chat_id}:owner

#### 数据类型

键值

#### 描述

仅群聊有该字段，记录群聊的群主id，可以用于验证聊天是私聊还是群聊

#### 示例

```bash
redis-cli
set chat:1:owner 10
```

### {chat_id}:admins

#### 数据类型

集合

#### 描述

仅群聊有该字段，记录群聊的所有管理员id

#### 示例

```bash
redis-cli
sadd chat:1:admins 10
```

### {chat_id}:notices

#### 数据类型

有序集合

#### 描述

仅群聊有该字段，存储群聊的群公告，其中排序标准（score）是notice_id，存储内容（member）是序列化的群公告内容

#### 示例

```bash
redis-cli
zadd chat:1:notices 2 notices
```

### {chat_id}:last_notice_id

#### 数据类型

键值

#### 描述

仅群聊有该字段，存储群聊的群公告的最后一个id，用于生成群公告是分配notice_id

#### 示例

```bash
redis-cli
set chat:1:last_notice_id 10
```

### {chat_id}:users

#### 数据类型

集合

#### 描述

存储聊天中的用户id

#### 示例

```bash
redis-cli
sadd chat:1:users 1
```

### {chat_id}:msgs

#### 数据类型

有序集合

#### 描述

按序存储聊天中的消息，其中排序标准（score）是in_chat_id，存储内容（member）是序列化的消息记录

#### 示例

```bash
redis-cli
zadd chat:1:msgs 2 msg
```

### {chat_id}:0

#### 数据类型

键值

#### 描述

仅私聊有该字段，存储私聊中较小的用户id

#### 示例

```bash
redis-cli
set chat:1:0 1
```

### {chat_id}:1

#### 数据类型

键值

#### 描述

仅私聊有该字段，存储私聊中较大的用户id

#### 示例

```bash
redis-cli
set chat:1:1 2
```

