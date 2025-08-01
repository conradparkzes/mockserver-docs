# Payment Scenario

This guide walks you through the complete Airtel Money payment flow using the Papi Mock Server. The process consists of three essential steps: getting an authentication token, initiating a payment, and checking the payment status.

## Prerequisites

- Access to the Papi Mock Server sandbox environment
- Valid API credentials for Airtel Money integration
- Basic understanding of REST API calls

## Complete Payment Flow

### Step 1: Get Authentication Token

First, you need to obtain an access token that will be used for subsequent API calls.

**Endpoint:** `POST https://sandbox.papi.mg/airtel/auth/oauth2/token`

**Headers:**
none

**Request Body (JSON):**
```
{
	"grant_type": "client_credentials",
	"client_id": "ed52af02-596c-42dc-bb12-0c90fcafb717",
	"client_secret": "31179cd6-21f8-4636-8868-21133f6f06f5"
}
```

**Success Response (200 OK):**
```json
{
  "access_token": "awA1BM1qxxoBS02iyyiNGQRt6yyMnz5MU31Syi3Z-S-Njw43Ctnfp9nyk5ll2bLBSxkEG2pVYhtSGzu4s9d0Vv3QiJn2p2FXJKoxmugxj30jYYHbtbse1Bf-lzJCAo3ulfuFpt6PVxn-DDzENc9WRmpYlJTcjewAXVCVqU47kqs",
  "token_type": "Bearer",
  "expires_in": 180
}
```

**Error Response Example - Missing grant_type:**
```json
{
  "error_description": "Invalid input grant_type: must not be blank; grant_type: must not be null; ",
  "error": "Validation Error"
}
```

> **Important:** Save the `access_token` from the success response, as you'll need to use this token in Step 2 as a Bearer token.

### Step 2: Initiate Payment

Using the `access_token` from Step 1, initiate a payment transaction.

**Endpoint:** `POST https://sandbox.papi.mg/airtel/merchant/v1/payments/`

**Headers:**
```
Authorization: Bearer {access_token_from_step_1}
X-Country: MG
X-Currency: MGA
```

**Request Body (JSON):**
```json
{
  "reference":"PAPI",
  "subscriber":{
    "country":"MG",
    "currency":"MGA",
    "msisdn": "331230002"
  },
  "transaction":{
    "amount":200,
    "country":"MG",
    "currency":"MGA",
    "id":"TestREFs00vfsd1"
  }
}
```

**Success Response (201 Created):**
```json
{
  "data": {
    "transaction": {
      "id": "TestREFs00vfsd1",
      "status": "Success."
    }
  },
  "status": {
    "response_code": "DP58071828535",
    "code": "200",
    "success": true,
    "result_code": "ESB264242",
    "message": "Success."
  }
}
```

**Error Response Example - Invalid Authorization:**
```json
{
  "path": "/airtel/merchant/v1/payments/",
  "error": "Internal Server Error",
  "message": "Your token was expired or not found. Please generate token again.",
  "timestamp": 1753945610329,
  "status": 500
}
```

> **Important:** Save the `id` from the body / success response, as you'll need it for the payment finalization in 
> Step 3.

### Step 3: Get Payment Status

Use the reference `id` from Step 2 along with the Bearer token. The reference id is added at the end of the endpoint.

**Endpoint:** `GET https://sandbox.papi.mg/airtel/standard/v1/payments/TestREFs00vfsd1`

**Headers:**
```
Authorization: Bearer {access_token_from_step_1}
X-Country: MG
X-Currency: MGA
version: 1.0
```

**Request Body:**
None (GET request)

**Success Response (200 OK):**
```json
{
  "data": {
    "transaction": {
      "id": "TestREFs00vfsd1",
      "status": "TS",
      "airtel_money_id": "MP848999",
      "message": "Transaction validée"
    }
  },
  "status": {
    "response_code": "DP501898490902",
    "code": "200",
    "success": true,
    "result_code": "ESB340432",
    "message": "SUCCESS"
  }
}
```

**Error Response Example - Invalid Reference Id:**
```json
{
  "status": {
    "code": "ESB658661",
    "message": "Invalid Token",
    "success": false,
    "response_code": "DP406272568337"
  }
}
```

## Request Parameters Explained

### Token Request Parameters

- **grant_type**: OAuth 2.0 grant type, must be "client_credentials"
- **client_id**: Your client identifier provided by Airtel Money
- **client_secret**: Your client secret provided by Airtel Money

### Payment Initiation Parameters

- **reference**: Merchant reference for the transaction (e.g., "PAPI")
- **subscriber**: Customer information object containing:
  - **country**: Country code (e.g., "MG" for Madagascar)
  - **currency**: Currency code (e.g., "MGA" for Malagasy Ariary)
  - **msisdn**: Customer's mobile phone number (e.g., "331230001")
- **transaction**: Transaction details object containing:
  - **amount**: Payment amount as integer (e.g., 200)
  - **country**: Country code (e.g., "MG")
  - **currency**: Currency code (e.g., "MGA")
  - **id**: Unique transaction identifier from your system

### Headers Explained

- **Authorization**: 
  - Step 1: Not required
  - Steps 2 & 3: Use `Bearer {access_token_from_step_1}`
- **X-Country**: Country code for the transaction (e.g., "MG")
- **X-Currency**: Currency code for the transaction (e.g., "MGA")
- **version**: API version (required for Step 3, use "1.0")

## Error Handling

Common error scenarios to handle in your integration:

### Step 1 - Token Generation Errors
- **Missing Parameters**: Ensure `grant_type`, `client_id`, and `client_secret` are included
- **Invalid Credentials**: Verify your client credentials are correct and active
- **Invalid Grant Type**: Must be "client_credentials"

### Step 2 - Payment Initiation Errors
- **Invalid Authorization**: Verify your access token is valid and not expired (180 seconds)
- **Missing Headers**: Ensure all required headers (`Authorization`, `X-Country`, `X-Currency`) are included
- **Invalid Parameters**: Verify phone number format and transaction details

### Step 3 - Status Check Errors
- **Invalid Reference**: Use the correct transaction ID from Step 2
- **Missing Headers**: Ensure all required headers including `version` are included
- **Token Expired**: Generate a new token if the access token has expired

## Response Status Codes

### Success Responses
- **200 OK**: Successful token generation (Step 1) or status retrieval (Step 3)
- **201 Created**: Could be returned for successful payment initiation (Step 2)

### Error Responses
- **400 Bad Request**: Invalid request parameters or validation errors
- **401 Unauthorized**: Invalid or expired authentication credentials
- **500 Internal Server Error**: Server-side errors or expired tokens

## Testing Notes

- The sandbox environment simulates real Airtel Money behavior
- Use the provided test phone numbers for development (see Conditional Phone Number Logic)
- Access tokens expire after 180 seconds (3 minutes) - much shorter than other providers
- Payment status in Step 2 is always "Success." regardless of phone number
- Final transaction status is determined in Step 3 based on the phone number used
- The mock server provides immediate status responses for testing purposes

## Important Implementation Notes

- **Token Expiration**: Airtel Money tokens expire quickly (180 seconds), implement proper token refresh logic
- **Transaction ID**: The `transaction.id` from Step 2 is required for Step 3 status checking
- **Status Interpretation**: 
  - `"status": "TS"` = Transaction Success
  - `"status": "TF"` = Transaction Failed
- **Response Structure**: All responses follow the nested `data` and `status` structure
- **Phone Number Testing**: Different phone numbers only affect Step 3 results, not Step 2
- **Error Handling**: Check both the HTTP status code and the `success` field in the response
