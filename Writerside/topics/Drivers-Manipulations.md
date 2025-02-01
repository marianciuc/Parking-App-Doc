# Drivers Manipulations

The `DriverController` is a REST controller that handles operations related to drivers within the "Drivers Management
Service". This controller is responsible for managing drivers' data, including retrieval, search, and updates. It
exposes endpoints prefixed with `/api/v1/drivers`, aligning with RESTful design standards and versioning practices.
Key responsibilities:

- Retrieve driver details by their unique identifier (`userId`).
- Search drivers based on filtering criteria, with pagination and sorting support.
- Retrieve authenticated driver profile details for the currently logged-in user.
- Update driver details using their unique identifier (`userId`).

## Endpoints Overview

### 1. **`GET /api/v1/drivers/{userId}`**

**Retrieve Driver by ID**

#### Description: {id="description_1"}

Fetches the details of the driver uniquely identified by the provided `userId`.

#### Request Details: {id="request-details_1"}

- **HTTP Method**: `GET`
- **Path Variable**:
    - `userId` (UUID): The unique identifier of the driver to be retrieved.

#### Access Security: {id="access-security_1"}

General access – No specific authentication mechanisms enforced on this endpoint.

#### Response Details: {id="response-details_1"}

- **Response Code**:
    - `200 OK` – Driver found.
    - `404 Not Found` – Driver with the given `userId` does not exist.

- **Response Body**: A JSON representation of the `DriverDto`.

#### Example Request: {id="example-request_1"}

``` http
GET /api/v1/drivers/e8e70633-e434-48b0-8b0c-9e64c1f50114 HTTP/1.1
Host: api.example.com
```

#### Example Response: {id="example-response_1"}

``` json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "userId": "e8e70633-e434-48b0-8b0c-9e64c1f50114",
  "profilePicture": "https://example.com/images/user.jpg",
  "firstName": "John",
  "lastName": "Doe",
  "country": "Poland",
  "city": "Gdańsk",
  "email": "johndoe@example.com",
  "phoneNumber": "+48123456789",
  "phoneCode": "+48",
  "birthDate": "1990-01-01",
  "isEmailVerified": true,
  "isPhoneVerified": true,
  "creationDate": "2023-01-01T12:00:00",
  "modificationDate": "2023-02-01T15:30:00"
}
```

#### Possible Errors: {id="possible-errors_1"}

- `404 Not Found`: The driver does not exist for the given `userId`.
- `400 Bad Request`: The provided `userId` is malformed.

### 2. **`GET /api/v1/drivers/search`**

**Search Drivers**

#### Description: {id="description_2"}

Searches for drivers based on filtering criteria (e.g., city, email, etc.) with support for pagination and sorting.

#### Request Details: {id="request-details_2"}

- **HTTP Method**: `GET`
- **Request Parameters**:
    - `size` (Integer, optional): Number of results per page (default: `10`).
    - `page` (Integer, optional): Zero-based page index (default: `0`).
    - `sort` (String, optional): Sorting order (`asc` or `desc`, default: `asc`).
    - `sortBy` (String, optional): Sorting field (default: `id`).

- **Request Body**:
    - `DriverFilters` (JSON): Filtering criteria for driver attributes.
        - Supported Filter Fields: `firstName`, `lastName`, `country`, `city`, `email`, `phoneNumber`, `phoneCode`,
          `userId`.

#### Access Security: {id="access-security_2"}

General access – No specific authentication mechanisms enforced.

#### Response Details: {id="response-details_2"}

- **Response Code**:
    - `200 OK` – A page of matching drivers found.
    - `400 Bad Request` – Invalid request parameters or filter structure.

- **Response Body**: A paginated `Page<DriverDto>` representation.

#### Example Request: {id="example-request_2"}

``` http
POST /api/v1/drivers/search?page=0&size=5&sort=asc&sortBy=lastName HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "city": "Gdańsk",
  "email": "johndoe@example.com"
}
```

#### Example Response: {id="example-response_2"}

``` json
{
  "content": [
    {
      "id": "123e4567-e89b-12d3-a456-426614174000",
      "userId": "e8e70633-e434-48b0-8b0c-9e64c1f50114",
      "firstName": "John",
      "lastName": "Doe"
    },
    {
      "id": "123e4567-e89b-12d3-a456-426614174001",
      "userId": "b8e70633-e434-48b0-8b0c-9e64c1f50115",
      "firstName": "Jane",
      "lastName": "Smith"
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 5
  },
  "totalElements": 10,
  "totalPages": 2
}
```

#### Possible Errors: {id="possible-errors_2"}

- `400 Bad Request`: Malformed filter structure or invalid pagination/sorting parameters.

### 3. **`GET /api/v1/drivers`**

**Get Authenticated Driver**

#### Description: {id="description_3"}

Retrieves the profile of the currently authenticated driver.

#### Request Details: {id="request-details_3"}

- **HTTP Method**: `GET`
- **Authentication**: Required. User must have the `DRIVER` role.

#### Access Security:

- **Pre-Authorization**: Annotated with `@PreAuthorize("hasRole('DRIVER')")`.

#### Response Details: {id="response-details_3"}

- **Response Code**:
    - `200 OK` – Successfully retrieved authenticated driver profile.
    - `403 Forbidden` – User does not have the appropriate role.

- **Response Body**: `DriverDto` object corresponding to the authenticated driver.

#### Example Request: {id="example-request_3"}

``` http
GET /api/v1/drivers HTTP/1.1
Host: api.example.com
Authorization: Bearer <JWT-Token>
```

#### Example Response: {id="example-response_3"}

``` json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "userId": "e8e70633-e434-48b0-8b0c-9e64c1f50114",
  "firstName": "John",
  "lastName": "Doe",
  ...
}
```

#### Possible Errors: {id="possible-errors_3"}

- `403 Forbidden`: Not authenticated or missing `DRIVER` role.

### 4. **`PATCH /api/v1/drivers/{userId}`**

**Update Driver**

#### Description:

Updates the details of the driver identified by the given `userId`.

#### Request Details:

- **HTTP Method**: `PATCH`
- **Path Variable**:
    - `userId` (UUID): The identifier of the driver to be updated.

- **Request Body**:
    - `DriverDto` (JSON): Contains updated driver details.

#### Response Details:

- **Response Code**:
    - `200 OK` – Successfully updated the driver details.
    - `404 Not Found` – Driver not found.

- **Response Body**: The updated `DriverDto`.

#### Example Request:

``` http
PATCH /api/v1/drivers/e8e70633-e434-48b0-8b0c-9e64c1f50114 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "firstName": "John Updated",
  "lastName": "Doe Updated",
  ...
}
```

#### Example Response:

``` json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "userId": "e8e70633-e434-48b0-8b0c-9e64c1f50114",
  "firstName": "John Updated",
  "lastName": "Doe Updated",
  ...
}
```

#### Possible Errors:

- `404 Not Found`: Driver not found for the given `userId`.

### Common Error Responses

- `401 Unauthorized`: User is not authenticated.
- `403 Forbidden`: Authenticated user lacks required permissions.
- `400 Bad Request`: Invalid input parameters.
