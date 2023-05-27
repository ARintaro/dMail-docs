### last_id

#### 数据类型

键值

#### 描述

维护最后一个请求ID，用于产生新的请求时分配id

#### 示例

```bash
redis-cli
set req:last_id 10
```

### {req_id}:info

#### 数据类型

键值

#### 描述

存储序列化的请求信息

#### 示例

```bash
redis-cli
set req:1:info info
```

### {req_id}:state

#### 数据类型

键值

#### 描述

存储请求的状态，有null、true、false三种状态，

null表示未处理，true表示同意，false表示拒绝

#### 示例

```bash
redis-cli
set req:1:state true
```

