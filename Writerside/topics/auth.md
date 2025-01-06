# Authorization and Authentication System Documentation

This document provides comprehensive guidelines for frontend developers on how to implement authorization and authentication in the system using REST APIs. The system relies on **JWT (JSON Web Tokens)** for secure communication. This guide outlines the authorization workflow, available API endpoints, error handling, and example usage to help you integrate seamlessly into the system.

---

## **Overview**

The authorization system is designed to securely manage user authentication and protect access to resources using tokens. It supports:
- **Credentials-based authentication**: Login using username and password.
- **JWT-based authorization**: Protect API requests using an access token (short-lived).
- **Token refreshing**: Obtain a new access token when the old one expires without re-authentication.

### **Authorization Workflow**
1. **Authentication**: A user logs in with their credentials and receives two tokens:
  - **Access Token**: Used for accessing protected resources.
  - **Refresh Token**: Used to generate a new access token when the old one expires.
2. **Securing Requests**: The access token is sent in the `Authorization` header for all secure API calls.
3. **Refreshing Tokens**: If an access token expires, the refresh token is used to obtain a new access token without requiring the user to log in again.

---

## **API Endpoints**

The following API endpoints define the authentication and authorization flow within the system.

### **1. Login: Token Retrieval**

Authenticate the user and retrieve tokens for future requests.

- **Endpoint**: `/api/v1/security/login`
- **Method**: `POST`

#### **Request** (Login):
- **Headers**: None
- **Body**:
```json
{
  "login": "<string>",
  "password": "<string>"
}
```

#### **Login Response**:
- **200 OK** (Success):
```json
{
  "accessToken": "<string>",
  "accessTokenExpiry": "<datetime>",
  "refreshToken": "<string>",
  "refreshTokenExpiry": "<datetime>"
}
```
- **401 Unauthorized** (Failure): If provided credentials are invalid.

---

### **2. Access Protected Resources**

Access secured endpoints by including the access token in the request header.

- **Endpoint**: Varies based on the specific resource (e.g., `/api/secure/resource`)
- **Method**: Depends on the requested resource (e.g., `GET`, `POST`, etc.)

#### **Request** (Protected Resource):
- **Headers**:
```http
Authorization: Bearer <accessToken>
```
- **Body** (if required by the resource):
```json
{
  "examplePayloadField": "<value>"
}
```

#### **Protected Resource Response**:
- **200 OK** (Success): Returns the requested resource.
- **Error Responses**:
  - **401 Unauthorized**: Invalid or expired access token.
  - **403 Forbidden**: User lacks the required permissions to access the resource.

---

### **3. Token Refresh**

Obtain a new access token using the refresh token.

- **Endpoint**: `/api/v1/security/jwt/refresh-token`
- **Method**: `POST`

#### **Request** (Token Refresh):
- **Headers**:
```http
Authorization: Bearer <refreshToken>
```
- **Body**: None

#### **Refresh Token Response**:
- **200 OK** (Success):
```json
{
  "accessToken": "<string>",
  "accessTokenExpiry": "<datetime>",
  "refreshToken": "<string>",
  "refreshTokenExpiry": "<datetime>"
}
```
- **401 Unauthorized** (Failure): If the refresh token is invalid or expired.

---

## **Detailed Authorization Workflow**

### **1. Login: Authorization Process**
1. The user sends a `POST` request to `/api/v1/security/login` with their username and password.
2. The system validates the credentials and returns:
  - An **access token**: Used for accessing resources.
  - A **refresh token**: For generating new access tokens when the current one expires.

### **2. Securing Requests**
1. Add the **access token** to the `Authorization` header of every request to secured resources as follows:
   ```http
   Authorization: Bearer <accessToken>
   ```
2. Provide any additional payload required by the specific endpoint.

### **3. Refreshing Tokens**
1. If the system returns a `401 Unauthorized` error due to an expired access token:
  - Send a `POST` request to `/api/v1/security/jwt/refresh-token` with the **refresh token** in the `Authorization` header.
2. Upon success, use the newly issued access token for subsequent requests.
3. If the refresh token is invalid or expired, prompt the user to log in again.

---

## **Example Usage Flow**

### **1. Login to Get Tokens**

**Request**:
```bash
POST /api/v1/security/login
Content-Type: application/json

{
  "login": "exampleUser",
  "password": "password123"
}
```

**Response**:
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cC...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cC...",
  "accessTokenExpiry": "2025-01-04T19:16:07.298428Z",
  "refreshTokenExpiry": "2025-01-05T19:11:07.298013Z"
}
```

---

### **2. Use Access Token to Fetch a Protected Resource**

**Request**:
```bash
GET /api/secure/resource
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5c...
```

**Response**:
```json
{
  "data": "Some protected resource"
}
```

---

### **3. Refresh Expired Token**

**Request**:
```bash
POST /api/v1/security/jwt/refresh-token
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5c...
```

**Response**:
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cC...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cC...",
  "accessTokenExpiry": "2025-01-04T19:16:07.298428Z",
  "refreshTokenExpiry": "2025-01-05T19:11:07.298013Z"
}
```

---

## **Error Handling**

### **Common Errors**

| **Error Code**       | **Description**                                                     |
|----------------------|---------------------------------------------------------------------|
| **401 Unauthorized** | The token is invalid, expired, or missing.                          |
| **403 Forbidden**    | The user does not have permission to access the requested resource. |

### **Resolving Errors**
1. **401 Unauthorized**:
  - Verify that the access token is valid and included in the `Authorization` header.
  - If the token is expired, refresh it using the `/api/v1/security/jwt/refresh-token` endpoint.
2. **403 Forbidden**:
  - Ensure that the user has sufficient permissions for the resource.

---

This documentation ensures that frontend developers can effectively authenticate and authorize users, protect sensitive endpoints, and handle common error scenarios when integrating with the system.