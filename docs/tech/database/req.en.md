### last_id

#### Data Type

Key-Value

#### Description

Maintains the last request ID to generate new IDs for requests.

#### Example

```bash
redis-cli
set req:last_id 10
```

### {req_id}:info

#### Data Type

Key-Value

#### Description

Stores the serialized information of the request.

#### Example

```bash
redis-cli
set req:1:info info
```

### {req_id}:state

#### Data Type

Key-Value

#### Description

Stores the state of the request, which can have three values: null, true, and false.

- null: the request is not processed.
- true: the request.
- false: the request.

#### Example

```bash
redis-cli
set req:1:state true
```