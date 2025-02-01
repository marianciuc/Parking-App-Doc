# Vehicle Management

The `VehicleController` is a REST controller for managing vehicles in the system. It provides endpoints to create,
update, search, and retrieve vehicles. All endpoints are hosted under the `/api/v1/vehicles` base URL, following *
*versioning best practices**.
This controller interacts with the **VehicleService** to process and manage vehicle data and ensure secure and efficient
data manipulation. It supports pagination for searching vehicles and provides validations for user inputs.

## Endpoints

### 1. **Search Vehicles**

- **HTTP Method**: `GET`
- **Endpoint**: `/api/v1/vehicles/search`
- **Description**: Searches for vehicles based on specific filters provided in the request body. Results are paginated
  with `page` and `size` parameters.

#### Parameters: {id="parameters_1"}

| Parameter | Type                   | Location     | Description                                       |
|-----------|------------------------|--------------|---------------------------------------------------|
| `page`    | `int`                  | Query Param  | The page number (`0` by default).                 |
| `size`    | `int`                  | Query Param  | The number of results per page (`10` by default). |
| `request` | `VehicleSearchRequest` | Request Body | Contains filters for vehicle search.              |

#### Request Example: {id="request-example_1"}

``` json
GET /api/v1/vehicles/search?page=0&size=10
Content-Type: application/json

{
"plateNumber": "ABC-12345",
"brand": "Toyota",
"yearOfProduction": 2020
}
```

#### Response: {id="response_1"}

| HTTP Status | Description                                     | JSON Example                |
|-------------|-------------------------------------------------|-----------------------------|
| `200 OK`    | Paginated list of vehicles matching the filter. | See response example below. |

``` json
{
  "content": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "brand": "Toyota",
      "model": "Corolla",
      "plateNumber": "ABC-12345",
      "color": "Red",
      "hasOwner": true
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10
  },
  "totalElements": 1,
  "totalPages": 1
}
```

#### Possible Errors:

| HTTP Status       | Error                     | Description                                                                        |
|-------------------|---------------------------|------------------------------------------------------------------------------------|
| `400 Bad Request` | `InvalidInputException`   | Input data validation failed, e.g., invalid JSON or parameter constraints not met. |
| `500 Internal`    | `VehicleServiceException` | Unexpected error in the service layer.                                             |

### 2. **Find by Plate Number**

- **HTTP Method**: `GET`
- **Endpoint**: `/api/v1/vehicles/find-by-plate`
- **Description**: Retrieves a vehicle by its license plate.

#### Parameters: {id="parameters_2"}

| Parameter | Type   | Location    | Description                        |
|-----------|--------|-------------|------------------------------------|
| `plate`   | String | Query Param | The license plate number to query. |

#### Request Example: {id="request-example_2"}

``` http
GET /api/v1/vehicles/find-by-plate?plate=ABC-12345
```

#### Response: {id="response_2"}

| HTTP Status     | Description                                  | JSON Example                                                   |
|-----------------|----------------------------------------------|----------------------------------------------------------------|
| `200 OK`        | The vehicle with the specified plate number. | `{ "id": "UUID", "brand": "Toyota", "model": "Corolla", ... }` |
| `404 Not Found` | `VehicleNotFoundException`                   | No vehicle found with the given plate.                         |

### 3. **Find by Vehicle ID**

- **HTTP Method**: `GET`
- **Endpoint**: `/api/v1/vehicles/{vehicleId}`
- **Description**: Retrieves a vehicle by its unique identifier.

#### Parameters: {id="parameters_3"}

| Parameter   | Type | Location   | Description                            |
|-------------|------|------------|----------------------------------------|
| `vehicleId` | UUID | Path Param | The unique ID of the vehicle to query. |

#### Request Example: {id="request-example_3"}

``` http
GET /api/v1/vehicles/550e8400-e29b-41d4-a716-446655440000
```

#### Response: {id="response_3"}

| HTTP Status     | Description                        | JSON Example                                                 |
|-----------------|------------------------------------|--------------------------------------------------------------|
| `200 OK`        | The vehicle with the specified ID. | `{ "id": "UUID", "brand": "Toyota", "model": "Yaris", ... }` |
| `404 Not Found` | `VehicleNotFoundException`         | No vehicle found with the given ID.                          |

### 4. **Update a Vehicle**

- **HTTP Method**: `PUT`
- **Endpoint**: `/api/v1/vehicles/{vehicleId}`
- **Description**: Updates the details of an existing vehicle.

#### Parameters: {id="parameters_4"}

| Parameter    | Type         | Location     | Description                             |
|--------------|--------------|--------------|-----------------------------------------|
| `vehicleId`  | UUID         | Path Param   | The unique ID of the vehicle to update. |
| `vehicleDto` | `VehicleDto` | Request Body | The updated vehicle details.            |

#### Request Example: {id="request-example_4"}

``` http
PUT /api/v1/vehicles/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{
  "brand": "Toyota",
  "model": "Camry",
  "plateNumber": "XYZ-67890",
  "color": "Blue"
}
```

#### Response: {id="response_4_1"}

| HTTP Status     | Description                | JSON Example                                                 |
|-----------------|----------------------------|--------------------------------------------------------------|
| `200 OK`        | Updated vehicle details.   | `{ "id": "UUID", "brand": "Toyota", "model": "Camry", ... }` |
| `404 Not Found` | `VehicleNotFoundException` | No vehicle found with the given ID.                          |

### 5. **Create a New Vehicle**

- **HTTP Method**: `POST`
- **Endpoint**: `/api/v1/vehicles`
- **Description**: Creates a new vehicle in the system.

#### Parameters:

| Parameter         | Type         | Location     | Description                                                      |
|-------------------|--------------|--------------|------------------------------------------------------------------|
| `vehicleDto`      | `VehicleDto` | Request Body | The details of the vehicle to create.                            |
| `assignOwnership` | `Boolean`    | Query Param  | Whether to assign ownership to the vehicle. Defaults to `false`. |

#### Request Example:

``` json
POST /api/v1/vehicles?assignOwnership=true
Content-Type: application/json

{
"brand": "Ford",
"model": "Fiesta",
"plateNumber": "FORD-2023",
"color": "Blue",
"yearOfProduction": 2023,
"vin": "1HGCM82633A123456"
}
```

#### Response: {id="response_4"}

| HTTP Status       | Description                     | JSON Example                                                |
|-------------------|---------------------------------|-------------------------------------------------------------|
| `201 Created`     | The newly created vehicle.      | `{ "id": "UUID", "brand": "Ford", "model": "Fiesta", ... }` |
| `400 Bad Request` | `InvalidVehicleDataException`   | Provided data is invalid.                                   |
| `409 Conflict`    | `VehicleAlreadyExistsException` | A vehicle with this plate already exists.                   |

## Error Responses

| Error Code        | Description                                     |
|-------------------|-------------------------------------------------|
| `400 Bad Request` | Validation errors in input data.                |
| `404 Not Found`   | Resource not found (e.g., invalid ID or plate). |
| `409 Conflict`    | Resource conflict (e.g., duplicate plate).      |
| `500 Internal`    | Unexpected server-side error.                   |
