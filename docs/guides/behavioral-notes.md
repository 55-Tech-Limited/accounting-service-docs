---
sidebar_position: 3
description: "Important behavioral notes and considerations for using the Wager Track API"
---

# Behavioral Notes & Considerations

This guide outlines important behavioral patterns and considerations when using the Wager Track API. Understanding these behaviors will help you integrate the API more effectively and avoid common pitfalls.

## Bet Management Behaviors

### Bet Activation and Deactivation

When managing bets through the admin API, there are several important behaviors to understand:

#### Setting Bets Inactive

When a bet is set to inactive using the `POST /admin/bets/:betId/set-inactive` endpoint:

- **Financial Rollback**: The system performs a complete rollback of the bet's financial impact
- **Balance Restoration**: User balances are restored to their state before the bet was placed
- **Exposure Adjustment**: User exposure is adjusted to remove the bet's impact
- **Status Change**: The bet's `isActive` flag is set to `false`
- **Audit Trail**: An action record is created with the admin user's details
- **Offer Status**: The bet's offer status remains unchanged

:::warning Balance Requirements

If users don't have sufficient balance to support the rollback operation, the request will fail with a detailed error showing which users have insufficient funds.

:::

#### Reactivating Bets

When a bet is set to active using the `POST /admin/bets/:betId/set-active` endpoint:

- **Financial Restoration**: The system restores the bet's financial impact
- **Balance Adjustment**: User balances are adjusted to reflect the bet's presence
- **Exposure Restoration**: User exposure is updated to include the bet
- **Status Change**: The bet's `isActive` flag is set to `true`
- **Audit Trail**: An action record is created with the admin user's details

### Override Outcome Management

The override outcome feature allows administrators to manually set bet outcomes, overriding the wager's natural outcome:

#### Behavior When Setting Override Outcomes

When using `POST /admin/bets/force-update-override-outcome`:

1. **Rollback First**: The system first rolls back the bet's current financial impact
2. **Update Override**: The `override_outcome` field is updated with the new value
3. **Status Logic**: 
   - If the bet is in "requesting" or "accepting" status and the override is not "undecided", the offer status changes to "expired"
   - If the override is set to "undecided" and the bet is "expired", the offer status is restored to "requesting" or "accepting"
4. **Reapply Impact**: New transaction actions are performed based on the updated outcome
5. **Audit Trail**: A detailed action record is created

#### Override Outcome Precedence

The system uses this precedence order for determining effective outcomes:
1. **Override Outcome** (highest priority) - manually set by admin
2. **Wager Outcome** - natural outcome from the wager resolution

## Transaction Processing

### Rollback Operations

All rollback operations follow a consistent pattern:

1. **Bet Trail Creation**: A bet trail record is created for audit purposes
2. **Conditional Processing**: Expired bets are handled differently than active bets
3. **Balance Validation**: The system checks if users have sufficient balance for rollback
4. **Transaction Recording**: All balance changes are recorded as transactions

### Transaction Action Operations

When performing transaction actions (the opposite of rollback):

1. **Balance Deduction**: User balances are reduced by the bet amount
2. **Exposure Tracking**: User exposure is updated to reflect potential losses
3. **Validation**: The system ensures users have sufficient balance
4. **Error Handling**: Detailed error messages are provided for insufficient balance scenarios

## Error Handling

### Balance Validation Errors

When users don't have sufficient balance for operations, the API returns structured error responses:

```json
{
  "status": 400,
  "error": "Unable to perform operation",
  "data": [
    {
      "message": "User balance not sufficient",
      "user": {
        "id": 2,
        "reference": "a2_user_Xhwz442NTj74z6F0",
        "current_balance": 0,
        "balance_after_action": -270000,
        "current_exposure": 0,
        "exposure_after_action": 0
      },
      "associated_bet_ids": [7]
    }
  ]
}
```

### State Validation

The API performs extensive state validation:

- **Bet Existence**: Ensures bets exist before operations
- **Status Checks**: Validates current bet status before state changes
- **Duplicate Actions**: Prevents duplicate operations (e.g., deactivating an already inactive bet)
- **Ownership Validation**: Ensures bets belong to the specified account

## Database Consistency

### Transaction Atomicity

All operations that modify multiple records use database transactions to ensure consistency:

- **Bet Updates**: Bet status changes are atomic
- **Balance Updates**: User balance changes are atomic
- **Audit Trails**: Bet trail creation is part of the same transaction

### Concurrency Handling

The system is designed to handle concurrent operations safely:

- **Row-Level Locking**: Database operations use appropriate locking
- **Consistency Checks**: State validation occurs within transactions
- **Rollback on Failure**: Failed operations are completely rolled back

## Performance Considerations

### Large Bet Operations

When dealing with bets that affect many users:

- **Batch Processing**: Operations are processed efficiently in batches
- **Memory Management**: Large datasets are handled with appropriate memory management
- **Timeout Handling**: Long-running operations have appropriate timeout handling

### Database Query Optimization

The API uses optimized queries:

- **Join Optimization**: Related data is fetched efficiently
- **Index Usage**: Database indexes are used for fast lookups
- **Selective Loading**: Only necessary data is loaded for operations

## Security Considerations

### Admin Access Control

Admin endpoints are protected with multiple layers:

- **Authentication**: Valid auth tokens are required
- **Authorization**: Admin privileges are validated
- **Account Scope**: Operations are scoped to specific accounts

### Audit Trail

All administrative actions create detailed audit trails:

- **User Tracking**: The admin user performing the action is recorded
- **Action Description**: Clear descriptions of what was done
- **Timestamp**: When the action was performed
- **Context**: Additional context about the operation

## Best Practices

### Error Handling

When integrating with the API:

1. **Check Response Status**: Always check HTTP status codes
2. **Parse Error Details**: Use the structured error data for user feedback
3. **Retry Logic**: Implement appropriate retry logic for transient failures
4. **Logging**: Log all API interactions for debugging

### State Management

When managing bet states:

1. **Verify State**: Always verify current state before operations
2. **Handle Failures**: Implement proper error handling for state changes
3. **Audit Integration**: Use audit trails for compliance and debugging
4. **Balance Monitoring**: Monitor user balances after operations

### Testing

When testing integrations:

1. **Test Error Scenarios**: Test insufficient balance scenarios
2. **State Transitions**: Test all possible state transitions
3. **Concurrent Operations**: Test concurrent access patterns
4. **Data Integrity**: Verify data consistency after operations
