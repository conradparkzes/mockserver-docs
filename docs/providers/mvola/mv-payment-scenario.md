# Payment Scenario

This guide walks you through the complete MVola payment flow using the Papi Mock Server. The process consists of three essential steps: getting an authentication token, initiating a payment, and checking the payment status.

## Prerequisites

- Access to the Papi Mock Server sandbox environment
- Valid API credentials for MVola integration
- Basic understanding of REST API calls

## Complete Payment Flow

### Step 1: Get Authentication Token

First, you need to obtain an access token that will be used for subsequent API calls.

**Endpoint:** `POST https://sandbox.papi.mg/token`

**Headers:**
```
X-Custom-Authorization: Basic your_token
Content-Type: application/x-www-form-urlencoded
```

**Request Body (URL-encoded):**
```
grant_type=client_credentials
scope=EXT_INT_MVOLA_SCOPE
```

**Success Response (200 OK):**
```json
{
    "access_token": "ft3G8TJZSkrgd8NRg5L7ttJVPvsI6WQ8F7i8k-3s8mIGuIhO5klmVH5EVYCWk8EGd2aluG6m5_HDfPu6dgIGyI6-J09lIxvlULrcbEz2ofOaYZEVztVEEo6egW31fofpJGXeu0u9cfdYHmuthyyznfZ-98pToKcEHznKyq2dNHI",
    "scope": "EXT_INT_MVOLA_SCOPE",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

**Error Response Example - Missing grant_type:**
```json
{
    "error_description": "Required request parameter 'grant_type' for method parameter type String is not present",
    "error": "Missing Parameter"
}
```

> **Important:** Save the `access_token` from the success response, as you'll need to use this token (not your initial authorization token) in the `X-Custom-Authorization` header for both Steps 2 and 3.

### Step 2: Initiate Payment

Using the `access_token` from Step 1, initiate a payment transaction.

**Endpoint:** `POST https://sandbox.papi.mg/mvola/mm/transactions/type/merchantpay/1.0.0`

**Headers:**
```
X-Custom-Authorization: Basic {access_token_from_step_1}
X-correlationID: dsq;,dvbwfghlkarhkfs
version: 1.0
Content-Type: application/json
```

**Request Body (JSON):**
```json
{
    "currency": "AR",
    "descriptionText": "12345",
    "amount": "250",
    "requestDate": "2025-04-16T07:44:50.080Z",
    "requestingOrganisationTransactionReference": "ab542bb4-d9c7-4caf-8ecd-db731b2cb9dc",
    "debitParty": [
        {
            "key": "msisdn",
            "value": "0341230001"
        }
    ],
    "creditParty": [
        {
            "key": "msisdn",
            "value": "0386206745"
        }
    ],
    "metadata": [
        {
            "key": "partnerName",
            "value": "Ibonia"
        }
    ]
}
```

**Success Response (200 OK):**
```json
{
    "status": "pending",
    "serverCorrelationId": "8tq6so4UyztNzGbyV9n0yw",
    "notificationMethod": "polling"
}
```

**Error Response Example - Missing Authorization Header:**
```json
{
    "errorCategory": "validation",
    "errorCode": "formatError",
    "errorDescription": "Missing field",
    "errorDateTime": "2025-07-30T07:16:36.628912145",
    "errorParameters": [
        {
            "value": "4001",
            "key": "mmErrorCode"
        }
    ]
}
```

> **Important:** Save the `serverCorrelationId` from the success response, as you'll need it for the status check in Step 3.

### Step 3: Check Payment Status

Use the `serverCorrelationId` from Step 2 and the `access_token` from Step 1 to check the payment status.

**Endpoint:** `GET https://sandbox.papi.mg/mvola/mm/transactions/type/merchantpay/1.0.0/status/{serverCorrelationId}`

**Headers:**
```
X-Custom-Authorization: Basic {access_token_from_step_1}
X-CorrelationID: {serverCorrelationId}
partnerName: Ibonia
UserAccountIdentifier: msisdn;0386206745
```

**Success Response (200 OK):**
```json
{
    "status": "success",
    "serverCorrelationId": "ZqgznmkbG4DRNYV_18dctw",
    "notificationMethod": "polling",
    "objectReference": ""
}
```

**Error Response Example - Incorrect Server Correlation ID:**
```json
{
    "errorCategory": "validation",
    "errorCode": "formatError",
    "errorDescription": "No transaction found with id ZqgznmkbG4DRNYV_18dct",
    "errorDateTime": "2025-07-30T07:19:07.237606457",
    "errorParameters": [
        {
            "value": "4001",
            "key": "mmErrorCode"
        }
    ]
}
```

## Request Parameters Explained

### Payment Initiation Parameters

- **currency**: Transaction currency code (e.g., "AR" for Ariary)
- **descriptionText**: Transaction description
- **amount**: Payment amount as string
- **requestDate**: ISO 8601 formatted timestamp
- **requestingOrganisationTransactionReference**: Unique transaction reference from your system
- **debitParty**: Payer's mobile number (MSISDN format)
- **creditParty**: Payee's mobile number (MSISDN format)
- **metadata**: Additional transaction metadata (partner name, etc.)

### Headers Explained

- **Authorization**: Use the `access_token` obtained from Step 1 (not your initial authorization token)
- **X-correlationID**: Unique identifier for request tracking
- **version**: API version (use "1.0")
- **partnerName**: Your organization's partner name
- **UserAccountIdentifier**: User account in MSISDN format

## Error Handling

Common error scenarios to handle in your integration:

1. **Missing Parameters**: Ensure all required fields are included in requests
2. **Invalid Authorization**: Verify your authentication token is valid and not expired
3. **Transaction Not Found**: Use the correct `serverCorrelationId` from the payment initiation response
4. **Format Errors**: Ensure date formats, phone numbers, and other fields follow the expected format

## Testing Notes

- The sandbox environment simulates real MVola behavior
- Use the provided test phone numbers for development (see Conditional Phone Number Logic)
- Tokens expire after 3600 seconds (1 hour)
- Always check the payment status before considering a transaction complete
