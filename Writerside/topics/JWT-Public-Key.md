# JWT Public Key

The `PublicKeyController` is part of the `pl.edu.zut.app.parking.auth.controllers` package. It is responsible for
handling security-related endpoints, particularly for managing cryptographic operations associated with JSON Web
Tokens (JWT). This controller provides the public key required to verify JWTs securely in client-server communication.

#### **Base URL**

The endpoints under this controller are versioned with `v1` to ensure backward compatibility:

``` 
/api/v1/security/jwt
```

This API is primarily used by other services, systems, and clients for verifying signatures on JWTs issued by this
application.

### **Endpoints**

#### **1. Get JWT Public Key**

#### **Description**

This endpoint retrieves the public key required to verify cryptographic signatures on JWTs. The public key is used by
clients and external systems to validate tokens issued by the application.

#### **Request Details**

- **HTTP Method**: `GET`
- **URL**: `/api/v1/security/jwt/public-key`
- **Consumes**: No specific data required in the request.
- **Produces**: `text/plain` (plain text containing the public key)

#### **Annotations**

``` java
@Operation(
    summary = "Get JWT Public Key",
    description = "Retrieves the public key used for cryptographic operations such as verifying JSON Web Tokens (JWTs).",
    tags = {"JWT", "Security"}
)
@ApiResponses(value = {
    @ApiResponse(responseCode = "200", description = "Successfully retrieved the public key", content = {@Content(mediaType = "text/plain")}),
    @ApiResponse(responseCode = "500", description = "Server error while retrieving the public key", content = @Content)
})
@GetMapping(value = "/public-key", produces = MediaType.TEXT_PLAIN_VALUE)
```

#### **Response** {id="response_1"}

The API returns the public key as plain text to be used for verifying JWTs.

- **HTTP Status Codes**:
    - `200 OK`: Public key successfully retrieved.
    - `500 Internal Server Error`: An error occurred while retrieving the key.

- **Response Body**:
    - **On Success**: The public key as a `String` (in plain text format).
    - **On Error**: Error details from the server, if applicable.

#### **Authorization**

This endpoint does not require authentication because the public key is intended to be accessible by external clients.

#### **Example Usage**

##### **Request**

``` http
GET /api/v1/security/jwt/public-key HTTP/1.1
Host: api.example.com
Accept: text/plain
```

##### **Response**

``` http
HTTP/1.1 200 OK
Content-Type: text/plain

-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAnjk2g8N9zKeAsd3...
-----END PUBLIC KEY-----
```

##### **Error Response**

If the server encounters any issues while retrieving the public key:

``` http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "timestamp": "2023-10-15T10:05:42.456+00:00",
  "status": 500,
  "error": "Internal Server Error",
  "message": "Could not retrieve public key",
  "path": "/api/v1/security/jwt/public-key"
}
```

### **Error Scenarios and Edge Cases**

| **Error Code**              | **Description**                                                | **Possible Causes**                                                                |
|-----------------------------|----------------------------------------------------------------|------------------------------------------------------------------------------------|
| `500 Internal Server Error` | The server encountered an issue while fetching the public key. | - The `JWTService` implementation failed or threw an exception.                    |
| `404 Not Found`             | The endpoint is misconfigured or not found on the server.      | - Misalignment in application routing or service mapping during server deployment. |

### **Implementation Notes**

1. **Business Logic**: The method `getPublicKey()` internally calls the `JWTService#getPublicKey` method to fetch the
   key. This service ensures secure storage and retrieval of the cryptographic materials.
    - The functionality ensures that any downstream system relying on token verification can access the public key.

2. **API Design Choices**:
    - **Versioning**: Versioned under `/api/v1/` to allow upgrades without breaking existing consumers.
    - **Plain Text Response**: The public key is returned directly in plain text format to simplify parsing by clients.

3. **Security**:
    - No authentication is applied because this is a public key.
    - However, this endpoint is sensitive, and more stringent security (e.g., API Gateway rate limiting) may be helpful
      to prevent misuse (e.g., DDoS).

4. **Testing and Monitoring**:
    - Ensure regular testing of the endpoint for speed, reliability, and accuracy of the public key retrieval.
    - Monitor logs for spikes in requests to ensure no misuse of the system.
