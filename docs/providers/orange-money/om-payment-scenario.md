# Payment Scenario

This guide walks you through the complete Orange Money payment flow using the Papi Mock Server. The process consists of three essential steps: getting an authentication token, initiating a webpayment, and finalizing the payment.

## Prerequisites

- Access to the Papi Mock Server sandbox environment
- Valid API credentials for Orange Money integration
- Basic understanding of REST API calls

## Complete Payment Flow

### Step 1: Get Authentication Token

First, you need to obtain an access token that will be used for subsequent API calls.

**Endpoint:** `POST https://sandbox.papi.mg/orange/oauth/v3/token`

**Headers:**
```
Authorization: Basic your_token
```

**Request Body (URL-encoded):**
```
grant_type=client_credentials
```

**Success Response (200 OK):**
```json
{
    "access_token": "CMhEx6kiLjMcheYvZtU-ap2N4Zx2wTgNWN2mRRMlSgoRz9kHoqh8mYcDYJHUND2Ek-hb8TPyJ_AEW8CtjYx6QsW-FOi0q-PS6IzaufS00Qnis5xqDQFL2uGixQRFQZDGMGrLkgtiuoSFJXpH1pcljsI6uuVaul-d0fO0l0GmqDY",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

**Error Response Example - Missing grant_type:**
```json
{
  "timestamp": "2025-07-31T05:58:48.964+00:00",
  "status": 415,
  "error": "Unsupported Media Type",
  "path": "/orange/oauth/v3/token"
}
```

> **Important:** Save the `access_token` from the success response, as you'll need to use this token in Step 2 as a Bearer token.

### Step 2: Initiate Webpayment

Using the `access_token` from Step 1, initiate a webpayment transaction.

**Endpoint:** `POST https://sandbox.papi.mg/orange/orange-money-webpay/mg/v1/webpayment`

**Headers:**
```
Authorization: Bearer {access_token_from_step_1}
Content-Type: application/json
```

**Request Body (JSON):**
```json
{
    "merchant_key": "18375049",
    "currency": "MGA",
    "order_id": "Mcsts5kk14ds8dd2d1dsqdqsd0_0ddd023457sdqdqs",
    "amount": 200,
    "return_url": "https://9935ca7e3c4b.ngrok-free.app",
    "cancel_url": "https://9935ca7e3c4b.ngrok-free.app/txncncld/",
    "notif_url": "https://google.com",
    "lang": "fr",
    "reference": "ref Merchant"
}
```

**Success Response (201 Created):**
```json
{
    "status": 201,
    "message": "OK",
    "pay_token": "0Oi5D_jzsXb6kj9EpKCVLA",
    "payment_url": "https://www.orange.com/payment",
    "notif_token": "cRi_RTVOSxHB_NJXTX39_g"
}
```

**Error Response Example - Invalid Authorization:**
```json
{
  "code": "401",
  "message": "Invalid Authorization Token",
  "description": "Your token was expired or not found. Please generate token again."
}
```

> **Important:** Save the `pay_token` from the success response, as you'll need it for the payment finalization in Step 3.

### Step 3: Finalize Payment

Use the `pay_token` from Step 2 along with the customer's phone number and OTP to finalize the payment.

**Endpoint:** `POST https://sandbox.papi.mg/orange/mg/mpayment/finalize`

**Headers:**
```
Content-Type: application/x-www-form-urlencoded
```

**Request Body (URL-encoded):**
```
Token={pay_token_from_step_2}
Msisdn=0331230001
Otp=25698
```

**Success Response (200 OK):**
```json
{
    "status": "SUCCESS",
    "accessLayer": "",
    "txnid": "MPwdPywkWsf9DiFK0TDznqEw",
    "message": "Votre paiement est confirme",
    "description": "Paiement effectué avec succès."
}
```

**Error Response Example - Invalid Pay Token:**
```json
{
"code": "400",
"message": "Invalid Request",
"description": null
}
```

## Request Parameters Explained

### Webpayment Initiation Parameters

- **merchant_key**: Your merchant identifier (e.g., "18375049")
- **currency**: Transaction currency code (e.g., "MGA" for Malagasy Ariary)
- **order_id**: Unique order identifier from your system
- **amount**: Payment amount as integer
- **return_url**: URL to redirect after successful payment
- **cancel_url**: URL to redirect after payment cancellation
- **notif_url**: URL for payment notifications/webhooks
- **lang**: Language code (e.g., "fr" for French)
- **reference**: Merchant reference for the transaction

### Payment Finalization Parameters

- **Token**: The pay token obtained from Step 2
- **Msisdn**: Customer's mobile phone number in MSISDN format
- **Otp**: One-time password provided by the customer

### Headers Explained

- **Authorization**: 
  - Step 1: Use `Basic your_initial_token`
  - Step 2: Use `Bearer {access_token_from_step_1}`
- **Content-Type**: 
  - Steps 1 & 3: `application/x-www-form-urlencoded`
  - Step 2: `application/json`

## Error Handling

Common error scenarios to handle in your integration:

1. **Missing Parameters**: Ensure all required fields are included in requests
2. **Invalid Authorization**: Verify your authentication tokens are valid and not expired
3. **Invalid Pay Token**: Use the correct `pay_token` from the webpayment initiation response
4. **Format Errors**: Ensure phone numbers, amounts, and other fields follow the expected format
5. **Payment Failures**: Handle different failure scenarios (insufficient balance, blocked accounts, etc.)

## Testing Notes

- The sandbox environment simulates real Orange Money behavior
- Use the provided test phone numbers for development (see Conditional Phone Number Logic)
- Access tokens expire after 3600 seconds (1 hour)
- In production, customers would be redirected to the `payment_url` from Step 2 to complete payment
- The mock server simulates the finalization step directly for testing purposes

