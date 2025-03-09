# Reviews

The `ReviewController` manages interactions with parking reviews. It provides endpoints to create, find, delete, report
reviews, and manage their moderation status.

- **Base Path:** `/api/v1/reviews`
- **Versioning:** The `v1` in the path ensures future compatibility with evolving API versions.
- **Authentication:** Certain endpoints enforce role-based permissions (`DRIVER`, `MODERATOR`, `ADMIN`) for access.

### **Endpoints Overview**

| **Method** | **Path**             | **Description**                               | **Roles Allowed**              |
|------------|----------------------|-----------------------------------------------|--------------------------------|
| `GET`      | `/search`            | Search reviews with filters and pagination.   | Public                         |
| `POST`     | `/`                  | Create a new parking review.                  | `DRIVER`                       |
| `DELETE`   | `/{reviewId}`        | Delete a review by its ID.                    | `DRIVER`, `MODERATOR`, `ADMIN` |
| `POST`     | `/{reviewId}/report` | Mark a review as reported for further review. | `DRIVER`, `MODERATOR`, `ADMIN` |
| `PUT`      | `/{reviewId}/status` | Change the moderation status of a review.     | `MODERATOR`, `ADMIN`           |

## **Endpoint Details**

### **1. Search Reviews**

- **Path:** `GET /api/v1/reviews/search`
- **Description:** Retrieve a paginated and filtered list of reviews.
- **Request Parameters:**

| **Type**    | **Name**    | **Description**                                             | **Required** | **Example**       |
|-------------|-------------|-------------------------------------------------------------|--------------|-------------------|
| Query       | `size`      | Number of reviews per page (default: `10`).                 | No           | `5`               |
| Query       | `page`      | Page number (default: `0`).                                 | No           | `0`               |
| Query       | `sort`      | Field to sort by (`date`, `rating`). (default: `date`).     | No           | `rating`          |
| Query       | `direction` | Sorting direction (`asc`, `desc`). (default: `desc`).       | No           | `asc`             |
| RequestBody | ReviewDto   | Filters such as author, parking, moderation status, rating. | No           | `{ "rating": 5 }` |

- **Response:**

| **Status Code** | **Response Type** | **Description**                                |
|-----------------|-------------------|------------------------------------------------|
| `200 OK`        | `Page<ReviewDto>` | A list of reviews matching specified criteria. |
| `404 Not Found` | `Error`           | If no reviews matched the criteria.            |

- **Example Request-Response:**

| **Request**                                                   | **Response**                                                                                      |
|---------------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| `GET /api/v1/reviews/search?size=5&sort=rating&direction=asc` | `{"content":[{"reviewText":"Great parking","rating":5,"authorId":"1234"}],"pageable":{"size":5}}` |

### **2. Create Review**

- **Path:** `POST /api/v1/reviews`
- **Description:** Create a new review.
- **Authentication:** Requires `DRIVER` role.
- **Request Body:**

| **Name**     | **Type** | **Description**                                  | **Required** | **Example**             |
|--------------|----------|--------------------------------------------------|--------------|-------------------------|
| `reviewText` | `String` | Text of the review.                              | Yes          | `"Nice parking area!"`  |
| `rating`     | `int`    | Rating (range 1-5).                              | Yes          | `5`                     |
| `authorId`   | `UUID`   | Unique identifier of the review's author.        | Yes          | `"1234-abcd-5678-efgh"` |
| `parkingId`  | `UUID`   | Unique identifier of the parking being reviewed. | Yes          | `"5678-dcba-1234-hgfe"` |

- **Response:**

| **Status Code**   | **Response Type** | **Description**                             |
|-------------------|-------------------|---------------------------------------------|
| `200 OK`          | `ReviewDto`       | Details of the created review.              |
| `404 Not Found`   | `Error`           | If the specified parking does not exist.    |
| `400 Bad Request` | `Error`           | If a review already exists for the parking. |

- **Example Request-Response:**

| **Request**                                                                                    | **Response**                                                                                                                                         |
|------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| `POST /api/v1/reviews`                                                                         
 { "reviewText": "Good service", "rating": 4, "authorId": "uuid1234", "parkingId": "uuid5678" } | { "reviewText": "Good service", "rating": 4, "authorId": "uuid1234", "parkingId": "uuid5678", "moderationStatus": "PENDING", "creationDate": "..." } |

### **3. Delete Review**

- **Path:** `DELETE /api/v1/reviews/{reviewId}`
- **Description:** Delete an existing review by its unique identifier.
- **Authentication:** Requires `DRIVER`, `MODERATOR`, or `ADMIN` roles.
- **Request Parameters:**

| **Type**      | **Name**   | **Description**                      | **Required** | **Example**             |
|---------------|------------|--------------------------------------|--------------|-------------------------|
| Path Variable | `reviewId` | The unique identifier of the review. | Yes          | `"uuid-1234-abcd-4567"` |

- **Response:**

| **Status Code** | **Response Type** | **Description**                                   |
|-----------------|-------------------|---------------------------------------------------|
| `200 OK`        | `Void`            | If the review was successfully deleted.           |
| `404 Not Found` | `Error`           | If the review does not exist.                     |
| `403 Forbidden` | `Error`           | If the user is unauthorized to delete the review. |

- **Example Request-Response:**

| **Request**                                  | **Response** |
|----------------------------------------------|--------------|
| `DELETE /api/v1/reviews/uuid-1234-abcd-4567` | `200 OK`     |

### **4. Report Review**

- **Path:** `POST /api/v1/reviews/{reviewId}/report`
- **Description:** Mark a review for reporting/multiple moderation.
- **Authentication:** Requires `DRIVER`, `MODERATOR`, or `ADMIN` roles.
- **Request Parameters:**

| **Type**      | **Name**   | **Description**                      | **Required** | **Example**             |
|---------------|------------|--------------------------------------|--------------|-------------------------|
| Path Variable | `reviewId` | The unique identifier of the review. | Yes          | `"uuid-5678-dcba-9876"` |

- **Response:**

| **Status Code** | **Response Type** | **Description**                          |
|-----------------|-------------------|------------------------------------------|
| `200 OK`        | `Void`            | If the review was successfully reported. |
| `404 Not Found` | `Error`           | If the review does not exist.            |

### **5. Change Review Moderation Status**

- **Path:** `PUT /api/v1/reviews/{reviewId}/status`
- **Description:** Update a review's moderation status (`APPROVED`, `REJECTED`, or `PENDING`).
- **Authentication:** Requires `MODERATOR` or `ADMIN` roles.
- **Request Parameters:**

| **Type**      | **Name**   | **Description**                      | **Required** | **Example**             |
|---------------|------------|--------------------------------------|--------------|-------------------------|
| Path Variable | `reviewId` | The unique identifier of the review. | Yes          | `"uuid-5678-dcba-9876"` |
| Query Param   | `status`   | The new moderation status.           | Yes          | `"APPROVED"`            |

- **Response:**

| **Status Code** | **Response Type** | **Description**                         |
|-----------------|-------------------|-----------------------------------------|
| `200 OK`        | `Void`            | If the status was updated successfully. |
| `404 Not Found` | `Error`           | If the review ID does not exist.        |

- **Example Request-Response:**

| **Request**                                                      | **Response** |
|------------------------------------------------------------------|--------------|
| `PUT /api/v1/reviews/uuid-5678-dcba-9876/status?status=APPROVED` | `200 OK`     |

### **Error Handling**

| **Error Code**    | **Possible Cause**                                            |
|-------------------|---------------------------------------------------------------|
| `404 Not Found`   | Resource does not exist (e.g., invalid review ID).            |
| `400 Bad Request` | Invalid input or operation (e.g., duplicate review creation). |
| `403 Forbidden`   | Unauthorized access to secured endpoint.                      |
