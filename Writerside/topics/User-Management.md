# Registration and Refresh Token

`AuthController` is responsible for handling authentication-related operations in the parking system. It provides
endpoints for registering users of specific types (e.g., DRIVER, ADMIN) and refreshing JSON Web Tokens (JWTs) for
authentication and session management.

### Base URL

All endpoints of this controller are versioned under `/api/v1/security`. Versioning ensures backward compatibility as
the system evolves. The controller's role is critical in managing user accounts and token-based authentication flows.

## Endpoints

### **1. Register a New User**

#### **HTTP Method and Endpoint** {id="http-method-and-endpoint_1"}

- **POST** `/api/v1/security/register/{userType}`

#### **Functionality** {id="functionality_1"}

This endpoint is used to register a new user of a specific type (e.g., DRIVER, ADMIN, PARKING_OWNER). It accepts user
registration details (e.g., email, password) and issues authentication tokens upon successful registration.

#### **Request Details** {id="request-details_1"}

- **Path Variable**:
    - `userType` (required):
        - **Type**: `String` (`UserType` Enum)
        - **Description**: Specifies the type of user to register. Supported values are:
            - `DRIVER`: Individual seeking parking spaces.
            - `PARKING_OWNER`: Owner of parking facilities.
            - `ORGANIZATION_PERSONAL`: Operational personnel.
            - `ADMIN`: System administrator.

        - **Example**: `"DRIVER"`

- **Request Body** (JSON):
    - **Type**: `UserRegistrationRequest`
    - **Fields**:
        - `email` (required):
            - **Type**: `String`
            - **Description**: The email address of the user.
            - **Validation Rules**: Must be a valid email.
            - **Example**: `"user@example.com"`

        - `password` (required):
            - **Type**: `String`
            - **Description**: Password for the user account.
            - **Validation Rules**:
                - At least 6 characters long.
                - Must include one uppercase letter, one lowercase letter, one digit, and one special character.

            - **Example**: `"Password123!"`

#### **Response Details** {id="response-details_1"}

- **Status Codes**:
    - `200 OK`: User successfully registered. Returns authentication tokens.
    - `400 Bad Request`: Invalid request body or invalid `userType`.
    - `401 Unauthorized`: Authorization issue.

- **Response Body** (JSON):
    - **Type**: `AuthenticationTokens`
    - **Fields**:
        - `accessToken`:
            - **Type**: `String`
            - **Description**: The issued access token for authenticated user actions.

        - `accessTokenExpiry`:
            - **Type**: `String`
            - **Description**: Expiry timestamp of the access token.

        - `refreshToken`:
            - **Type**: `String`
            - **Description**: Token used to refresh expired access tokens.

        - `refreshTokenExpiry`:
            - **Type**: `String`
            - **Description**: Expiry timestamp of the refresh token.

#### **Example Usage** {id="example-usage_1"}

**Request**:

``` http
POST /api/v1/security/register/DRIVER HTTP/1.1
Content-Type: application/json

{
  "email": "driver@example.com",
  "password": "Password123!"
}
```

**Response**:

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cC...",
  "accessTokenExpiry": "2023-12-31T23:59:59",
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4=",
  "refreshTokenExpiry": "2024-01-07T23:59:59"
}
```

#### **Errors/Edge Cases**: {id="errors-edge-cases_1"}

- **400 Bad Request**:
    - Missing or invalid fields in the request body (e.g., `password` length too short).
    - Invalid `userType`.

- **401 Unauthorized**:
    - Client lacks valid authorization details for registration.

### **2. Refresh JWT Tokens**

#### **HTTP Method and Endpoint**

- **POST** `/api/v1/security/jwt/refresh`

#### **Functionality**

This endpoint allows authenticated users to refresh their JWT tokens. It validates the existing refresh token and issues
new authentication and refresh tokens.

#### **Request Details**

- **Headers**:
    - `Authorization`:
        - **Type**: `Bearer Token`
        - **Description**: The refresh token of the user.

- **Precondition**:
    - User must have the role `ROLE_REFRESH_TOKEN`.

- **Authentication**:
    - This endpoint requires the user to be authenticated.

#### **Response Details**

- **Status Codes**:
    - `200 OK`: Tokens successfully refreshed.
    - `403 Forbidden`: User does not have the `ROLE_REFRESH_TOKEN` role.
    - `401 Unauthorized`: Invalid or missing token.

- **Response Body** (JSON):
    - **Type**: `AuthenticationTokens`
    - **Fields**:
        - `accessToken`:
            - **Type**: `String`
            - **Description**: The new access token issued.

        - `accessTokenExpiry`:
            - **Type**: `String`
            - **Description**: Expiry timestamp of the new access token.

        - `refreshToken`:
            - **Type**: `String`
            - **Description**: The new refresh token issued.

        - `refreshTokenExpiry`:
            - **Type**: `String`
            - **Description**: Expiry timestamp of the new refresh token.

#### **Example Usage**

**Request**:

``` http
POST /api/v1/security/jwt/refresh HTTP/1.1
Authorization: Bearer <refresh_token>
Content-Type: application/json
```

**Response**:

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cC...",
  "accessTokenExpiry": "2023-12-31T23:59:59",
  "refreshToken": "dGhpcyBpcyBhbm90aGVyIHJlZnJlc2ggdG9rZW4=",
  "refreshTokenExpiry": "2024-01-14T23:59:59"
}
```

#### **Errors/Edge Cases**:

- **401 Unauthorized**:
    - Missing or invalid `Bearer Token` in the `Authorization` header.

- **403 Forbidden**:
    - User is authenticated but does not have the `ROLE_REFRESH_TOKEN`.

## Common Response Structure

Both endpoints follow the standard Spring `ResponseEntity` structure:

- **Status Code**: Indicates success or failure of the operation.
- **Headers**: Additional response metadata (e.g., authorization).
- **Body**: Primary data content (e.g., authentication tokens or error message).

### Notes

- **Security**:
    - Authentication mechanisms like `Bearer Token` and `ROLE_REFRESH_TOKEN` ensure secure operation.

- **Validation**:
    - Field-level validation is implemented using annotations like `@NotEmpty`, `@Size`, and `@Pattern`.
