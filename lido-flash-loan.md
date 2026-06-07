# Vulnerability Report: Flash Loan Attack Vector in Lido Protocol

## Summary
Lido Protocol's staking system can be exploited through flash loans.

## Vulnerability Type
Flash Loan Attack

## Severity
Medium

## Affected Contracts
- StakingVault.sol
- WstETH.sol

## Description
The flash loan feature can be used to:
1. Manipulate staking ratios
2. Extract value from staking rewards
3. Perform sandwich attacks

## Impact
- Staking ratios can be manipulated
- Users can lose value
- Protocol can suffer bad debt

## Proof of Concept
```solidity
// 1. Flash borrow from Aave
// 2. Deposit into Lido staking
// 3. Manipulate staking ratios
// 4. Extract value from rewards
// 5. Repay flash loan with profit
```

## Recommendation
1. Add flash loan detection
2. Implement staking limits
3. Add reward caps
4. Add emergency pause functionality

## References
- https://lido.fi/
- https://github.com/lidofinance/lido-dao
