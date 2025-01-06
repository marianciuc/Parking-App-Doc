# API Keys Management Service
## Comprehensive Documentation for `ApiKey` REST Controller

The `ApiKey` REST Controller is designed to manage **CRUD operations** and additional functionalities for handling API keys in the **Parking Management System**. These keys are critical for authentication and must be included in all client requests using a custom header: `X-Api-Key: {apiKeyValue}`.

The controller interacts with the `ApiKey` entity and provides the ability to **create**, **retrieve**, **update**, **revoke**, and **delete API keys**. Only users with **administrator privileges** can generate API keys. Below, you will find all the necessary specifications and detailed information for implementing and using the available endpoints.

---

## **Endpoint Specifications**

### 1. **Create a New API Key**
- **URL**: `/api/v1/api-keys/generate/{parkingId}`
- **Method**: `POST`
- **Description**: Allows administrators to create a new API key.
- **Path Parameters**:
    - `parkingId`: UUID of the parkingId to associate new key.
    - **Key Format**:
        - `keyValue`: At least 16 characters long, containing letters and digits.
- **Request Body (JSON)**:
  ```json
  {
    "scope": ["SCOPE_1", "SCOPE_2"]
  }
  ```
- **Response (201 - Created)**:
  ```json
  {
    "id": "UUID of the API key",
    "keyValue": "unique-key-value",
    "parkingId": "UUID of the parking",
    "scope": ["SCOPE_1", "SCOPE_2"],
    "issuedBy": "UUID of the issuer",
    "revokedBy": null,
    "status": "ACTIVE"
  }
  ```
- **Error Responses**:
    - **400 Bad Request**: Invalid request body structure or missing required fields.
---

### 2. **Fetch API Key Details**
- **URL**: `/api/v1/api-keys/{id}`
- **Method**: `GET`
- **Description**: Retrieves the details of a specific API key using its unique ID.
- **Path Parameters**:
    - `id`: UUID of the API key to fetch information for.
- **Response (200 - Success)**:
  ```json
  {
    "id": "UUID of the API key",
    "keyValue": "unique-key-value",
    "parkingId": "UUID of the parking",
    "scope": ["SCOPE_1", "SCOPE_2"],
    "issuedBy": "UUID of the issuer",
    "revokedBy": null,
    "status": "ACTIVE"
  }
  ```
- **Error Responses**:
    - **404 Not Found**: API key with the specified ID does not exist.

---

### 3. **List All API Keys**
- **URL**: `/api/v1/api-keys/search`
- **Method**: `GET`
- **Description**: Fetches a paginated list of all registered API keys with optional filters.
- **Query Parameters**:
    - `status` (Optional): Filter results by status (`ACTIVE`, `REVOKED`).
    - `parkingId` (Optional): Filtering by parking id
    - **Filters Logic**: All specified filters are applied using `AND` logic.
    - `page` (Optional): Page number for pagination (default: `1`).
    - `size` (Optional): Number of results per page (default: `10`).
- **Response (200 - Success)**:
  ```json
  {
    "content": [
      {
        "id": "UUID of the API key",
        "parkingId": "UUID of the parking",
        "scope": ["SCOPE_1", "SCOPE_2"],
        "status": "ACTIVE"
      }
    ],
    "page": 1,
    "size": 10,
    "totalElements": 50
  }
  ```

---

### 4. **Revoke an API Key**
- **URL**: `/api/v1/api-keys/{id}/revoke`
- **Method**: `PUT`
- **Description**: Revokes an active API key, marking it as no longer valid.
- **Path Parameters**:
    - `id`: UUID of the API key to be revoked.
- **Request Body (JSON)**:
  ```json
  {
    "revokedBy": "UUID of the current session user (filled automatically)"
  }
  ```
- **Response (200 - Success)**:
  ```json
  {
    "id": "UUID of the API key",
    "keyValue": "unique-key-value",
    "parkingId": "UUID of the parking",
    "scope": ["SCOPE_1", "SCOPE_2"],
    "issuedBy": "UUID of the issuer",
    "revokedBy": "UUID of revoker",
    "status": "REVOKED"
  }
  ```
- **Error Responses**:
    - **400 Bad Request**: API key is already revoked.
    - **404 Not Found**: API key with the specified ID does not exist.

---

### 5. **Delete an API Key**
- **URL**: `/api/v1/api-keys/{id}`
- **Method**: `DELETE`
- **Description**: Permanently deletes the specified API key from the system.
- **Path Parameters**:
    - `id`: UUID of the API key to be deleted.
- **Response (204 - No Content)**:
    - No content provided on success.
- **Error Responses**:
    - **404 Not Found**: API key with the specified ID does not exist.
---

### 6. **Validate API Key**
- **URL**: `/api/v1/api-keys/validate/{keyValue}`
- **Method**: `GET`
- **Description**: Validates the provided API key and retrieves its details if valid.
- **Path Parameters**:
    - `keyValue`: API key to be validated.
- **Response (200 - Success)**:
  ```json
  {
    "id": "UUID of the API key",
    "keyValue": "unique-key-value",
    "parkingId": "UUID of the parking",
    "scope": ["SCOPE_1", "SCOPE_2"],
    "issuedBy": "UUID of the issuer",
    "revokedBy": null,
    "status": "ACTIVE"
  }
  ```
- **Error Responses**:
    - **404 Not Found**: API key with the specified ID does not exist.
---

## **Request Validation Rules**
1. **Key Uniqueness**: The `keyValue` field must be globally unique.
2. **Valid UUID Format**:
    - Fields such as `parkingId`, `issuedBy`, and `revokedBy` must conform to the UUID standard.
    - `revokedBy` field is populated automatically with the session user's UUID; cannot be set manually.
3. **Field Constraints**:
    - `scope` field must not be empty and must include at least one valid permission.
    - Valid values for `scope`: `['SCOPE_1', 'SCOPE_2']`. The list of permissions is synchronized with the system's database.
    - `status` must be one of the accepted values: `ACTIVE`, `INACTIVE`, or `REVOKED`.

---

## **Controller Class Outline**
The `ApiKeyController` class implements the following endpoints:
- `@PostMapping("/api/v1/api-keys/generate/{parkingId}")`: Create a new API key.
- `@GetMapping("/api-keys/search")`: List all API keys (paginated).
- `@PutMapping("/api-keys/{id}/revoke")`: Revoke an API key by ID.
- `@DeleteMapping("/api-keys/{id}")`: Delete an existing API key by ID.
- `@DeleteMapping("/api-keys/validate/{keyValue}")`: Validate an API key

---

## **Example Use Case**
### Revoking an API Key
1. **Client Action**:
    - The administrator sends a request to revoke an API key with ID `1234`.
2. **System Validation**:
    - Confirms the key exists and its current status is `ACTIVE`.
3. **Outcome**:
    - API key's status is updated to `REVOKED`.
    - The `revokedBy` field is populated automatically with the session user's UUID; cannot be set manually.

Example Request:
```http
PUT /api/v1/api-keys/1234/revoke
Content-Type: application/json
```

Example Response:
```json
{
  "id": "1234",
  "keyValue": "unique-key-value",
  "parkingId": "UUID of the parking",
  "scope": ["SCOPE_1", "SCOPE_2"],
  "issuedBy": "UUID of the issuer",
  "revokedBy": "admin-uuid",
  "status": "REVOKED"
}
```

---

## **Security and Logging**
- This API uses secure authentication mechanisms such as tokens or API key validation.
- All sensitive operations (creation, revocation, deletion) should be logged for auditing purposes.

**Notes**:
1. Proper error handling ensures robust fault tolerance.
2. This API design is compliant with REST principles for scalability and maintainability.