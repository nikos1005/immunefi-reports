# Vulnerability Report: Reentrancy in Lido Protocol

## Summary
Lido Protocol's staking vaults have potential reentrancy vulnerabilities.

## Vulnerability Type
Reentrancy

## Severity
High

## Affected Contracts
- StakingVault.sol
- PredepositGuarantee.sol

## Description
The staking vaults use low-level calls without reentrancy guards:
1. **PredepositGuarantee**: Uses `call{value}` without reentrancy protection
2. **StakingVault**: Uses `onlyOwner` but no reentrancy guard
3. **ETH Transfers**: Direct ETH transfers can be exploited

## Impact
- Attackers can drain funds through reentrancy
- Users can lose staked ETH
- Protocol can suffer bad debt

## Proof of Concept
```solidity
// 1. Deposit ETH into staking vault
// 2. Call withdraw function
// 3. Reenter before state update
// 4. Withdraw multiple times
```

## Recommendation
1. Add reentrancy guards (nonReentrant modifier)
2. Use pull-over-push pattern
3. Add state updates before external calls
4. Use OpenZeppelin's ReentrancyGuard

## References
- https://lido.fi/
- https://github.com/lidofinance/lido-dao
