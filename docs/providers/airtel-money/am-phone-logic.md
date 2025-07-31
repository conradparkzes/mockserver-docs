# Conditional Phone Number Logic

The Papi Mock Server provides specific test phone numbers that simulate different payment scenarios for Airtel Money transactions. These phone numbers should be used in the `msisdn` field during payment initiation (Step 2) and will determine the final status returned in Step 3 (Get Status).

## Test Phone Numbers

| Phone Number | Scenario | Final Status | Response Code | Description |
|--------------|----------|--------------|---------------|-------------|
| `331230001` | Success | `TS` | 200           | Transaction completed successfully |
| `331230002` | Insufficient Balance | `TF` | 400           | Insufficient funds in account |
| `331230003` | Invalid Phone Number | `TF` | 401           | Phone number format or validity issue |
| `331230004` | Account Blocked | `TF` | 401           | Account is blocked or restricted |

## How It Works

The conditional phone number logic affects the **final status** returned in Step 3 (Get Status), not the intermediate steps:

1. **Step 1 (Get Token)**: Always succeeds regardless of phone number
2. **Step 2 (Initiate Payment)**: Always returns `"status": "Success."` regardless of phone number
3. **Step 3 (Get Status)**: Returns different statuses based on the `msisdn` phone number used in Step 2

## Response Examples

### Success Scenario (331230001)

**Status Response:**
```json
{
  "data": {
    "transaction": {
      "id": "TestREF281",
      "status": "TS",
      "airtel_money_id": "MP024493",
      "message": "Transaction validée"
    }
  },
  "status": {
    "response_code": "DP402840321655",
    "code": "200",
    "success": true,
    "result_code": "ESB101005",
    "message": "SUCCESS"
  }
}
```

### Failed Scenarios

#### Insufficient Balance (331230002)
```json
{
  "data": {
    "transaction": {
      "id": "TestREF267523",
      "status": "TF",
      "airtel_money_id": "MP060291",
      "message": "Insufficient balance"
    }
  },
  "status": {
    "response_code": "DP05538018594",
    "code": "400",
    "success": false,
    "result_code": "ESB881971",
    "message": "FAILED"
  }
}
```

#### Invalid Phone Number (331230003)
```json
{
  "data": {
    "transaction": {
      "id": "TestREF967131",
      "status": "TF",
      "airtel_money_id": "MP940350",
      "message": "Invalid phone number"
    }
  },
  "status": {
    "response_code": "DP571783374548",
    "code": "401",
    "success": false,
    "result_code": "ESB312138",
    "message": "FAILED"
  }
}
```

#### Account Blocked (331230004)
```json
{
  "data": {
    "transaction": {
      "id": "TestREF907893",
      "status": "TF",
      "airtel_money_id": "MP460733",
      "message": "Account blocked"
    }
  },
  "status": {
    "response_code": "DP425714684359",
    "code": "401",
    "success": false,
    "result_code": "ESB737253",
    "message": "FAILED"
  }
}
```

## Testing Instructions

### Testing Successful Payment Flow
- Use `331230001` as the `msisdn` phone number in Step 2
- Complete all three steps of the payment flow
- Verify that the final status returns `"status": "TS"` and `"success": true`
- Use this to test your success handling logic

### Testing Insufficient Balance Handling
- Use `331230002` as the `msisdn` phone number in Step 2
- Complete all three steps of the payment flow
- Verify that the final status returns `"status": "TF"` and `"success": false`
- Use this to test how your system handles insufficient fund scenarios

### Testing Invalid Phone Number Handling
- Use `331230003` as the `msisdn` phone number in Step 2
- Complete all three steps of the payment flow
- Verify that the final status returns `"status": "TF"` and `"success": false`
- Use this to test how your system handles invalid phone number scenarios

### Testing Account Blocked Handling
- Use `331230004` as the `msisdn` phone number in Step 2
- Complete all three steps of the payment flow
- Verify that the final status returns `"status": "TF"` and `"success": false`
- Use this to test how your system handles blocked account scenarios

## Example Usage

When initiating a payment to test insufficient balance scenario:

```json
{
    "reference": "PAPI",
    "subscriber": {
        "country": "MG",
        "currency": "MGA",
        "msisdn": "331230002"
    },
    "transaction": {
        "amount": 200,
        "country": "MG",
        "currency": "MGA",
        "id": "TestREF-insufficient-balance-001"
    }
}
```

## Integration Testing Strategy

1. **Comprehensive Flow Testing**: Test each phone number through the complete 3-step flow
2. **Status Polling**: Implement proper status checking logic that can handle both TS (success) and TF (failed) scenarios
3. **Error Handling**: Ensure your application properly handles failed payment statuses based on the `success` field
4. **Response Processing**: Handle both `airtel_money_id` for successful transactions and error messages for failed ones
5. **User Experience**: Design appropriate user feedback for different failure scenarios
6. **Logging**: Log different scenarios with their respective response codes for debugging and monitoring purposes

## Important Notes

- Phone numbers only affect the final status in Step 3 (Get Status)
- Steps 1 and 2 will always succeed regardless of the phone number you plan to use in Step 2
- The `reference` and `transaction.id` from Step 2 are required for Step 3 status checking
- Response codes and result codes provide specific failure reasons that can be used for error handling
- The `airtel_money_id` is only provided for successful transactions (TS status)
