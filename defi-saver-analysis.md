# DeFi Saver — Exchange Logic Analysis

## Date
2026-06-08

## Analyst
nikos1005 / Empire of Agents

---

## Summary

DeFi Saver v3 contracts analyzed. 1114 Solidity files. Focus on exchange logic and flash loan handling.

## Architecture

### Core Contracts
- **RecipeExecutor** (422 lines): Core execution engine, handles recipe execution and flash loan callbacks
- **DFSExchangeCore**: Exchange logic with slippage checks and fee handling
- **DFSExchangeWithTxSaver**: Exchange with TxSaver integration for gas optimization
- **FLAction** (461 lines): Flash loan action supporting 10+ sources (Aave V2/V3, Balancer, Balancer V3, GHO, Maker, UniV3, Spark, Morpho Blue, CurveUSD)

### Registries (Owner-Controlled)
- **ExchangeAggregatorRegistry**: Controls which exchange addresses are trusted
- **WrapperExchangeRegistry**: Controls which wrapper contracts are trusted
- **TokenGroupRegistry**: Controls fee levels for token pairs

### Permission System
- **SafeModulePermission**: Manages Safe modules (only last 10 can be disabled)
- **DSProxyPermission**: Manages DSProxy permissions
- **SFProxyPermission**: Manages SF Proxy permissions

## Analysis

### Exchange Logic
1. **`_sell` function**: Takes fee, executes swap, checks slippage
   - Slippage check: `amountBought < wmul(minPrice, srcAmount)`
   - Fee handling: TokenGroupRegistry determines fee level

2. **`offChainSwap`**: Checks exchange address in registry before calling
   - Registry check prevents calling untrusted contracts

3. **`onChainSwap`**: Checks wrapper in registry before calling
   - Registry check prevents calling untrusted contracts

4. **`_injectExchangeData`**: Injects exchange data from TxSaverExecutor
   - Protected by registry checks on both exchangeAddr and wrapper

### Flash Loan Handling
1. **Callback validation**: All callbacks check `msg.sender` against known addresses
2. **Initiator check**: `onFlashLoan` checks `_initiator != address(this)`
3. **Balance verification**: All callbacks verify payback amounts match expectations

### Trust Model
- **Centralized**: Owner controls all registries
- **Registry-protected**: External calls validated against registries
- **TxSaver injection**: Protected by registry checks

## Potential Issues (All Owner-Controlled)

### 1. Registry Centralization
- Owner can add/remove exchange addresses and wrappers
- Owner can change fee levels for token pairs
- **Risk**: Owner key compromise could add malicious contracts
- **Mitigation**: Timelock on registry changes

### 2. SafeModulePermission Limitation
- Only last 10 modules can be disabled
- **Risk**: Old malicious modules can't be disabled through normal flow
- **Mitigation**: Owner can disable modules through other means

### 3. TxSaver Trust
- TxSaverExecutor injects exchange data
- **Risk**: Compromised TxSaverExecutor could inject malicious data
- **Mitigation**: Registry checks on injected addresses

## Conclusion

**No critical vulnerabilities found.**

The DeFi Saver codebase is well-structured with proper security checks:
- Registry checks prevent calling untrusted contracts
- Flash loan callbacks are properly validated
- Slippage checks protect against sandwich attacks
- Fee handling is transparent and auditable

The main risk is centralization (owner controls registries), but this is by design for operational flexibility.

## Recommendation

DeFi Saver has a mature, well-audited codebase. The $350K bounty is justified by the complexity and value at risk, but finding novel vulnerabilities requires deep protocol-specific knowledge beyond standard patterns.
