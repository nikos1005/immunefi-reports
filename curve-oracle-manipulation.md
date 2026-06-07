# Vulnerability Report: Oracle Manipulation in Curve Stablecoin

## Summary
Curve Stablecoin's oracle system can be manipulated through price oracle attacks.

## Vulnerability Type
Price Oracle Manipulation

## Severity
High

## Affected Contracts
- price_oracles/AggregateStablePrice2.vy
- price_oracles/CryptoWithStablePriceFrxethN.vy

## Description
The Curve oracle system has several potential attack vectors:
1. **Stable Price Oracle**: The stable price oracle can be manipulated through large trades
2. **Crypto Price Oracle**: The crypto price oracle can be manipulated through flash loans
3. **Admin Functions**: Admin functions can be called to manipulate prices

## Impact
- Users can be unfairly liquidated
- Attackers can profit from manipulated prices
- Protocol can suffer bad debt

## Proof of Concept
```python
# 1. Take flash loan from Aave
# 2. Manipulate stable price oracle on Curve
# 3. Use manipulated price to affect liquidation thresholds
# 4. Profit from unfair liquidations
```

## Recommendation
1. Implement multiple oracle sources with consensus
2. Add circuit breakers for extreme price movements
3. Use TWAP (Time-Weighted Average Price) instead of spot price
4. Add oracle delay to prevent manipulation

## References
- https://resources.curve.fi/
- https://github.com/curvefi/curve-stablecoin
