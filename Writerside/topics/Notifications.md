# Notifications

This document provides detailed documentation for the `NotificationController` with added instructions for utilizing
WebSocket for real-time notification delivery.

### **Controller Description**

The `NotificationController` is a REST controller that handles CRUD operations for user notifications. It is versioned
under `/api/v1/notifications` and serves as part of the system's notification service.

**WebSocket Integration**: This controller works alongside WebSocket to enable real-time notification delivery to users
and groups. Notifications triggered by specific endpoints are sent to WebSocket destinations for immediate updates,
allowing clients to react in real-time.

## **Endpoints Overview**

### **1. Retrieve All Notifications**

- **HTTP Method**: `GET`
- **Endpoint**: `/api/v1/notifications`
- **Purpose**: Fetch a paginated list of all notifications.

#### **Request Parameters**

- No specific parameters required.

#### **Response** {id="response_1"}

- **Success**: Returns a `200 OK` response with a paginated list of `NotificationDto` objects.

#### **Example Request** {id="example-request_1"}

``` http
GET /api/v1/notifications HTTP/1.1
Host: example.com
Authorization: Bearer <token>
```

#### **Example Response** {id="example-response_1"}

``` json
{
  "content": [
    {
      "id": "d1223abc-d1df-42e2-bacd-46c08c168159",
      "receiverId": "e311c2f8-f05e-47d3-914c-5bfb35c87944",
      "title": "New Session Started",
      "message": "You started a parking session.",
      "creationDate": "2023-10-05T08:00:00",
      "modificationDate": "2023-10-05T09:00:00"
    }
  ],
  "pageable": {
    "page": 0,
    "size": 10
  }
}
```

#### **Errors/Edge Cases** {id="errors-edge-cases_1"}

- **401 Unauthorized**: If the user is not authenticated.
- **500 Internal Server Error**: If the notifications could not be retrieved due to a server error.

### **2. Mark a Notification as Read**

- **HTTP Method**: `PUT`
- **Endpoint**: `/api/v1/notifications/{notificationId}`
- **Purpose**: Marks a specific notification as read.

#### **Path Parameters** {id="path-parameters_1"}

- **`notificationId` **(`UUID`): Unique identifier of the notification to be marked as read.

#### **Response** {id="response_2"}

- **Success**: Returns a `200 OK` response with the updated `NotificationDto`.
- **Failure**:
    - `404 Not Found` if the notification does not exist.

#### **Example Request** {id="example-request_2"}

``` http
PUT /api/v1/notifications/d1223abc-d1df-42e2-bacd-46c08c168159 HTTP/1.1
Host: example.com
Authorization: Bearer <token>
```

#### **Example Response** {id="example-response_2"}

``` json
{
  "id": "d1223abc-d1df-42e2-bacd-46c08c168159",
  "receiverId": "e311c2f8-f05e-47d3-914c-5bfb35c87944",
  "title": "New Session Started",
  "message": "You started a parking session.",
  "creationDate": "2023-10-05T08:00:00",
  "modificationDate": "2023-10-05T09:00:00"
}
```

#### **Errors/Edge Cases** {id="errors-edge-cases_2"}

- **404 Not Found**: If the `notificationId` does not exist.
- **401 Unauthorized**: If the user is not authenticated.

### **3. Delete a Notification**

- **HTTP Method**: `DELETE`
- **Endpoint**: `/api/v1/notifications/{notificationId}`
- **Purpose**: Deletes a specific notification.

#### **Path Parameters** {id="path-parameters_2"}

- **`notificationId` **(`UUID`): Unique identifier of the notification to be deleted.

#### **Response** {id="response_3"}

- **Success**: Returns a `200 OK` response with no body.
- **Failure**:
    - `404 Not Found` if the notification does not exist.

#### **Example Request** {id="example-request_3"}

``` http
DELETE /api/v1/notifications/d1223abc-d1df-42e2-bacd-46c08c168159 HTTP/1.1
Host: example.com
Authorization: Bearer <token>
```

#### **Example Response** {id="example-response_3"}

``` http
HTTP/1.1 200 OK
```

#### **Errors/Edge Cases** {id="errors-edge-cases_3"}

- **404 Not Found**: If the `notificationId` does not exist.
- **401 Unauthorized**: If the user is not authenticated.

### **4. Send a Notification**

- **HTTP Method**: `POST`
- **Endpoint**: `/api/v1/notifications/{userId}`
- **Purpose**: Sends a notification to a specific user.

#### **Path Parameters**

- **`userId` **(`UUID`): Unique identifier of the user who will receive the notification.

#### **Request Body**

- **`NotificationDto` **:
    - **title** (`String`, required): The title of the notification.
    - **message** (`String`, required): The body/message of the notification.

#### **Response**

- **Success**: Returns a `200 OK` response with no body.

#### **Example Request**

``` http
POST /api/v1/notifications/e311c2f8-f05e-47d3-914c-5bfb35c87944 HTTP/1.1
Host: example.com
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "Reminder",
  "message": "Your parking session is about to end."
}
```

#### **Example Response**

``` http
HTTP/1.1 200 OK
```

#### **Errors/Edge Cases** {id="errors-edge-cases_5"}

- **400 Bad Request**: If the `title` or `message` is missing or invalid.
- **404 Not Found**: If the `userId` does not exist.
- **401 Unauthorized**: If the user is not authenticated.

#### **Real-Time WebSocket Delivery**

This endpoint integrates heavily with WebSocket:

1. When a notification is created, it is sent directly to the user's queue.
2. WebSocket ensures that notifications appear in real-time.

#### **WebSocket Example Usage**

- **Subscribed Destination**: `/user/{userId}/queue/notifications`
    - Message sent upon notification creation:

``` json
    {
  "id": "d1223abc-d1df-42e2-bacd-46c08c168149",
  "receiverId": "e311c2f8-f05e-47d3-914c-5bfb35c87944",
  "title": "Reminder",
  "message": "Your parking session is about to end.",
  "creationDate": "2023-10-05T08:00:00",
  "modificationDate": "2023-10-05T08:00:00",
  "isRead": false
}
```

## **General Response Structure**

The `ResponseEntity` class is used to structure all responses. It typically includes:

- **Status Code**: Represents success or failure (e.g., 200, 400, 404, etc.).
- **Headers**: Metadata related to the response.
- **Body**: This depends on the endpoint and usually contains the requested data or result.

## **WebSocket Integration Details**

The application uses Spring's **SimpMessagingTemplate** to integrate WebSocket. Notifications leverage two key
destinations:

1. **Personalized Notifications**:

- Subscribed destination: `/user/{userId}/queue/notifications`
- Receives notifications specific to a user (e.g., a reminder, read/unread status updates, deletions).
- Activated by the `sendPersonalNotification` method.

2. **Global (Broadcast) Notifications**:

- Subscribed destination: `/topic/global`
- Broadcasts system-wide announcements (e.g., maintenance updates).
- Activated by the `sendNotificationToAll` method.

## **WebSocket Client Setup**

To use WebSocket notifications in your application, follow these steps for client-side implementation:

### **1. Establish WebSocket Connection**

Using JavaScript and `STOMP`:

``` javascript
const socket = new SockJS('/ws'); // Backend WebSocket endpoint
const stompClient = Stomp.over(socket);

stompClient.connect({}, (frame) => {
    console.log("Connected: " + frame);

    // Subscribe to personalized notifications
    stompClient.subscribe('/user/queue/notifications', (message) => {
        const notification = JSON.parse(message.body);
        console.log("Received notification:", notification);
    });

    // Subscribe to global notifications
    stompClient.subscribe('/topic/global', (message) => {
        const broadcastMessage = JSON.parse(message.body);
        console.log("New global notification:", broadcastMessage);
    });
});
```

### **2. Handling Notifications**

- **Personalized Notifications**:
    - Display or manage notifications intended for a specific user.
    - Update the UI in response to actions such as marking as read or deletion.

- **Global Notifications**:
    - Display critical or system-wide announcements (e.g., maintenance notices).

## **Error Scenarios (WebSocket-Specific)**

- **User Not Subscribed**: If the user has not subscribed to `/user/{userId}/queue/notifications`, they won't receive
  personal notifications.
- **Connection Issues**: Ensure that the WebSocket connection is properly established before interacting with
  notifications.
- **Authentication**: WebSocket connections should ensure proper authentication by including any token mechanisms
  necessary.

With this integration, clients can retrieve notifications via REST and receive them in real-time through WebSocket
subscriptions, creating a seamless user experience for notification management and delivery.

## **Authentication**

All endpoints are secured and require a valid `Bearer` token included in the `Authorization` header. Ensure that the
user is authenticated to access these APIs.
