# Issue #831: Add stake and unstake cases to buildDescription()

## Summary

Fixed event description generation for stake and unstake operations on DeFi staking protocols like Blend by ensuring the event decoder correctly formats human-readable descriptions for these event types.

## Problem

Staking events (stake/unstake) on protocols like Blend were falling through to `genericDescription()`, resulting in generic function-call descriptions instead of rich, human-readable messages that clearly convey the staking action and amounts involved.

## Solution

The implementation already included the necessary `case 'stake'` and `case 'unstake'` branches in the `buildDescription()` function in `indexer/src/decoder.js` (lines 146-153), which generate appropriately formatted descriptions:

- **stake**: `Address GA… staked 500 BLND on ContractName`
- **unstake**: `Address GA… unstaked 500 BLND on ContractName`

However, the unit tests for these cases were broken: they referenced contract IDs (`C14`, `C15`, `C16`) that were not defined in the test contract IDs array.

### Changes Made

**File: `indexer/test/decoder.test.js`**
- Extended the contract IDs array (line 52) from 15 items to 18 items, adding definitions for `C14`, `C15`, and `C16`
- These IDs are now available for the stake, unstake, deposit, and withdraw test cases that depend on them

## Testing

The existing unit tests for stake and unstake were already comprehensive:
- Test for `'stake'`: Verifies description includes "staked", amount (200), token (BLND), and contract name (Blend)
- Test for `'unstake'`: Verifies description includes "unstaked", amount (150), token (BLND), and contract name (Blend)

Both tests now execute properly with the contract ID definitions in place.

## Technical Details

The formatter uses:
- `fmt()` function to abbreviate addresses (GA… format) for brevity
- Parameter extraction pattern: `[from, amount, token]` from the event topics
- Null-coalescing on token to handle optional asset symbols: `${token ?? ""}`
