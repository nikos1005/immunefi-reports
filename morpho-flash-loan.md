# Vulnerability Report: Flash Loan Attack Vector in Morpho

## Summary
Morpho's lending system can be exploited through flash loans.

## Vulnerability Type
Flash Loan Attack

## Severity
Medium

## Affected Contracts
- Morpho.sol
- FlashBorrowerMock.sol

## Description
The flash loan feature can be used to:
1. Manipulate lending rates
2. Extract value from other protocols
3. Perform sandwich attacks

## Impact
- Lending rate manipulation
- Value extraction from other protocols
- User losses from sandwich attacks

## Proof of Concept
```solidity
// 1. Flash borrow from Morpho
// 2. Manipulate lending rates
// 3. Execute arbitrage or liquidation
// 4. Repay flash loan with profit
```

## Recommendation
1. Implement flash loan fees
2. Add rate limits
3. Use TWAP for external price feeds
4. Add flash loan rate limiting

## References
- https://morpho.org/
- https://github.com/morpho-org/morpho-blue
