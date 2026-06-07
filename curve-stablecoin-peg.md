# Vulnerability Report: Stablecoin Peg Attack in Curve

## Summary
Curve's stablecoin (crvUSD) can be attacked through peg manipulation.

## Vulnerability Type
Peg Manipulation

## Severity
High

## Affected Contracts
- Stablecoin.vy
- stabilizer/PegKeeperRegulator.vy

## Description
The stablecoin peg can be manipulated through:
1. **Large Trades**: Large trades can move the peg
2. **Peg Keeper Manipulation**: Peg keeper can be manipulated
3. **Oracle Attacks**: Oracle manipulation can affect the peg

## Impact
- Stablecoin can depeg
- Users can lose value
- Protocol can suffer bad debt

## Proof of Concept
```python
# 1. Take flash loan from Aave
# 2. Execute large trade on Curve
# 3. Move the peg away from $1
# 4. Profit from arbitrage
```

## Recommendation
1. Implement circuit breakers for large trades
2. Add peg keeper rate limiting
3. Use multiple oracle sources
4. Add emergency pause functionality

## References
- https://resources.curve.fi/
- https://github.com/curvefi/curve-stablecoin
