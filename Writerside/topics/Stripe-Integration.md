# Stripe Integration Documentation

This document outlines the integration of the Stripe API into the current system, used as a payment provider. The Stripe
API facilitates various payment-related operations, providing a secure and reliable platform for managing account
balances. These operations include depositing funds into an account and withdrawing funds to a bank account.

## Overview of Integration

- **API Type**: The system utilizes Stripe's RESTful APIs.
- **Authentication**: Configured with role-based access control, providing secure access to authorized users. Publicly
  available endpoints are secured for unauthorized access where applicable.
- **Use Case**: Useful for managing account balance operations, such as deposits and withdrawals.

---

## API Endpoints

### 1. Stripe Client Key

The **Stripe Client Key endpoint** provides the public key configured in the system for secure interaction with *
*Stripe's API**. This key is used by the client application to communicate securely with Stripe.

#### **Endpoint Details** {id="endpoint-details_2"}

| **HTTP Method**   | **GET**                            |
|-------------------|------------------------------------|
| **URL**           | `/api/v1/stripe`                   |
| **Authorization** | Not required (Publicly accessible) |

#### **Example Usage**

**Request:**

```bash
GET /api/v1/stripe
```

**Response:**

```bash
200 OK
Body:
{
    "pk_test_51Habcdefghxxxxx..."
}
```

#### **Error Handling**

| **Error Code**              | **Description**                                                                                  |
|-----------------------------|--------------------------------------------------------------------------------------------------|
| `500 INTERNAL SERVER ERROR` | Indicates that the `stripePublicKey` property is not properly configured or cannot be retrieved. |

---

### 2. Deposit Funds

This endpoint allows authorized users to deposit funds into a specific account. The details, including the amount and
currency to be deposited, are provided as part of the request body.

#### **Endpoint Details**

| **HTTP Method**   | **POST**                                                  |
|-------------------|-----------------------------------------------------------|
| **URL**           | `/api/v1/balance/operations/accounts/{accountId}/deposit` |
| **Authorization** | Requires roles: `DRIVER`, `PARKING_OWNER`                 |

#### **Request Parameters**

| **Parameter Type** | **Parameter Name** | **Type**                | **Description**                                                    |
|--------------------|--------------------|-------------------------|--------------------------------------------------------------------|
| Path Variable      | `accountId`        | `UUID`                  | Unique identifier for the account to deposit funds into.           |
| Request Body       | `paymentRequest`   | `PaymentRequest (JSON)` | Contains details about the payment, including amount and currency. |

- **PaymentRequest Schema**:

| **Field**  | **Type**     | **Description**                                   |
|------------|--------------|---------------------------------------------------|
| `amount`   | `BigDecimal` | The amount to deposit.                            |
| `currency` | `String`     | The currency of the deposit, like `USD` or `EUR`. |

#### **Response**

| **HTTP Status**             | **Description**                               |
|-----------------------------|-----------------------------------------------|
| `200 OK`                    | Returns a confirmation message as a `String`. |
| `400 BAD REQUEST`           | Invalid input, e.g., a negative amount.       |
| `403 FORBIDDEN`             | The user lacks the appropriate role.          |
| `404 NOT FOUND`             | The given `accountId` does not exist.         |
| `500 INTERNAL SERVER ERROR` | Occurs on unexpected server-side errors.      |

#### **Example Usage** {id="example-usage_2"}

**Request:**

```bash
POST /api/v1/balance/operations/accounts/123e4567-e89b-12d3-a456-426614174000/deposit
Authorization: Bearer <JWT_TOKEN>
Body:
{
    "amount": 150.75,
    "currency": "USD"
}
```

**Response:**

```bash
200 OK
"Deposit successfully completed."
```

---

### 3. Withdraw Funds

This endpoint enables authorized users to withdraw funds from their account to a specified bank account. The request
body provides necessary details, such as the amount and target bank account ID.

#### **Endpoint Details** {id="endpoint-details_1"}

| **HTTP Method**   | **POST**                                                        |
|-------------------|-----------------------------------------------------------------|
| **URL**           | `/api/v1/balance/operations/accounts/owners/{ownerId}/withdraw` |
| **Authorization** | Requires roles with withdrawal permissions                      |

#### **Request Parameters** {id="request-parameters_1"}

| **Parameter Type** | **Parameter Name** | **Type**                 | **Description**                                                       |
|--------------------|--------------------|--------------------------|-----------------------------------------------------------------------|
| Path Variable      | `ownerId`          | `UUID`                   | The unique identifier of the account owner initiating the withdrawal. |
| Request Body       | `withdrawRequest`  | `WithdrawRequest (JSON)` | Contains withdrawal details such as amount and bank account ID.       |

- **WithdrawRequest Schema**:

| **Field**       | **Type**     | **Description**                                  |
|-----------------|--------------|--------------------------------------------------|
| `bankAccountId` | `UUID`       | The ID of the bank account to transfer funds to. |
| `amount`        | `BigDecimal` | The amount to withdraw.                          |

#### **Response** {id="response_1"}

| **HTTP Status**             | **Description**                                            |
|-----------------------------|------------------------------------------------------------|
| `200 OK`                    | An empty response body indicating success.                 |
| `400 BAD REQUEST`           | Input validation error, e.g., insufficient balance.        |
| `403 FORBIDDEN`             | User lacks proper role permissions to perform this action. |
| `404 NOT FOUND`             | The `ownerId` or `bankAccountId` is not valid.             |
| `500 INTERNAL SERVER ERROR` | Occurs on unexpected server-side issues.                   |

#### **Example Usage** {id="example-usage_1"}

**Request:**

```bash
POST /api/v1/balance/operations/accounts/owners/123e4567-e89b-12d3-a456-426614174000/withdraw
Authorization: Bearer <JWT_TOKEN>
Body:
{
    "bankAccountId": "223e4567-e89b-12d3-a456-426614174001",
    "amount": 300.00
}
```

**Response:**

```bash
200 OK
"Withdrawal requested successfully."
```

---

## Notes and Considerations

1. **Security**: All endpoints involving sensitive operations require proper authentication and user roles in the
   system.
2. **Validation**: Clients should ensure that request parameters meet specified constraints to avoid response errors.
3. **Error Handling**: All errors return descriptive HTTP status codes with possible explanations in the error body.

This documentation should serve as a detailed integration guide for using Stripe API functionalities within the current
system.