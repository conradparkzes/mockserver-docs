# Conditional Phone Number Logic

The Papi Mock Server provides specific test phone numbers that simulate different payment scenarios for MVola transactions. These phone numbers should be used in the `debitParty` field during payment initiation and will determine the final status returned in Step 3 (Get Status).

## Test Phone Numbers

| Phone Number | Scenario | Final Status | Description |
|--------------|----------|--------------|-------------|
| `0341230001` | Success | `success` | Transaction completed successfully |
| `0341230002` | Insufficient Balance | `failed` | Insufficient funds in account |
| `0341230003` | Invalid Phone Number | `failed` | Phone number format or validity issue |
| `0341230004` | Account Blocked | `failed` | Account is blocked or restricted |

## How It Works

The conditional phone number logic affects the **final status** returned in Step 3 (Get Status), not the intermediate steps:

1. **Step 1 (Get Token)**: Always succeeds regardless of phone number
2. **Step 2 (Initiate Payment)**: Always returns `"status": "pending"` regardless of phone number
3. **Step 3 (Get Status)**: Returns different statuses based on the `debitParty` phone number used in Step 2

## Response Examples

### Success Scenario (0341230001)

**Status Response:**
```json
{
    "status": "success",
    "serverCorrelationId": "8tq6so4UyztNzGbyV9n0yw",
    "notificationMethod": "polling",
    "objectReference": ""
}
```

### Failed Scenarios (0341230002, 0341230003, 0341230004)

**Status Response:**
```json
{
    "status": "failed",
    "serverCorrelationId": "8tq6so4UyztNzGbyV9n0yw",
    "notificationMethod": "polling",
    "objectReference": ""
}
```

## Testing Instructions

### Testing Successful Payment Flow
- Use `0341230001` as the `debitParty` phone number
- Complete all three steps of the payment flow
- Verify that the final status returns `"success"`
- Use this to test your success handling logic

### Testing Insufficient Balance Handling
- Use `0341230002` as the `debitParty` phone number
- Complete all three steps of the payment flow
- Verify that the final status returns `"failed"`
- Use this to test how your system handles insufficient fund scenarios

### Testing Invalid Phone Number Handling
- Use `0341230003` as the `debitParty` phone number
- Complete all three steps of the payment flow
- Verify that the final status returns `"failed"`
- Use this to test how your system handles invalid phone number scenarios

### Testing Account Blocked Handling
- Use `0341230004` as the `debitParty` phone number
- Complete all three steps of the payment flow
- Verify that the final status returns `"failed"`
- Use this to test how your system handles blocked account scenarios

## Example Usage

When initiating a payment to test insufficient balance scenario:

```json
{
    "currency": "AR",
    "descriptionText": "Test insufficient balance",
    "amount": "250",
    "requestDate": "2025-04-16T07:44:50.080Z",
    "requestingOrganisationTransactionReference": "test-insufficient-balance-001",
    "debitParty": [
        {
            "key": "msisdn",
            "value": "0341230002"
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

## Integration Testing Strategy

1. **Comprehensive Flow Testing**: Test each phone number through the complete 3-step flow
2. **Status Polling**: Implement proper status checking logic that can handle both success and failed scenarios
3. **Error Handling**: Ensure your application properly handles failed payment statuses
4. **User Experience**: Design appropriate user feedback for different failure scenarios
5. **Logging**: Log different scenarios for debugging and monitoring purposes

## Important Notes

- The `creditParty` phone number does not affect the payment outcome
- Only the `debitParty` phone number determines the final status
- All phone numbers will successfully complete Steps 1 and 2
- The difference only appears in the final status check (Step 3)
