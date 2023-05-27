### email_to_id

#### 数据类型

哈希

#### 描述

维护从邮箱到用户id的映射

#### 示例

```bash
redis-cli
hset user:email_to_id a@a.com 1
```

| key（邮箱） | value（user_id） |
| :---------: | :--------------: |
|   a@a.com   |        1         |
|   b@b.com   |        2         |

### fri_to_ch

#### 数据类型

哈希

#### 描述

维护朋友关系到chat id的映射

朋友关系：{user1\_id : user2\_id}，其中​user1\_id < user2\_id

#### 示例

```bash
redis-cli
hset user:fri_to_ch 1:2 1
```

| key（朋友关系） | value（chat_id） |
| :-------------: | :--------------: |
|       1:2       |        1         |
|       1:3       |        2         |

### last_id

#### 数据类型

键

#### 描述

维护最后一个用户ID，用于新用户注册时分配ID

#### 示例

```bash
redis-cli
set user:last_id 10
```

### {user_id}:info

#### 数据类型

键

#### 描述

存储序列化的用户信息

#### 示例

```bash
redis-cli
set user:1:info info
```

### {user_id}:pass

#### 数据类型

键

#### 描述

存储用户密码（经过哈希）

#### 示例

```bash
redis-cli
set user:1:pass password
```

### {user_id}:mail

#### 数据类型

键

#### 描述

存储用户邮箱

#### 示例

```bash
redis-cli
set user:1:mail a@a.com
```

### {user_id}:chats

#### 数据类型

哈希表

#### 描述

存储用户每个chat中看到最后一条消息的id（in_chat_id）。

#### 示例

```bash
redis-cli
hset user:1:chats 1 10
```

| key（chat_id） | value（in_chat_id） |
| :------------: | :-----------------: |
|       1        |         10          |
|       2        |         12          |

### {user_id}:reqs

#### 数据类型

有序集合

#### 描述

按序存储与用户有关的请求，其中的排序标准（score）是每个请求的req_id，存储内容（member）也是每个请求的req_id

#### 示例

```bash
redis-cli
zadd user:1:reqs 10 10
```

| member（req_id） | score（req_id） |
| :--------------: | :-------------: |
|        8         |        8        |
|        10        |       10        |

### {user_id}:not

#### 数据类型

有序集合

#### 描述

按序存储用户的重要通知（提及消息、撤回消息），用于服务端数据库与前端缓存数据库的同步，其中排序标准（score）是时间戳，存储内容（member）是序列化的通知内容

#### 示例

```bash
redis-cli
zadd user:1:not 10000000 1
```

| member（notice） | score（timestamp） |
| :--------------: | :----------------: |
|     notice1      |       100000       |
|     notice2      |       200000       |

### name_to_id

#### 数据类型

哈希表

#### 描述

维护每个姓名下的所有用户id，用于根据姓名查找用户id，其中键值的ids是所有同名用户的id组成的向量转成的字符串。

#### 示例

```bash
redis-cli
hset user:name_to_id name [1,2,3]
```

| key（name） | value（ids） |
| :---------: | :----------: |
|    name1    |  [8,20,37]   |
|    name2    |     [10]     |

### {user_id}:settings

#### 数据类型

键值

#### 描述

存储用户设置，存储前端发送的已序列化的的settings字符串

#### 示例

```bash
redis-cli
set user:1:settings setting
```

### {user_id}:token

#### 数据类型

键值

#### 描述

存储用户的临时令牌，用于自动登录

#### 示例

```bash
redis-cli
set user:1:token token
```

### {user_id}:pre_join

#### 数据类型

集合

#### 描述

存储了用户发送了但未被处理的加群申请的群聊id

#### 示例

```bash
redis-cli
sadd user:1:pre_join 10
```

### {user_id}:exist

#### 数据类型

键值

#### 描述

表示该用户id是否仍被使用，在注册时置1，注销时置0

#### 示例

```bash
redis-cli
set user:1:exist 1
```

