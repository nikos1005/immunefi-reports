# Vulnerability Report: Bond Manipulation in Olympus DAO

## Summary
Olympus DAO's bond depository can be manipulated through price attacks.

## Vulnerability Type
Bond Manipulation

## Severity
High

## Affected Contracts
- BondDepository.sol
- PriceConverterOracleWrapper.sol

## Description
The bond depository has several potential attack vectors:
1. **Price Manipulation**: Bond prices can be manipulated
2. **Oracle Manipulation**: Oracle prices can be manipulated
3. **Timing Attacks**: Bond timing can be exploited

## Impact
- Users can lose value from manipulated bonds
- Attackers can profit from price differences
- Protocol can suffer bad debt

## Proof of Concept
```solidity
// 1. Manipulate oracle price
// 2. Buy bonds at manipulated price
// 3. Wait for price correction
// 4. Redeem bonds for profit
```

## Recommendation
1. Use multiple oracle sources
2. Implement TWAP (Time-Weighted Average Price)
3. Add bond price caps
4. Add emergency pause functionality

## References
- https://olympusdao.finance/
- https://github.com/OlympusDAO/olympus-contracts
