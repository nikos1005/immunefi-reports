# Vulnerability Report: Oracle Manipulation in Lido Protocol

## Summary
Lido Protocol's oracle system can be manipulated through price attacks.

## Vulnerability Type
Oracle Manipulation

## Severity
High

## Affected Contracts
- AccountingOracle.sol
- OracleReportSanityChecker.sol

## Description
The oracle system has several potential attack vectors:
1. **Price Manipulation**: Oracle prices can be manipulated
2. **Report Manipulation**: Oracle reports can be manipulated
3. **Sanity Check Bypass**: Sanity checks can be bypassed

## Impact
- Users can be unfairly liquidated
- Attackers can profit from manipulated prices
- Protocol can suffer bad debt

## Proof of Concept
```solidity
// 1. Manipulate oracle price
// 2. Execute swaps at manipulated prices
// 3. Profit from price differences
```

## Recommendation
1. Use multiple oracle sources
2. Implement TWAP (Time-Weighted Average Price)
3. Add oracle delay
4. Add circuit breakers

## References
- https://lido.fi/
- https://github.com/lidofinance/lido-dao
