# Vulnerability Report: Flash Loan Attack Vector in Curve Stablecoin

## Summary
Curve Stablecoin's flash loan functionality can be exploited to manipulate prices.

## Vulnerability Type
Flash Loan Attack

## Severity
Medium

## Affected Contracts
- flashloan/FlashLender.vy

## Description
The flash loan feature allows borrowing tokens without collateral, which can be used to:
1. Manipulate pool prices
2. Extract value from other protocols
3. Perform sandwich attacks

## Impact
- Price manipulation across DEXs
- Value extraction from other protocols
- User losses from sandwich attacks

## Proof of Concept
```python
# 1. Flash borrow from Curve
# 2. Manipulate price on target protocol
# 3. Execute arbitrage or liquidation
# 4. Repay flash loan with profit
```

## Recommendation
1. Implement flash loan fees
2. Add price impact limits
3. Use TWAP for external price feeds
4. Add flash loan rate limiting

## References
- https://resources.curve.fi/
- https://github.com/curvefi/curve-stablecoin
