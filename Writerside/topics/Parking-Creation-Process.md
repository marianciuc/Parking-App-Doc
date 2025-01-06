# Parking Creation Process

### Endpoint: POST `/parking/creation`

#### Description:

This endpoint is used to create a new parking and associate it with an owner. To successfully interact with this
endpoint, the request must include a valid JWT token for authentication and authorization.

#### Requirements:

- **JWT Token**: A valid JSON Web Token (JWT) must be included in the request headers for authentication.
- **User Role**:
    - The user must have the `PARKING_OWNER` role in the system.
    - The user must also have the necessary permissions associated with this role.

#### Purpose:

This operation is restricted to users assigned the `PARKING_OWNER` role, ensuring that only authorized individuals can
associate a new parking entity to the specified owner.


Request body to create new parking:

``` JSON
{
  "name": "Central Parking",
  "address": {
    "city": "New York",
    "countryCode": "US",
    "postalCode": "10001",
    "street": "5th Avenue",
    "buildingNumber": "12A",
    "latitude": 40.712776,
    "longitude": -74.005974,
    "isBelongToAnyInstitution": true,
    "institutionName": "Big Corp"
  }
}

```

