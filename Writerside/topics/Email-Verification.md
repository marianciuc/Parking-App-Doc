# Email Verification

## Overview

The `EmailVerificationController` class handles all functionalities related to email verification for drivers in the
system. It provides a RESTful interface for sending email verification codes to drivers and verifying those codes. This
controller is exposed under the versioned API path `/api/v1/drivers/email-verification`.

### Responsibilities:

1. **Sending Email Verification Code**: Allows authorized drivers to request an email verification code to be sent to
   their registered email.
2. **Verifying Email**: Provides a mechanism for drivers to verify their email addresses using the code sent.

### API Versioning:

- **Base Path**: `/api/v1`
- **Email Verification Endpoint Path**: `/drivers/email-verification`

## REST Endpoints

### 1. **Verify Email**

#### **Endpoint** {id="endpoint_1"}

``` http
PATCH /api/v1/drivers/email-verification
```

#### **Description** {id="description_1"}

This endpoint verifies an email address using a unique verification code. The code is provided by the driver as a query
parameter.

#### **HTTP Method** {id="http-method_1"}

`PATCH`

#### **Request Parameters**

- **`code`** (`String`, required)
  A unique verification code sent to the user's email. This code is used to verify the email address.

#### **Authorization** {id="authorization_1"}

- Drivers must be authenticated to perform this action.
- Uses the `SecurityUtils.getAuthenticatedUserId()` to validate the authenticated driver.

#### **Response** {id="response_1"}

- **Success**:
    - **Status Code**: `200 OK`
    - **Body**: No content (`void`)

- **Error Scenarios**:
    - **`400 Bad Request`**: If the provided verification code is invalid or expired.
    - **`404 Not Found`**: If the driver associated with the code cannot be found.
    - **`409 Conflict`**: If the email has already been verified.

#### **Example Usage** {id="example-usage_1"}

**Request:**

``` http
PATCH /api/v1/drivers/email-verification?code=12345
```

**Response (Success):**

``` http
HTTP/1.1 200 OK
```

**Response (Error - Invalid Validation Code):**

``` Bash
HTTP/1.1 400 Bad Request
{
  "error": "InvalidValidationCode",
  "message": "The provided validation code is invalid or has expired."
}
```

### 2. **Send Email Verification Code**

#### **Endpoint**

``` http
POST /api/v1/drivers/email-verification
```

#### **Description**

This endpoint sends a unique email verification code to the email address of the currently authenticated driver.

#### **HTTP Method**

`POST`

#### **Request Body**

No request body is required.

#### **Authorization**

- Drivers must be authenticated to perform this action.
- Uses `SecurityUtils.getAuthenticatedUserId()` to identify the currently logged-in driver.

#### **Response**

- **Success**:
    - **Status Code**: `200 OK`
    - **Body**: No content (`void`)

- **Error Scenarios**:
    - **`403 Forbidden`**: If the user is not authenticated or lacks permissions.
    - **`404 Not Found`**: If the driver associated with the authenticated ID does not exist.
    - **`409 Conflict`**: If the email has already been verified.

#### **Example Usage**

**Request:**

``` http
POST /api/v1/drivers/email-verification
Authorization: Bearer <valid-token>
```

**Response (Success):**

``` http
HTTP/1.1 200 OK
```

**Response (Error - Email Already Verified):**

``` Bash
HTTP/1.1 409 Conflict
{
  "error": "EmailAlreadyVerified",
  "message": "The email address has already been verified."
}
```

## Response Details

Both endpoints return a `ResponseEntity<Void>` object:

- **200 OK**: Indicates the action was completed successfully. There is no body returned.
- **Error Responses**: Return standard HTTP error statuses along with an error response in JSON format.

Example error response:

``` json
{
  "error": "<ErrorCode>",
  "message": "<Detailed error message>"
}
```

## Possible Errors

Below is an exhaustive list of possible exceptions for each endpoint and their associated HTTP status codes.

### **Verify Email**

- `400 Bad Request` - The provided verification code is invalid or expired.
- `404 Not Found` - The driver associated with the verification code cannot be found.
- `409 Conflict` - The email address is already verified.

### **Send Email Verification Code**

- `403 Forbidden` - The user is not authenticated or lacks permissions.
- `404 Not Found` - The authenticated driver does not exist in the system.
- `409 Conflict` - The email address is already verified.

## Example Scenarios

### **1. Verifying an Email (Happy Path)**

**Request:**

``` http
PATCH /api/v1/drivers/email-verification?code=abc123
```

**Response:**

``` http
HTTP/1.1 200 OK
```

### **2. Sending Email Verification Code**

**Request:**

``` http
POST /api/v1/drivers/email-verification
Authorization: Bearer <valid-auth-token>
```

**Response:**

``` http
HTTP/1.1 200 OK
```

### **3. Edge Case - Invalid Code**

**Request:**

``` http
PATCH /api/v1/drivers/email-verification?code=invalid_code
```

**Response:**

``` BASH
HTTP/1.1 400 Bad Request
{
  "error": "InvalidValidationCode",
  "message": "The provided validation code is invalid or has expired."
}
```

This documentation adheres to RESTful best practices and is structured for easy integration into tools like Swagger or
included in project README files for technical reference.
