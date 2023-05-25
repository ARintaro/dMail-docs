### lst_upd

#### 数据类型

键值

#### 描述

维护最后一个文件上传请求ID，用于产生新的文件上传请求时分配id

#### 示例

```bash
redis-cli
set file:lst_upd 10
```

### upload

#### 数据类型

哈希表

#### 描述

维护上传请求ID到请求状态的JSON，当上传请求成功/失败后，字段会被删除

#### 示例

```bash
redis-cli
hset file:upload 1 json
```

### url

#### 数据类型

哈希表

#### 描述

维护上传请求的文件哈希到URL和URL过期时间的映射

#### 示例

```bash
redis-cli
hset file:url hash url&&expire
```

