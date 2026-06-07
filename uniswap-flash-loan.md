# Vulnerability Report: Flash Loan Attack Vector in Uniswap V3

## Summary
Uniswap V3's flash swap functionality can be exploited to manipulate prices across pools.

## Vulnerability Type
Flash Loan Attack

## Severity
Medium

## Affected Contracts
- UniswapV3Pool.sol

## Description
The flash swap feature allows borrowing tokens without collateral, which can be used to:
1. Manipulate pool prices
2. Extract value from other protocols
3. Perform sandwich attacks

## Impact
- Price manipulation across DEXs
- Value extraction from other protocols
- User losses from sandwich attacks

## Proof of Concept
```solidity
// 1. Flash borrow from Uniswap V3 pool
// 2. Manipulate price on target pool
// 3. Execute arbitrage or liquidation
// 4. Repay flash loan with profit
```

## Recommendation
1. Implement flash loan fees
2. Add price impact limits
3. Use TWAP for external price feeds

## References
- https://docs.uniswap.org/
- https://github.com/Uniswap/v3-core
