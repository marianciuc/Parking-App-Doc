# Vehicle Ownership Management

The **OwnershipController** is a REST API controller for managing vehicle ownership-related operations in the system. It handles all interactions related to transferring and managing vehicle ownerships between users.
This controller provides endpoints under the base URL: `/api/v1/vehicles/{vehicleId}/ownership`.
### **Responsibilities**
- Initiates and manages ownership transfer requests.
- Approves or rejects ownership transfer requests.
- Manages secure ownership transfers with restricted access to specific roles (e.g., admin only).

## REST API Endpoints
### 1. **Request Ownership Transfer**
#### **Endpoint** {id="endpoint_1"}
``` http
POST /api/v1/vehicles/{vehicleId}/ownership/transfer/request
```
#### **Description** {id="description_1"}
This endpoint allows a user to initiate a request to transfer the ownership of a vehicle specified by its `vehicleId`.
#### **Request Details** {id="request-details_1"}
- **Path Parameter**:
    - `vehicleId` (UUID): The ID of the vehicle for which the ownership transfer request is initiated.

- **Request Body**:
    - **Type**: `OwnershipTransferRequest`
        - Currently, this object does not have any fields, but it is reserved for extending future requirements.

#### **Response** {id="response_1"}
- **Status**: `200 OK`
- **Body**: A `Boolean` value indicating whether the ownership transfer request was successfully initiated.

#### **Authentication** {id="authentication_1"}
No role restrictions for this endpoint. Any authenticated user can access it.
#### **Example Request** {id="example-request_1"}
``` http
POST /api/v1/vehicles/123e4567-e89b-12d3-a456-426614174000/ownership/transfer/request
Content-Type: application/json
```
#### **Example Response** {id="example-response_1"}
``` json
true
```
#### **Error Cases** {id="error-cases_1"}
- `404 Not Found`: If no vehicle with the specified `vehicleId` exists.

### 2. **Perform Ownership Transfer**
#### **Endpoint** {id="endpoint_2"}
``` http
POST /api/v1/vehicles/{vehicleId}/ownership/transfer
```
#### **Description** {id="description_2"}
Transfers the ownership of a vehicle to a new owner. This is restricted to users with the `ADMIN` role.
#### **Request Details** {id="request-details_2"}
- **Path Parameter**:
    - `vehicleId` (UUID): The ID of the vehicle whose ownership needs to be transferred.

- **Query Parameters**:
    - `newOwnerId` (UUID): The unique ID of the new owner.
    - `ownerType` (String): Type of the new owner (`PERSON` or `FAMILY`).

#### **Response** {id="response_2"}
- **Status**: `200 OK`
- **Body**: A `Boolean` value indicating whether the transfer was successfully completed.

#### **Authentication** {id="authentication_2"}
- `@PreAuthorize("hasRole('ADMIN')")`: Only users with the `ADMIN` role can access this endpoint.

#### **Example Request** {id="example-request_2"}
``` http
POST /api/v1/vehicles/123e4567-e89b-12d3-a456-426614174000/ownership/transfer?newOwnerId=456b789c-e89b-56d3-b456-426614174111&ownerType=PERSON
Content-Type: application/json
```
#### **Example Response** {id="example-response_2"}
``` json
true
```
#### **Error Cases** {id="error-cases_2"}
- `400 Bad Request`: If the `ownerType` is invalid or missing.
- `404 Not Found`: If no vehicle with the given `vehicleId` exists.
- `403 Forbidden`: If the user does not have the required `ADMIN` role.

### 3. **Accept Ownership Transfer Request**
#### **Endpoint** {id="endpoint_3"}
``` http
PUT /api/v1/vehicles/{vehicleId}/ownership/{requestId}/accept
```
#### **Description** {id="description_3"}
Accepts a pending ownership transfer request for a specific vehicle.
#### **Request Details** {id="request-details_3"}
- **Path Parameters**:
    - `vehicleId` (UUID): The ID of the vehicle involved in the request.
    - `requestId` (UUID): The unique ID of the ownership transfer request to be accepted.

#### **Response** {id="response_3"}
- **Status**: `200 OK`
- **Body**: A `Boolean` value indicating whether the approval was successful.

#### **Authentication** {id="authentication_3"}
No specific permissions required, but the user must own the vehicle or be authorized to manage its ownership transfers.
#### **Example Request** {id="example-request_3"}
``` http
PUT /api/v1/vehicles/123e4567-e89b-12d3-a456-426614174000/ownership/456f7890-e89b-56d3-b456-426614174111/accept
Content-Type: application/json
```
#### **Example Response** {id="example-response_3"}
``` json
true
```
#### **Error Cases** {id="error-cases_3"}
- `404 Not Found`: If no ownership request with the specified `requestId` exists for the vehicle.
- `409 Conflict`: If the request is already accepted/rejected.

### 4. **Reject Ownership Transfer Request**
#### **Endpoint**
``` http
PUT /api/v1/vehicles/{vehicleId}/ownership/{requestId}/reject
```
#### **Description**
Rejects a pending ownership transfer request for a specific vehicle.
#### **Request Details**
- **Path Parameters**:
    - `vehicleId` (UUID): The ID of the vehicle involved in the request.
    - `requestId` (UUID): The unique ID of the ownership transfer request to be rejected.

#### **Response**
- **Status**: `200 OK`
- **Body**: A `Boolean` value indicating whether the rejection was successful.

#### **Authentication**
No specific permissions required, but the user must own the vehicle or be authorized to manage its ownership transfers.
#### **Example Request**
``` http
PUT /api/v1/vehicles/123e4567-e89b-12d3-a456-426614174000/ownership/456f7890-e89b-56d3-b456-426614174111/reject
Content-Type: application/json
```
#### **Example Response**
``` json
true
```
#### **Error Cases**
- `404 Not Found`: If no ownership request with the specified `requestId` exists for the vehicle.
- `409 Conflict`: If the request was already accepted/rejected.

## General Error Responses
- **`400 Bad Request`**: If any required parameter is missing or invalid.
- **`403 Forbidden`**: If the user's role or access level does not permit the requested operation.
- **`404 Not Found`**: If the specified `vehicleId` or `requestId` does not exist in the system.
- **`500 Internal Server Error`**: If an unexpected error occurs during processing.
