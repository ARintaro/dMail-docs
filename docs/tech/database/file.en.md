### lst_upd

#### Data Type

Key-Value

#### Description

Maintains the last file upload request ID to generate new IDs for file upload requests.

#### Example

```bash
redis-cli
set file:lst_upd 10
```

### upload

#### Data Type

Hash Table

#### Description

Maintains the mapping of upload request IDs to request status JSON. The fields are deleted after a successful or failed upload request.

#### Example

```bash
redis-cli
hset file:upload 1 json
```

### url

#### Data Type

Hash Table

#### Description

Maintains the mapping of file hashes in upload requests to the URL and expiration time.

#### Example

```bash
redis-cli
hset file:url hash url&&expire
```