# Conditional Phone Number Logic

The Papi Mock Server provides specific test phone numbers that simulate different payment scenarios for Orange Money transactions. These phone numbers should be used in the `Msisdn` field during payment finalization (Step 3) and will determine the final status and response returned.

## Test Phone Numbers

| Phone Number | Scenario | Final Status | Response Code | Description |
|--------------|----------|--------------|---------------|-------------|
| `0321230001` | Success | `SUCCESS` | 200           | Transaction completed successfully |
| `0321230002` | Insufficient Balance | `FAILED` | 400           | Insufficient funds in account |
| `0321230003` | Invalid Phone Number | `FAILED` | 401           | Phone number format or validity issue |
| `0321230004` | Account Blocked | `FAILED` | 401           | Account is blocked or restricted |

## How It Works

The conditional phone number logic affects the **payment finalization** in Step 3, not the intermediate steps:

1. **Step 1 (Get Token)**: Always succeeds regardless of phone number
2. **Step 2 (Initiate Webpayment)**: Always returns status 201 with pay_token regardless of phone number  
3. **Step 3 (Finalize Payment)**: Returns different statuses and error codes based on the `Msisdn` phone number used

## Response Examples

### Success Scenario (0331230001)

**Finalization Response:**
```json
{
    "status": "SUCCESS",
    "accessLayer": "",
    "txnid": "MPwdPywkWsf9DiFK0TDznqEw",
    "message": "Votre paiement est confirme",
    "description": "Paiement effectué avec succès."
}
```

### Failed Scenarios

#### Insufficient Balance (0331230002)
```json
{
    "status": "FAILED",
    "message": "Votre paiement a echoue",
    "description": "Insufficient balance",
    "code": 970
}
```

#### Invalid Phone Number (0331230003)
```json
{
    "status": "FAILED",
    "message": "Votre paiement a echoue",
    "description": "Invalid phone number",
    "code": 109
}
```

#### Account Blocked (0331230004)
```json
{
    "status": "FAILED",
    "message": "Votre paiement a echoue",
    "description": "Account blocked",
    "code": 925
}
```

## Testing Instructions

### Testing Successful Payment Flow
- Use `0331230001` as the `Msisdn` in Step 3 (finalization)
- Complete all three steps of the payment flow
- Verify that the final response returns `"status": "SUCCESS"`
- Use this to test your success handling logic and transaction ID processing

### Testing Insufficient Balance Handling
- Use `0331230002` as the `Msisdn` in Step 3 (finalization)
- Complete all three steps of the payment flow
- Verify that the final response returns `"status": "FAILED"` with code 970
- Use this to test how your system handles insufficient fund scenarios

### Testing Invalid Phone Number Handling
- Use `0331230003` as the `Msisdn` in Step 3 (finalization)
- Complete all three steps of the payment flow
- Verify that the final response returns `"status": "FAILED"` with code 109
- Use this to test how your system validates and handles invalid phone numbers

### Testing Account Blocked Handling
- Use `0331230004` as the `Msisdn` in Step 3 (finalization)
- Complete all three steps of the payment flow
- Verify that the final response returns `"status": "FAILED"` with code 925
- Use this to test how your system handles blocked account scenarios

## Example Usage

When finalizing a payment to test insufficient balance scenario:

**Step 3 Request Body:**
```
Token=0Oi5D_jzsXb6kj9EpKCVLA
Msisdn=0331230002
Otp=25698
```

**Expected Response:**
```json
{
    "status": "FAILED",
    "message": "Votre paiement a echoue",
    "description": "Insufficient balance",
    "code": 970
}
```

## Integration Testing Strategy

1. **Comprehensive Flow Testing**: Test each phone number through the complete 3-step flow
2. **Error Code Handling**: Implement proper handling for different error codes (970, 109, 925)
3. **Status Processing**: Ensure your application properly differentiates between SUCCESS and FAILED statuses
4. **User Experience**: Design appropriate user feedback for different failure scenarios with localized messages
5. **Transaction Management**: Handle transaction IDs for successful payments and error tracking for failed ones
6. **Logging**: Log different scenarios with their respective error codes for debugging and monitoring

## Important Notes

- Phone numbers only affect the finalization step (Step 3)
- Steps 1 and 2 will always succeed regardless of the phone number you plan to use
- The `pay_token` from Step 2 is always valid and required for Step 3
- Error codes provide specific failure reasons that can be used for user messaging
- OTP value (25698) is fixed for testing purposes
