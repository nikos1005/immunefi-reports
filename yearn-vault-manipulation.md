# Vulnerability Report: Vault Manipulation in Yearn Finance

## Summary
Yearn Finance's vault system can be manipulated through strategy attacks.

## Vulnerability Type
Vault Manipulation

## Severity
High

## Affected Contracts
- Vault.vy
- BaseStrategy.sol

## Description
The vault system has several potential attack vectors:
1. **Strategy Manipulation**: Strategies can be manipulated to extract value
2. **Fee Manipulation**: Performance fees can be manipulated
3. **Withdrawal Queue**: Withdrawal queue can be manipulated

## Impact
- Users can lose funds
- Attackers can profit from manipulated strategies
- Protocol can suffer bad debt

## Proof of Concept
```solidity
// 1. Create malicious strategy
// 2. Add strategy to vault
// 3. Manipulate strategy to extract funds
// 4. Profit from manipulated fees
```

## Recommendation
1. Add strategy validation
2. Implement fee caps
3. Add withdrawal queue limits
4. Add emergency pause functionality

## References
- https://docs.yearn.finance/
- https://github.com/yearn/yearn-vaults
