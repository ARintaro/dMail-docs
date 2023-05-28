# Software Design Document

## Project Background

Currently, instant messaging services like WeChat, Telegram, etc., have become essential tools for individuals or organizations to exchange information on the internet. However, for end-users, using public instant messaging services for communication and data transmission poses a significant risk of information leakage. Chat messages, files, and data in public instant messaging platforms are stored by the service providers, and users have limited control over them. Most public instant messaging services are closed-source due to their profit-oriented nature, making it difficult for users to verify the effectiveness of service providers' commitments to information security. Therefore, deploying a private instant messaging service using an open-source project is crucial to maximize user information security.

However, the current open-source community lacks a comprehensive ecosystem for instant messaging. In this context, this project aims to provide a private instant messaging deployment solution that achieves a balance between deployment cost, reliability, and performance.

## Project Objectives

To provide users with an easy-to-deploy, high-performance, and secure solution for private instant messaging deployment.

## Overview of Technology Stack

For most small and medium-sized instant messaging deployments, distributed deployment costs are unaffordable. Therefore, single-machine performance is crucial, as it needs to handle a large number of concurrent requests on a single connected server. On the other hand, the system also needs sufficient horizontal scalability to meet the demands of serving more users.

Therefore, the overall technology stack selection focuses on sacrificing development speed and feature richness in exchange for performance.

* The server-side development language chosen is `Rust`. The primary consideration is undoubtedly performance and memory safety. In addition, Rust's robust error handling mechanism can improve code quality. As a compiled language, it also reduces user deployment costs, as there is no need for additional runtime installations or complex dependencies. Its powerful generic capabilities achieve zero runtime cost abstractions, and the main code abstractions rely on generics and conditional compilation rather than virtual tables.
* The web client utilizes the `React` framework with `TypeScript` as the programming language, and `Mobx` as the state management library. Compared to traditional web applications with a B/S architecture, an instant messaging system, as a single-page application, resembles more of a C/S program. The native approach of React makes it difficult to separate data and rendering, and accumulating a sufficient amount of functional code can significantly decrease the project's maintainability. Furthermore, due to browser limitations, only `WebSocket` can be used for maintaining long connections, so a protocol is wrapped on top of `WebSocket`.
* The backend database layer implements a complete abstraction and uses conditional compilation to switch between different databases. However, due to time constraints, only the compatibility with Redis protocol databases is implemented. Currently, `Pika`, an open-source, persistent, Redis-protocol-compatible distributed database, is used for actual deployments. While SQL provides rich functionality, its QPS is far from meeting the requirements of instant messaging, and distributed scaling requires modifying backend logic manually to implement sharding and partitioning, unlike key-value databases, which are more convenient to deploy in a distributed manner.

## Technical Selection Analysis

### Analysis of the Connection Server Architecture

The framework selected for the web service is `Actix Web`, and all operations that require IO are implemented asynchronously to ensure non-blocking functionality.

The connection server utilizes two instances of `Tokio` runtime, each running on a separate thread pool.

The first thread pool is responsible for handling connections and simple requests. The actors provided by the `Actix` framework also run on this thread pool. The asynchronous model of the `Actor` used for user connections ensures that even if a user thread crashes, it will not affect the normal operation of the backend services. It also provides a convenient solution for distributed scaling.

The second thread pool handles high-load tasks such as message retrieval for read diffusion. These tasks involve extensive computation, string concatenation, and significant waiting time for IO operations. It is undesirable for these tasks to impact user connections, so they are executed on a separate thread pool. However, a basic thread pool can only run synchronously. Therefore, another instance of the `Tokio` runtime is launched to handle these tasks. It is worth noting that this approach may not be optimal on machines with insufficient CPU cores.

The connection maintenance hash table utilizes distributed locks to minimize lock conflicts. Apart from this hash table, the backend service is nearly stateless, with all information stored in a database.

These design choices provide the connection server with strong horizontal scalability, and increasing the number of CPU cores in a single machine ensures performance improvement. The main bottleneck in terms of processing speed for individual requests lies in the database. Based on this fact, a distributed deployment of the database combined with a single-machine connection server can handle a large number of user requests.

### Communication Protocol Design

Due to the possibility of IM services being deployed without a domain name, making it impossible to sign HTTPS certificates, we have implemented our own encryption protocol similar to HTTPS. Although it lacks third-party signatures and is vulnerable to man-in-the-middle attacks, this encryption protocol is used concurrently with HTTPS in the test services deployed on Secoder and Tencent Cloud. In practice, once HTTPS is enabled, there is no need to use this custom protocol anymore. Additionally, this protocol increases the debugging cost for attackers, as they would need to implement the encryption protocol themselves in order to communicate directly with the backend.

Specifically, during WebSocket connection establishment, the client sends its RSA public key to the server. The server generates a symmetric encryption key using AES-GCM-128 and encrypts the symmetric key using the user's RSA public key, sending it back to the client. The client decrypts the message using its RSA private key and uses the symmetric encryption key for further communication. Subsequently, all information is transmitted through the encrypted WebSocket channel, eliminating the need for HTTP connections and improving server performance.

### Database Selection

Traditional web applications often use SQL databases to store information. However, for an IM service, the QPS (Queries Per Second) of SQL databases is relatively low, and distributed scaling can be complex, requiring manual sharding and table partitioning, which leads to business logic intrusion. Moreover, whether it is write diffusion or read diffusion, maintaining a large number of timelines is necessary. In SQL databases, multiple timelines are typically stored within a single table, which means that operations related to a single timeline, such as writing or reading, are influenced by the logarithmic scale of the entire table rather than just the scale of the individual timeline. On the other hand, KV (Key-Value) databases naturally separate these timelines, resulting in better complexity for individual operations. They are also well-suited for distributed support, as different timelines can be distributed across multiple machines. However, a disadvantage is that most open-source distributed KV databases do not support transactions and have lower data integrity. Another alternative is to combine

 SQL with Redis caching for optimization, but this approach introduces complexity in terms of updating, querying, and writing to the cache, making it more cumbersome to implement.

In practice, many KV databases are compatible with the Redis protocol, and therefore, we have chosen the Redis protocol. Currently, the test services deployed online utilize the open-source object storage service called `Pika` from 360. Another alternative is `Tendis`, an open-source project released by Tencent.

### Data Diffusion with Caching

The web client employs `LocalForge` to cache almost all data, avoiding repeated fetching from the server. Only when a user actually views certain information, does the client request it from the server.

With the implementation of caching, the client logic becomes "do not fetch from the server if the data is already present locally." However, in many cases, this data is not immutable. For example, a user's avatar and nickname may change over time, and in such cases, if the client does not fetch the updated data, it cannot stay in sync with the latest data from the server. Similar to WeChat, we re-fetch the data from the server when the user opens the specific information of another user.

Another challenging feature is the "retraction" of messages. Any existing chat message on the client-side can be retracted on the server-side. If the client is offline when a retraction occurs, it cannot determine the validity of the cached data, resulting in cache invalidation and the need to fetch messages from the server every time to refill the cache. We address this issue by creating a separate timeline (or write diffusion) for each user, specifically designed to handle real-time modifications to client-cached data, such as retractions.

Additionally, since incremental fetching is implemented, each chat session does not fetch the complete message history during login but only retrieves the most recent fixed number of messages. This poses a problem for the "mention" feature since messages mentioning a user might not be fetched by the client, and the client remains unaware of being mentioned. To solve this problem, we send write diffusion messages in the aforementioned timeline to indicate mentions.

This timeline itself uses a timestamp stored on the client side for incremental fetching, and the server-side synchronizes with the client's timestamp after retrieval.

### Direct Upload to Object Storage Service (OSS)

Instant messaging services need to transmit binary objects, and transferring binary files can significantly consume server bandwidth and disk resources. To mitigate this, it is necessary to separate the transmission service from the connection server, ensuring that the transfer service does not use connection server resources. Thus, we enable users to directly upload files to the object storage service through the server while maintaining control over file permissions and conducting file checks. Below is the complete file upload process:

1. The client sends a request to the connection server, including the file hash and size to be uploaded.
2. The connection server checks the validity of the request (whether the hash exists and the size is valid). If the request passes the verification, the server generates a file GUID and returns a pre-signed URL for file upload to the object server, allowing the user to upload the file directly.
3. The client uploads the file directly to the object server and sends a message to the connection server upon successful upload.
4. Upon receiving the "upload complete" message, the connection server queries the file hash and size corresponding to the GUID from the object upload server. It then compares this information with the hash and size initially sent by the client in the first step to verify whether the client deceived the server during the initial request.
5. If the verification is successful, the connection server returns a pre-signed download link to the user. If the verification fails, the object server deletes the file.

In practice, we have chosen to be compatible with the Amazon S3 protocol. Currently, the test services deployed online use the open-source object storage service called `Minio`. However, compared to mature commercial closed-source projects, the available functionality is relatively limited.

### Audio and Video Calls

WebRTC technology is used to implement video calls, which is almost the only solution for video calls in the browser with numerous restrictions. Establishing P2P connections through local network traversal often fails in complex network environments. Therefore, we have also deployed a COTURN server as a TURN server for traffic relay when traversal fails.

### Virtual DOM

The message list utilizes the `Virtuso` library to implement a virtual DOM, where only a limited number of nodes are actually added to the DOM tree. Nodes are switched in real-time as the user scrolls, avoiding any stuttering caused by rendering a large number of messages on the frontend.