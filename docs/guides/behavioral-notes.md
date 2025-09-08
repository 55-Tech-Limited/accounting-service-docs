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
- **Status Change**: The bet's `is_active` flag is set to `false`
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
- **Status Change**: The bet's `is_active` flag is set to `true`
- **Audit Trail**: An action record is created with the admin user's details

### Bet Offer Cancellation

The cancel bet offer feature allows users to cancel their own open betting offers with flexible options:

#### Cancellation Types

**Complete Cancellation**:
- Cancels the entire bet amount
- Sets the original bet as inactive (`is_active: false`)
- Reduces user exposure by the full bet amount
- No new bet is created

**Partial Cancellation**:
- Cancels only a portion of the bet amount
- Sets the original bet as inactive
- Creates a new bet for the remaining amount
- Reduces user exposure by only the canceled amount
- The new bet maintains the same odds and wager reference

**Bulk Cancellation (`cancel_all`)**:
- Cancels all open bets for a specific wager
- Can be used to quickly close all positions for a particular wager
- Processes multiple bets in a single transaction

#### Cancellation Business Rules

1. **Status Restrictions**: Only bets with "requesting" status can be canceled
2. **Self-Ownership**: Users can only cancel their own bet offers
3. **Amount Validation**: Cancel amount cannot exceed the total available cancelable amount
4. **Atomic Operations**: All cancellations are processed within database transactions
5. **Exposure Management**: User exposure is automatically reduced by canceled amounts

#### Cancellation Process Flow

When a bet cancellation is processed:

1. **Validation**: System validates user ownership, bet status, and cancel amount
2. **Original Bet Update**: Original bet is marked as inactive with action metadata
3. **Bet Trail Creation**: Audit trail is created for the canceled bet
4. **Partial Handling**: If partial cancellation, a new bet is created for remaining amount
5. **Exposure Reduction**: User exposure is decreased by the canceled amount
6. **Response Generation**: Detailed response is returned with cancellation details

:::info Partial Cancellation Details

When partially canceling a bet:
- **Original bet**: Marked as inactive with canceled amount in action metadata
- **New bet**: Created with remaining amount, same odds, and new bet ID
- **Bet trails**: Both canceled and new bets have corresponding audit trails
- **Description format**: "Create remaining bet after partial cancellation (original: X, canceled: Y, remaining: Z)"

:::

### Bet Offer Acceptance and Partial Matching

The bet acceptance system supports sophisticated partial matching capabilities:

#### Partial Bet Acceptance

When a user accepts a bet offer, they don't need to accept the entire amount:

**Example Scenario**:
- User A offers to bet $100 at 2.0 odds
- User B wants to accept only $60 of that bet
- **Result**: 
  - Original bet becomes accepted for $60
  - New bet is created for remaining $40, still available for acceptance
  - User A's exposure remains $100 (original amount)
  - User B's exposure increases by $60 (their accepted amount)

#### Acceptance Process Flow

1. **Amount Calculation**: `amountToAccept = Math.min(requestingAmount, acceptingAmountRemaining)`
2. **Original Bet Update**: Updated with accepted portion and marked as "accepted"
3. **Remaining Bet Creation**: If `requestingAmount > amountToAccept`, new bet created for remainder
4. **Exposure Management**: Both users' exposures updated appropriately
5. **Bet Trail Recording**: Trails created for both accepted and newly created bets

#### Cross-Multiple-Bets Acceptance

The acceptance system can span across multiple existing bets:

- If User B wants to accept $150 but available bets are $100, $80, $70
- System processes oldest bets first until acceptance amount is fulfilled
- Results in multiple accepted bets and potentially one partial acceptance

:::info Acceptance Business Rules

- **Self-Betting Prevention**: Users cannot accept their own bet offers
- **Maximum Odds Filtering**: Only bets with odds ≤ `maximum_odds` are considered
- **FIFO Processing**: Bets are processed in creation order (oldest first)
- **Account Isolation**: Acceptance only matches bets within the same account

:::

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

### Bet Cancellation Errors

The cancel bet offer API provides specific error responses for various failure scenarios:

#### Common Cancellation Errors

**No Cancelable Bets Found**:
```json
{
  "status": 404,
  "error": "No cancelable bets found"
}
```

**Cancel Amount Exceeds Available**:
```json
{
  "status": 400,
  "error": "Cancel amount greater than the maximum cancelable amount of \"150\""
}
```

**Missing Cancellation Parameters**:
```json
{
  "status": 400,
  "error": "Either cancel_all must be true or cancel_amount must be provided"
}
```

**Invalid User Reference**:
```json
{
  "status": 404,
  "error": "Requesting user not found"
}
```

#### Cancellation-Specific Validations

The system performs several validations before processing cancellations:

- **Bet Ownership**: Ensures the user owns the bets they're trying to cancel
- **Bet Status**: Only "requesting" status bets can be canceled
- **Amount Limits**: Cancel amount cannot exceed total available cancelable amount
- **Parameter Consistency**: Validates that required parameters are provided
- **Wager Existence**: Ensures the specified wager exists and belongs to the account

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

### Bet Cancellation Best Practices

When implementing bet cancellation functionality:

1. **Pre-Cancellation Validation**: 
   - Query open bets using `GET /bets/open-bets` before attempting cancellation
   - Verify that bets are still in "requesting" status
   - Check available cancelable amounts before partial cancellations

2. **User Confirmation**:
   - Always implement user confirmation dialogs for cancellation actions
   - Clearly show the impact on user exposure and balance
   - For bulk cancellations (`cancel_all`), provide extra confirmation

3. **Partial Cancellation Handling**:
   - Track the new bet ID returned for partial cancellations
   - Update your UI to reflect the new bet with remaining amount
   - Consider the original bet as canceled/inactive

4. **Error Handling Strategies**:
   - Handle "No cancelable bets found" gracefully 
   - Provide clear feedback when cancel amounts exceed available amounts
   - Implement retry logic for network failures, but not for business logic errors

5. **Exposure Management**:
   - Update user exposure displays immediately after successful cancellations
   - Account for exposure reduction in risk management calculations
   - Monitor exposure changes for compliance and limit management

6. **Audit and Compliance**:
   - Log all cancellation attempts for audit purposes
   - Track user cancellation patterns for behavioral analysis
   - Maintain records of canceled vs. completed bets for reporting

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

### Testing Bet Cancellation Features

When testing bet cancellation integrations:

1. **Cancellation Scenarios**:
   - Test complete bet cancellation 
   - Test partial bet cancellation with various amounts
   - Test bulk cancellation using `cancel_all: true`
   - Test cancellation of multiple bets across different wagers

2. **Error Scenario Testing**:
   - Attempt to cancel non-existent bets
   - Try canceling already accepted bets
   - Test cancellation with invalid user references
   - Test partial cancellation with amounts exceeding available balance
   - Test cancellation without required parameters

3. **Edge Cases**:
   - Cancel the exact available amount (boundary condition)
   - Test rapid successive cancellations
   - Test cancellation of very small amounts
   - Test cancellation when user has exactly zero exposure

4. **State Consistency Verification**:
   - Verify user exposure is correctly reduced after cancellation
   - Check that original bets are marked as inactive
   - Ensure new bets are created correctly for partial cancellations
   - Validate bet trail creation for audit purposes

5. **Integration Testing**:
   - Test cancellation followed by new bet offers
   - Test cancellation in conjunction with bet acceptance flows
   - Verify behavior when wagers are decided after cancellation
   - Test cancellation impact on account-level reporting

6. **Performance Testing**:
   - Test bulk cancellation performance with many bets
   - Test cancellation response times under load
   - Test concurrent cancellation attempts by the same user

:::info Testing Data Setup

For comprehensive testing, create test scenarios with:
- Multiple users with various balance levels
- Multiple wagers in different states
- Bets with various amounts and odds
- Mix of requesting, accepted, and expired bets

:::

:::warning Race Condition Testing

Pay special attention to testing race conditions where:
- A bet is accepted while cancellation is being processed
- Multiple partial cancellations are attempted simultaneously
- Cancellation occurs while wager outcome is being decided

:::
