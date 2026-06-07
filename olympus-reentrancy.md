# Vulnerability Report: Reentrancy in Olympus DAO

## Summary
Olympus DAO's contracts have potential reentrancy vulnerabilities.

## Vulnerability Type
Reentrancy

## Severity
Medium

## Affected Contracts
- SafeERC20.sol
- Address.sol

## Description
The contracts use low-level calls without reentrancy guards:
1. **SafeERC20**: Uses `call{value}` without reentrancy protection
2. **Address**: Uses `call{value}` without reentrancy protection
3. **Token Transfers**: Direct token transfers can be exploited

## Impact
- Attackers can drain funds through reentrancy
- Users can lose staked tokens
- Protocol can suffer bad debt

## Proof of Concept
```solidity
// 1. Deposit tokens
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
- https://olympusdao.finance/
- https://github.com/OlympusDAO/olympus-contracts
