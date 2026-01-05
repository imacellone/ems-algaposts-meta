# AlgaPosts System Implementation

You are required to develop a distributed system composed of two microservices that communicate **asynchronously** via **RabbitMQ**. The system is responsible for receiving text publications (posts), processing their content in the background, and persisting the processing results.
The system must support the following features:
* Create a new post
* Retrieve post details
* List posts with pagination

Post processing consists of counting the number of words in the text and calculating an estimated value based on the word count.

---

## 1. Microservice: PostService

* Expose a REST API for creating and querying posts.
* Send new posts to the queue `text-processor-service.post-processing.v1.q`.
* Consume processed results from the queue `post-service.post-processing-result.v1.q`.
* Store the post and the processing result.
* Use **H2** for persistence.
* Generate the post ID using **UUID**.
* Correctly link the processed result to the original post.

### Endpoints

#### POST /api/posts
* Creates a new post.
* Sends post data to the queue.
* Returns **201 Created** and a `PostOutput` in the body.

#### GET /api/posts/{postId}
* Returns `PostOutput`.
* Returns **200 OK** in case of success.
* Returns **404 Not Found** if the post does not exist.

#### GET /api/posts
* Returns a paginated list of `PostSummaryOutput`.
* Supports parameters `page` and `size`.
* Returns **200 OK** with the following structure:

```json
{
  "page": 0,
  "size": 10,
  "totalElements": 45,
  "totalPages": 5,
  "content": [ /* list of PostSummaryOutput */ ]
}
```

### DTOs used in PostService

**PostInput** - Creation model:
```json
{
  "title": "string",
  "body": "string",
  "author": "string"
}
```

**PostOutput** - Detailed display model:
```json
{
  "id": "string",
  "title": "string",
  "body": "string",
  "author": "string",
  "wordCount": 125,
  "calculatedValue": 12.5
}
```

**PostSummaryOutput** - Simplified model:
```json
{
  "id": "string",
  "title": "string",
  "summary": "string",
  "author": "string"
}
```

### Validations in PostService
* `title`, `body`, and `author` are mandatory.
* `body` must contain non-empty text.
* `id` must be a UUID and unique.
* `summary` contains the first 3 lines of the `body`.

---

## 2. Microservice: TextProcessorService

* Consume the queue `text-processor-service.post-processing.v1.q`.
* Process the `body` field of the received post.
* Calculate:
    * The quantity of words in the text body.
    * The estimated value (words * $0.10).
* Send the processing result to the queue `post-service.post-processing-result.v1.q`.

### Consumed Message Format (Queue: `text-processor-service.post-processing.v1.q`)
```json
{
  "postId": "string", // UUID
  "postBody": "string"
}
```

### Produced Message Format (Queue: `post-service.post-processing-result.v1.q`)
```json
{
  "postId": "string", // UUID
  "wordCount": 125,
  "calculatedValue": 12.5
}
```

---

## 3. General Rules for Both Microservices

* Create the type of **Exchange** you find necessary.
* Configure a **DLQ** (Dead Letter Queue) for each queue.
* Configure queues and exchanges via Java.
* Always use DTOs in REST communications and via messaging.
* Configure the Jackson serializer for JSON in messages.
* To send a message directly to a queue without a named Exchange, use:
  `rabbitTemplate.convertAndSend(queueName, messagePayload)`

---

## 4. Challenge Tasks

1.  Implement **PostService** with REST endpoints and queue sending/consuming logic.
2.  Implement **TextProcessorService** as a consumer of the posts queue and producer of the results queue.
3.  Ensure proper data persistence in PostService, including processed fields.
4.  Define input and output DTOs.
5.  Ensure HTTP responses are correct (200, 404, 201).
6.  Test the complete flow with RabbitMQ running.

### Tips
* Use logs to record and debug message exchange.
* Use whichever package model you prefer.
* Create Service classes if you wish.

Good luck with the challenge!