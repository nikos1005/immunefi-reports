# Parallel Protocol — Deep Security Analysis

## Date
2026-06-08

## Analyst
nikos1005 / Empire of Agents

## Bounty Program
Immunefi — https://immunefi.com/bug-bounty/parallel/
Max Bounty: $250K critical (10% of funds)

---

## Protocol Overview

**Parallel** is a modular stablecoin protocol allowing creation of over-collateralized, decentralized stablecoins (EUR, USD, CHF, ETH, BTC, etc.).

**Tech Stack:**
- Solidity 0.8.28
- Diamond Pattern (EIP-2535)
- Ethereum + Polygon PoS
- Chainlink oracles + internal oracle types
- Fork of Angle Protocol's Transmuter

**Audit History:**
- Bailsec + Certora (Jan 2025)
- Certora did formal verification of mathematical invariants
- BUT: Certora does NOT cover composition logic between modules

**Key Risk Factors:**
- Modular DAO architecture — modules can be added/removed by DAO
- External manager pattern — managed collateral delegated to external contracts
- Cross-chain deployment (Ethereum + Polygon)
- Relatively new (Oct 2025 launch)

---

## Architecture Analysis

### Diamond Pattern (EIP-2535)
- **DiamondProxy**: Main entry point, delegates calls to facets
- **Facets**: Swapper, Redeemer, Surplus, Getters, SettersGovernor, SettersGuardian, RewardHandler
- **Libraries**: LibOracle, LibManager, LibSurplus, LibHelpers, LibWhitelist, LibDiamond

### Key Components

#### 1. Swapper (646 lines)
- Handles mint (collateral → stablecoin) and burn (stablecoin → collateral) operations
- Uses `nonReentrant` modifier
- Checks hard caps after minting
- Supports Permit2 for gasless approvals

#### 2. LibOracle (270 lines)
- Complex oracle system with multiple read types:
  - CHAINLINK_FEEDS: Chainlink price feeds with stale period checks
  - STABLE: Returns BASE_18 (for stablecoins)
  - NO_ORACLE: Returns base value
  - WSTETH, CBETH, RETH, SFRXETH: LST-specific oracles
  - MORPHO_ORACLE: Morpho oracle integration
  - EXTERNAL: External oracle contract
  - MAX: Maximum value (used as target price)
- `readMint`: Returns min(oracleValue, targetPrice)
- `readBurn`: Has "firewall" for small deviations

#### 3. LibManager (105 lines)
- Handles managed collateral (strategies)
- External managers are trusted contracts
- Functions: transferRecipient, release, invest, totalAssets, maxAvailable
- **Risk**: External managers could return incorrect values

#### 4. Surplus (72 lines)
- Processes surplus collateral
- Swaps collateral for stablecoins
- Distributes to payees (or burns if payee is address(0))

---

## Attack Surface Analysis

### 1. Oracle Manipulation
**Status**: Partially mitigated

- Chainlink feeds have stale period checks ✅
- `readMint` returns min(oracle, target) — protects against overvaluation ✅
- `readBurn` has deviation firewall ✅
- **Risk**: If target price is set too low via governance, minting could overvalue collateral

### 2. External Manager Trust
**Status**: Potential risk

- External managers are called via `abi.decode(data, (IManager))`
- `totalAssets()` could return inflated values → overestimated surplus
- `release()` could return less than expected
- **Mitigation**: DAO governance approves managers
- **Risk**: If manager contract is compromised or malicious

### 3. Diamond Pattern Vulnerabilities
**Status**: Standard risks

- Storage collisions between facets (mitigated by LibStorage)
- Function selector clashes (mitigated by DiamondCut)
- Upgrade risks (controlled by governance)

### 4. Cross-Chain State Divergence
**Status**: Not analyzed

- Ethereum + Polygon deployments
- State could diverge between chains
- Bridge-related risks not in scope

---

## Specific Findings

### Finding 1: Potential Surplus Overestimation via External Manager
**Severity**: Medium
**Location**: LibSurplus._computeCollateralSurplus

```solidity
if (collatInfo.isManaged > 0) {
    (, currentCollateralBalance) = LibManager.totalAssets(collatInfo.managerData.config);
}
```

**Description**: For managed collateral, the surplus calculation relies on `LibManager.totalAssets()`, which calls an external manager contract. If the external manager returns an inflated value, the surplus would be overestimated, potentially allowing more collateral to be released than the underlying assets are worth.

**Impact**: If external manager is compromised or malicious, could drain protocol funds through inflated surplus claims.

**Mitigation**: 
- DAO governance controls which managers are approved
- Manager contracts should be audited separately
- Consider adding sanity checks on surplus amounts

**PoC (Pseudocode)**:
```
1. Attacker compromises or controls external manager contract
2. External manager returns inflated totalAssets() value
3. processSurplus() calculates inflated collateralSurplus
4. Collateral is released based on inflated surplus
5. Attacker receives more collateral than underlying assets
```

**Recommendation**: Add a maximum surplus percentage cap or time-weighted average check on external manager values.

---

### Finding 2: Burn Ratio Deviation Threshold Risk
**Severity**: Low
**Location**: LibOracle.readBurn

```solidity
if (oracleValue * BASE_18 < targetPrice * (BASE_18 - burnRatioDeviation)) {
    ratio = (oracleValue * BASE_18) / targetPrice;
} else if (oracleValue < targetPrice) {
    oracleValue = targetPrice;
}
```

**Description**: The `burnRatioDeviation` parameter controls when burns are penalized. If set too high, burns could happen at 1:1 even when the oracle value is significantly below target, allowing users to burn stablecoins for more collateral than they're worth.

**Impact**: Could lead to unfair burns if oracle value drops significantly but deviation threshold is too lenient.

**Mitigation**: 
- Governance controls `burnRatioDeviation` parameter
- Should be set conservatively based on asset volatility

**Recommendation**: Document recommended ranges for `burnRatioDeviation` based on asset volatility. Consider adding governance timelock for parameter changes.

---

### Finding 3: Missing Slippage Protection in Surplus Release
**Severity**: Low
**Location**: Surplus.processSurplus

```solidity
uint256 minExpectedAmount = LibSurplus._minExpectedAmount(stableSurplus, ts.slippageTolerance[collateral]);
```

**Description**: The `slippageTolerance` is per-collateral and set by governance. If set too low (e.g., 0), there's no slippage protection. If set too high, users could receive significantly less than expected.

**Impact**: In extreme market conditions, surplus processing could result in significant value loss.

**Mitigation**: 
- Governance controls slippage tolerance
- Consider minimum/maximum bounds on slippage tolerance

**Recommendation**: Add bounds checking on slippageTolerance parameter (e.g., min 0.1%, max 5%).

---

## Code Quality Assessment

### Strengths
1. Well-structured Diamond pattern with proper storage separation
2. Comprehensive access control (AccessManaged)
3. Reentrancy protection on critical functions
4. Chainlink stale period checks
5. Formal verification by Certora (mathematical invariants)

### Weaknesses
1. Heavy reliance on external managers without on-chain verification
2. Complex oracle configuration (6+ read types)
3. No on-chain monitoring or circuit breakers
4. Cross-chain deployment adds complexity

---

## Comparison with Angle Protocol

Parallel is a fork of Angle Protocol's Transmuter. Key differences:
- Angle is being sunset (redeem before March 2027)
- Parallel adds modular DAO architecture
- Parallel adds support for more oracle types (Morpho)
- Parallel has additional collateral types

**Note**: Angle Protocol's Transmuter has not had any known exploits, which is a positive signal for the codebase.

---

## Recommendations for Bug Bounty Hunters

1. **Focus on External Manager Integration**: Test edge cases with malicious/manipulated manager contracts
2. **Oracle Manipulation**: Try to manipulate oracle reads through governance or flash loans
3. **Module Interaction Bugs**: Test adding/removing modules mid-operation
4. **Cross-Chain Edge Cases**: Test state divergence between Ethereum and Polygon
5. **Diamond Storage Collisions**: Look for storage layout issues between facets

---

## Summary

| Finding | Severity | Status |
|---------|----------|--------|
| Surplus Overestimation via External Manager | Medium | Documented |
| Burn Ratio Deviation Threshold Risk | Low | Documented |
| Missing Slippage Protection in Surplus Release | Low | Documented |

**Overall Assessment**: Parallel Protocol is a well-structured modular stablecoin protocol. The main risk areas are external manager trust and oracle configuration. No critical vulnerabilities found in this analysis, but the medium-severity finding regarding external manager trust warrants further investigation.

**Next Steps**:
1. Deep dive into external manager contracts (if public)
2. Test with Foundry fork simulations
3. Analyze cross-chain state synchronization
