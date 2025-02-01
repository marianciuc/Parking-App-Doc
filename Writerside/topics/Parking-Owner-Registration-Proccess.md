# Parking Owner Registration Proccess

The `RegistrationController` handles the registration process for parking owners. It is part of a step-by-step
registration workflow that allows a parking owner to provide personal data and address information. All endpoints are
prefixed with **`/api/v1/owners/registration` **, following the system's versioning conventions.

After filling that data user status will be changed to `registeration_complete`. 

## Endpoints Overview

### **1. Step 1: Register Personal Data**

#### **Endpoint** {id="endpoint_1"}

`POST /api/v1/owners/registration/step/1`

#### **Description** {id="description_1"}

Registers the personal data of the parking owner as part of the first step in the registration process.

#### **Request** {id="request_1"}

##### Body {id="body_1"}

The request body must contain valid personal data, structured as:

- **PersonalDataRegistrationRequest**:

``` json
  {
  "firstName": "String",
  "lastName": "String",
  "email": "String",
  "phoneNumber": "String",
  "NIP": "String"
}
```

| **Field**     | **Type** | **Required** | **Description**                     |
|---------------|----------|--------------|-------------------------------------|
| `firstName`   | String   | Yes          | First name of the parking owner.    |
| `lastName`    | String   | Yes          | Last name of the parking owner.     |
| `email`       | String   | Yes          | Email address of the parking owner. |
| `phoneNumber` | String   | Yes          | Phone number of the parking owner.  |
| `NIP`         | String   | Yes          | Tax Identification Number (NIP).    |

##### Validation {id="validation_1"}

- Fields must not be null or empty.
- `email` should be a valid email format.

#### **Response** {id="response_1"}

- **HTTP 200 OK**: Indicates that the personal data has been successfully registered.
- **HTTP 400 Bad Request**: If the request body contains invalid or incomplete data.

#### **Example Usage** {id="example-usage_1"}

``` http
POST /api/v1/owners/registration/step/1 HTTP/1.1
Content-Type: application/json

{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@example.com",
  "phoneNumber": "123456789",
  "NIP": "1234567890"
}
```

**Response:**

``` http
HTTP 200 OK
```

#### **Error Scenarios** {id="error-scenarios_1"}

- `400 Bad Request`: Occurs when the provided data fails validation (e.g., missing required fields, invalid email
  format).
- `500 Internal Server Error`: If the service fails to process the data internally.

### **2. Step 2: Register Address**

#### **Endpoint**

`POST /api/v1/owners/registration/step/2`

#### **Description**

Registers the address of the parking owner as part of the second step in the registration process.

#### **Request**

##### Body

The request body must contain valid address information, structured as:

- **AddressDto**:

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

| **Field**         | **Type** | **Required** | **Description**                                       |
|-------------------|----------|--------------|-------------------------------------------------------|
| `country`         | String   | Yes          | The country of the parking owner.                     |
| `city`            | String   | Yes          | The city of the parking owner.                        |
| `street`          | String   | Yes          | The street name of the parking owner.                 |
| `houseNumber`     | String   | Yes          | The house number of the parking owner.                |
| `apartmentNumber` | String   | No           | The apartment number of the parking owner (optional). |
| `postalCode`      | String   | Yes          | The postal code of the parking owner.                 |

##### Validation

- Fields must not be null or empty.
- `postalCode` must follow the specified format for a valid postal code.

#### **Response**

- **HTTP 200 OK**: Indicates that the address data has been successfully registered.
- **HTTP 400 Bad Request**: If the request body contains invalid or incomplete data.

#### **Example Usage**

``` http
POST /api/v1/owners/registration/step/2 HTTP/1.1
Content-Type: application/json

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

#### **Error Scenarios**

- `400 Bad Request`: Occurs if required fields are missing or validations fail (e.g., invalid postal code format).
- `500 Internal Server Error`: If the service fails to process the data internally.

## Error Handling For All Endpoints

The following HTTP statuses may apply to any endpoint in this controller:

- **400 Bad Request**: Indicates that the request is not valid, due to either missing required fields or incorrect data
  formats.
- **500 Internal Server Error**: Indicates that the server encountered an unexpected condition and was unable to
  complete the request.
