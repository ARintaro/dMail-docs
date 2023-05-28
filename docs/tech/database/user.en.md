### email_to_id

#### Data Type

Hash Table

#### Description

Maintains the mapping from email addresses to user IDs.

#### Example

```bash
redis-cli
hset user:email_to_id a@a.com 1
```

| Key (Email) | Value (user_id) |
| :---------: | :-------------: |
|   a@a.com   |        1        |
|   b@b.com   |        2        |

### fri_to_ch

#### Data Type

Hash Table

#### Description

Maintains the mapping from friend relationships to chat IDs.

Friend Relationship: {user1\_id : user2\_id}, where user1\_id < user2\_id.

#### Example

```bash
redis-cli
hset user:fri_to_ch 1:2 1
```

| Key (Friend Relationship) | Value (chat_id) |
| :-----------------------: | :-------------: |
|            1:2            |        1        |
|            1:3            |        2        |

### last_id

#### Data Type

Key

#### Description

Maintains the last user ID to allocate IDs for new user registrations.

#### Example

```bash
redis-cli
set user:last_id 10
```

### {user_id}:info

#### Data Type

Key

#### Description

Stores the serialized user information.

#### Example

```bash
redis-cli
set user:1:info info
```

### {user_id}:pass

#### Data Type

Key

#### Description

Stores the hashed user password.

#### Example

```bash
redis-cli
set user:1:pass password
```

### {user_id}:mail

#### Data Type

Key

#### Description

Stores the user's email address.

#### Example

```bash
redis-cli
set user:1:mail a@a.com
```

### {user_id}:chats

#### Data Type

Hash Table

#### Description

Stores the last seen message ID (in_chat_id) for each chat that the user has participated in.

#### Example

```bash
redis-cli
hset user:1:chats 1 10
```

| Key (chat_id) | Value (in_chat_id) |
| :-----------: | :----------------: |
|       1       |         10         |
|       2       |         12         |

### {user_id}:reqs

#### Data Type

Sorted Set

#### Description

Stores the requests related to the user in a sorted manner. The score is the req_id of each request, and the member is also the req_id.

#### Example

```bash
redis-cli
zadd user:1:reqs 10 10
```

| Member (req_id) | Score (req_id) |
| :-------------: | :------------: |
|        8        |       8        |
|       10        |       10       |

### {user_id}:not

#### Data Type

Sorted Set

#### Description

Stores important notifications for the user (mentioned messages and message recalls). It is used for synchronization between the server-side database and the frontend cache database. The score is the timestamp, and the member is the serialized notification content.

#### Example

```bash
redis-cli
zadd user:1:not 10000000 1
```

| Member (notice) | Score (timestamp) |
| :-------------: | :---------------: |
|     notice1     |      100000       |
|     notice2     |      200000       |

### name_to_id

#### Data Type

Hash Table

#### Description

Maintains a list of user IDs for each name. It is used to lookup user IDs based on names. The value is a string representation of a vector containing the IDs of users with the same name.

#### Example

```bash
redis-cli
hset user:name_to_id name [1,2,3]
```

| Key (name) | Value (ids) |
| :--------: | :---------: |
|   name1    |  [8,20,37]  |
|   name2    |    [10]     |

### {user_id}:settings

#### Data Type

Key-Value

#### Description

Stores the user's settings. It stores the serialized settings string received from the frontend.

#### Example

```bash
redis-cli
set user:1:settings setting
```

### {user_id}:token

#### Data Type

Key-Value

#### Description

Stores the temporary token for the user, used for automatic login.

#### Example

```bash
redis-cli
set user:1:token token
```

### {user_id}:pre_join

#### Data Type

Set

#### Description

Stores the group chat IDs for which the user has sent but unprocessed join requests.

#### Example

```bash
redis-cli
sadd user:1:pre_join 10
```

### {user_id}:exist

#### Data Type

Key-Value

#### Description

Indicates whether the user ID is still in use. It is set to 1 on registration and 0 on logout.

#### Example

```bash
redis-cli
set user:1:exist 1
```