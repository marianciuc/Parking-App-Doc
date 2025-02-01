# Users Manipulations

The `UserController` class provides RESTful endpoints for managing users within the application. It includes functionalities like searching, retrieving user details, blocking/unblocking users, and changing passwords.

### Base Endpoint
All user-related APIs are accessed via **`/api/v1/users`**. The versioning prefix (`/api/v1`) allows for backward compatibility in future updates.

### Role in the System
This controller plays a key role in user management, interacting with `UserService` to fetch, update, and administer user data. These endpoints are secured with JWT authentication (`bearerAuth`) and authorization mechanisms.

---

## Endpoints Documentation

### 1. Search Users
#### **GET** `/api/v1/users/search`

Retrieve a paginated list of users based on optional search criteria such as email, record status, and user type.

**Request Parameters:**

| Parameter       | Type         | Required | Location  | Description                                                                                                                                  |
|------------------|--------------|----------|-----------|----------------------------------------------------------------------------------------------------------------------------------------------|
| `email`         | `String`     | No       | Query     | Filter users by email (optional).                                                                                                          |
| `page`          | `Integer`    | No       | Query     | Page number for pagination (default is 0).                                                                                                 |
| `size`          | `Integer`    | No       | Query     | Page size for pagination (default is 10).                                                                                                  |
| `recordStatus`  | `Enum`       | No       | Query     | Filter users by record status (`ACTIVE`, `DELETED`, `BANNED`).                                                                              |
| `userType`      | `Enum`       | No       | Query     | Filter users by their role (`DRIVER`, `PARKING_OWNER`, `ADMIN`).                                                                            |

**Response:**

- **200 OK**: Returns a paginated `Page<UserDto>` object with user details.
- **400 Bad Request**: Invalid search criteria provided.
- **401 Unauthorized**: The client is not authenticated.

**Example Usage:**

**Request:**
```http
GET /api/v1/users/search?email=johndoe@example.com&page=0&size=5&recordStatus=ACTIVE&userType=ADMIN
Authorization: Bearer <JWT_TOKEN>
```

**Response:**
```json
{
  "content": [
    {
      "id": "7bc4ae74-3443-4c53-a82a-8edab9100348",
      "email": "johndoe@example.com",
      "userType": "ADMIN",
      "status": "ACTIVE"
    }
  ],
  "totalPages": 1,
  "totalElements": 1,
  "number": 0,
  "size": 5
}
```

**Errors/Edge Cases:**

- 400 Bad Request: Invalid `page` or `size` values.
- 401 Unauthorized: Access without a valid JWT token.

---

### 2. Get User Details
#### **GET** `/api/v1/users/{userId}`

Retrieve details of a user by their unique ID.

**Request Parameters:**

| Parameter  | Type     | Required | Location | Description                         |
|------------|----------|----------|----------|-------------------------------------|
| `userId`   | `UUID`   | Yes      | Path     | The unique identifier of the user. |

**Response:**

- **200 OK**: Returns the `UserDto` with the user's details.
- **404 Not Found**: No user found with the provided `userId`.
- **401 Unauthorized**: The client is not authenticated.

**Example Usage:**

**Request:**
```http
GET /api/v1/users/7bc4ae74-3443-4c53-a82a-8edab9100348
Authorization: Bearer <JWT_TOKEN>
```

**Response:**
```json
{
  "id": "7bc4ae74-3443-4c53-a82a-8edab9100348",
  "email": "johndoe@example.com",
  "userType": "ADMIN",
  "status": "ACTIVE"
}
```

**Errors/Edge Cases:**

- 404 Not Found: User ID does not exist.
- 401 Unauthorized: Access without a valid JWT token.

---

### 3. Block a User
#### **PATCH** `/api/v1/users/{userId}/block`

Block a user by their unique ID. **Requires ADMIN privileges**.

**Request Parameters:**

| Parameter  | Type     | Required | Location | Description                              |
|------------|----------|----------|----------|------------------------------------------|
| `userId`   | `UUID`   | Yes      | Path     | The unique identifier of the user to block. |

**Response:**

- **200 OK**: Returns the updated `UserDto` object.
- **403 Forbidden**: The authenticated user does not have the necessary `ADMIN` privileges.
- **404 Not Found**: No user found with the provided `userId`.
- **401 Unauthorized**: The client is not authenticated.

**Example Usage:**

**Request:**
```http
PATCH /api/v1/users/7bc4ae74-3443-4c53-a82a-8edab9100348/block
Authorization: Bearer <JWT_TOKEN>
```

**Response:**
```json
{
  "id": "7bc4ae74-3443-4c53-a82a-8edab9100348",
  "email": "johndoe@example.com",
  "userType": "ADMIN",
  "status": "BANNED"
}
```

**Errors/Edge Cases:**

- 403 Forbidden: User without ADMIN privileges attempts this operation.
- 404 Not Found: User to block not found.
- 401 Unauthorized: Access without valid token.

---

### 4. Unblock a User
#### **PATCH** `/api/v1/users/{userId}/unblock`

Unblock a user by their unique ID. **Requires ADMIN privileges**.

**Request Parameters:**

| Parameter  | Type     | Required | Location | Description                                |
|------------|----------|----------|----------|--------------------------------------------|
| `userId`   | `UUID`   | Yes      | Path     | The unique identifier of the user to unblock. |

**Response:**

- **200 OK**: Returns the unblocked user's `UserDto` object.
- **403 Forbidden**: The authenticated user does not have the necessary `ADMIN` privileges.
- **404 Not Found**: User to unblock is not found.
- **401 Unauthorized**: User not authenticated.

**Example Usage:**

**Request:**
```http
PATCH /api/v1/users/7bc4ae74-3443-4c53-a82a-8edab9100348/unblock
Authorization: Bearer <JWT_TOKEN>
```

**Response:**
```json
{
  "id": "7bc4ae74-3443-4c53-a82a-8edab9100348",
  "email": "johndoe@example.com",
  "userType": "ADMIN",
  "status": "ACTIVE"
}
```

---

### 5. Change Password
#### **PUT** `/api/v1/users`

Allows a user to change their password.

**Request Body:**

| Field          | Type     | Required | Description                                |
|-----------------|----------|----------|--------------------------------------------|
| `oldPassword`  | `String` | Yes      | The user's current password.              |
| `newPassword`  | `String` | Yes      | The new password to be set.               |

**Response:**

- **200 OK**: Password was successfully changed.
- **400 Bad Request**: Validation errors such as invalid passwords.
- **401 Unauthorized**: Access without valid JWT token.

**Example Usage:**

**Request:**
```http
PUT /api/v1/users
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "oldPassword": "currentPass123",
  "newPassword": "newPass456"
}
```

---

## Authentication and Security

All endpoints require a valid JWT token passed in the `Authorization` header as a bearer token. Some endpoints are restricted to users with `ADMIN` roles.

--- 