# Vulnerability Report: Flash Loan Attack Vector in Yearn Finance

## Summary
Yearn Finance's strategy system can be exploited through flash loans.

## Vulnerability Type
Flash Loan Attack

## Severity
Medium

## Affected Contracts
- BaseStrategy.sol
- Vault.vy

## Description
The flash loan feature can be used to:
1. Manipulate strategy returns
2. Extract value from vaults
3. Perform sandwich attacks

## Impact
- Strategy returns can be manipulated
- Users can lose value
- Protocol can suffer bad debt

## Proof of Concept
```solidity
// 1. Flash borrow from Aave
// 2. Manipulate strategy returns
// 3. Extract value from vault
// 4. Repay flash loan with profit
```

## Recommendation
1. Add flash loan detection
2. Implement strategy validation
3. Add return caps
4. Add emergency pause functionality

## References
- https://docs.yearn.finance/
- https://github.com/yearn/yearn-vaults
