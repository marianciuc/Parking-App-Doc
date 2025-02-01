# Parking Owners Management

The `OwnerController` is a REST controller in the Parking Owners Management System responsible for managing operations
related to parking owners. These operations include checking permissions, retrieving owner information, and updating
owner details. All endpoints are prefixed with **`/api/v1/owners` **, following the system's versioning conventions.

## Endpoints Overview

### **1. Check Owner Permissions**

#### **Endpoint** {id="endpoint_1"}

`GET /api/v1/owners/check-permission`

#### **Description** {id="description_1"}

Validates if the currently authenticated user has the necessary permissions to log into the parking panel.

#### **Authorization** {id="authorization_1"}

- Requires `LOGIN_IN_PARKING_PANEL` role.
- This endpoint is protected with `@PreAuthorize`.

#### **Response** {id="response_1"}

- **HTTP 200 OK**: If the authenticated user has the required role, an empty response is returned.
- **HTTP 401 Unauthorized**: If the user is not authenticated or doesn't have the required role.

#### **Example Usage** {id="example-usage_1"}

``` http
GET /api/v1/owners/check-permission HTTP/1.1
Authorization: Bearer <access_token>
```

**Response:**

``` http
HTTP 200 OK
```

#### **Error Scenarios** {id="error-scenarios_1"}

- `401 Unauthorized`: The user is not logged in, or `LOGIN_IN_PARKING_PANEL` role is missing.

### **2. Get Current Parking Owner Details**

#### **Endpoint** {id="endpoint_2"}

`GET /api/v1/owners/details`

#### **Description** {id="description_2"}

Fetches the details of the currently authenticated parking owner.

#### **Request**

- No parameters required.

#### **Response** {id="response_2"}

- **HTTP 200 OK**: Returns the parking owner details in the form of a `ParkingOwnerDto` object.
- **HTTP 401 Unauthorized**: If the user is not authenticated.

#### **Response Body Example**

``` json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "firstName": "John",
  "middleName": "D.",
  "lastName": "Doe",
  "NIP": "1234567890",
  "phoneNumber": "123456789",
  "phoneNumberCode": "+1",
  "address": {
    "country": "USA",
    "city": "New York",
    "street": "1st Avenue",
    "houseNumber": "123",
    "apartmentNumber": "4B",
    "postalCode": "10001"
  },
  "isRegistrationCompleted": true,
  "creationDate": "2023-01-01T12:34:56",
  "modificationDate": "2023-06-01T09:15:00"
}
```

#### **Example Usage** {id="example-usage_2"}

``` http
GET /api/v1/owners/details HTTP/1.1
Authorization: Bearer <access_token>
```

#### **Error Scenarios**

- `401 Unauthorized`: The user is not authenticated.
- `404 Not Found`: If the authenticated user does not exist.

### **3. Search Parking Owner Details**

#### **Endpoint** {id="endpoint_3"}

`GET /api/v1/owners/details/search`

#### **Description** {id="description_3"}

Fetches details of a parking owner using one or more query parameters such as `ownerId`, `firstName`, `lastName`, `NIP`,
`phoneNumber`, or `countryCode`.

#### **Request Parameters** {id="request-parameters_1"}

- **`ownerId` **(UUID): Identifier of the owner to search for.
- **`firstName` **(String): Owner's first name.
- **`lastName` **(String): Owner's last name.
- **`NIP` **(String): Owner's NIP (tax number).
- **`phoneNumber` **(String): Owner’s phone number.
- **`countryCode` **(String): Country code of the owner.

#### **Authorization**

- Requires `ADMIN` role.
- Protected by `@PreAuthorize`.

#### **Response** {id="response_3"}

- **HTTP 200 OK**: Search operation executed successfully (body purposely empty).
- **HTTP 401 Unauthorized**: If the user is unauthorized.
- **HTTP 403 Forbidden**: If the user does not have `ADMIN` role.
- **HTTP 404 Not Found**: If no results match the search criteria.

### **4. Update Parking Owner Address**

#### **Endpoint** {id="endpoint_4"}

`PUT /api/v1/owners/{userId}/address`

#### **Description** {id="description_4"}

Updates the address of a parking owner with the given `userId`.

#### **Request Parameters** {id="request-parameters_2"}

- **Path Variable:**
    - **`userId`** (UUID): The unique identifier of the parking owner whose address is being updated.

- **Request Body:**
    - **AddressDto:**

``` json
    {
  "country": "String",
  "city": "String",
  "street": "String",
  "houseNumber": "String",
  "apartmentNumber": "String",
  "postalCode": "String"
}
```

#### **Response** {id="response_4"}

- **HTTP 200 OK**: Address updated successfully.
- **HTTP 400 Bad Request**: The request body contains invalid data.
- **HTTP 404 Not Found**: The `userId` does not exist.

#### **Example Usage** {id="example-usage_3"}

``` http
PUT /api/v1/owners/123e4567-e89b-12d3-a456-426614174000/address HTTP/1.1
Content-Type: application/json
Authorization: Bearer <access_token>

{
  "country": "USA",
  "city": "New York",
  "street": "1st Avenue",
  "houseNumber": "123",
  "apartmentNumber": "4B",
  "postalCode": "10001"
}
```

**Response:**

``` http
HTTP 200 OK
```

### **5. Update Parking Owner Details**

#### **Endpoint**

`PUT /api/v1/owners/{userId}/details`

#### **Description**

Updates the personal details of a parking owner with the given `userId`.

#### **Request Parameters**

- **Path Variable:**
    - **`userId` **(UUID): The identifier of the parking owner.

- **Request Body:**
    - **ParkingOwnerDto:**

``` json
    {
  "id": "UUID",
  "firstName": "String",
  "middleName": "String",
  "lastName": "String",
  "NIP": "String",
  "phoneNumber": "String",
  "phoneNumberCode": "String",
  "address": {
    "country": "String",
    "city": "String",
    "street": "String",
    "houseNumber": "String",
    "apartmentNumber": "String",
    "postalCode": "String"
  },
  "isRegistrationCompleted": true,
  "creationDate": "LocalDateTime",
  "modificationDate": "LocalDateTime"
}
```

#### **Response**

- **HTTP 200 OK**: Details updated successfully.
- **HTTP 400 Bad Request**: Invalid data provided in the request body.
- **HTTP 404 Not Found**: The `userId` does not exist.

#### **Example Usage**

``` http
PUT /api/v1/owners/123e4567-e89b-12d3-a456-426614174000/details HTTP/1.1
Content-Type: application/json
Authorization: Bearer <access_token>

{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "firstName": "John",
  "middleName": "D.",
  "lastName": "Doe",
  "NIP": "1234567890",
  "phoneNumber": "123456789",
  "phoneNumberCode": "+1",
  "address": {
    "country": "USA",
    "city": "New York",
    "street": "1st Avenue",
    "houseNumber": "123",
    "apartmentNumber": "4B",
    "postalCode": "10001"
  },
  "isRegistrationCompleted": true,
  "creationDate": "2023-01-01T12:34:56",
  "modificationDate": "2023-06-01T09:15:00"
}
```

**Response:**

``` http
HTTP 200 OK
```

### **Error Handling For All Endpoints**

- **401 Unauthorized**: User is not logged in or doesn't have proper credentials.
- **403 Forbidden**: User does not have the required roles for the endpoint.
- **404 Not Found**: Resource (e.g., `userId`) does not exist.
- **400 Bad Request**: Invalid data provided (e.g., malformed JSON, invalid field values).
