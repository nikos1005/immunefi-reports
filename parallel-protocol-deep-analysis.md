# Parallel Protocol — Deep Analysis Update

## Date
2026-06-08

## Analyst
nikos1005 / Empire of Agents

---

## Critical Finding: Non-Monotonic Redemption Fee Curve

### Description
The redemption fee curve (`xRedemptionCurve` / `yRedemptionCurve`) is non-monotonic, creating an unusual fee structure where users pay MORE fees at intermediate collateralization levels than at extreme levels.

### Configuration (Mainnet)
```
xRedemptionCurve: [750000000, 850000000, 950000000, 970000000]
yRedemptionCurve: [995000000, 950000000, 950000000, 995000000]
```

### Fee Structure
| Collateral Ratio | Penalty Factor | Effective Fee |
|-----------------|---------------|---------------|
| 75% | 99.5% | **0.5%** |
| 80% | ~97.25% | ~2.75% |
| 85% | 95% | **5%** |
| 90% | 95% | **5%** |
| 95% | 95% | **5%** |
| 97% | 99.5% | **0.5%** |
| ≥100% | 99.5% | **0.5%** |

### Impact
1. **Counterintuitive behavior**: Users pay 5x more fees at 85-95% collateralization than at 75% or 97%
2. **Potential arbitrage**: If collateral ratio moves from 96% → 97%, redemption fee drops from ~5% to 0.5%
3. **Game theory issue**: Rational actors may time redemptions around collateral ratio thresholds

### PoC (Scenario)
```
1. Protocol collateral ratio is at 96%
2. User wants to redeem 1000 USDp
3. At 96%, fee is ~5% → user gets ~950 USDp worth of collateral
4. User waits for collateral ratio to reach 97% (or contributes to increase it)
5. At 97%, fee is 0.5% → user gets ~995 USDp worth of collateral
6. User saves 45 USDp (4.5%) by timing the redemption
```

### Severity
**Medium** — Economic inefficiency, not direct fund loss

### Recommendation
Consider using a monotonic increasing penalty curve as collateral ratio decreases:
```
x: [75%, 85%, 95%, 97%]
y: [95%, 90%, 85%, 80%]  // Higher penalty at lower ratios
```

---

## Finding 2: External Manager Not Currently Used

### Description
The code supports external managers (`ManagerType.EXTERNAL`) for managed collateral, but **no collaterals on mainnet are configured as managed**. All 4 collaterals use `isManaged = 0`.

### Implication
- The external manager attack surface I identified earlier is **theoretical, not currently exploitable**
- If managers are added in the future, they could introduce vulnerabilities
- The `totalAssets()` function could return inflated values, affecting surplus calculations

### Severity
**Informational** — No current risk, but future risk if managers are added

---

## Finding 3: Morpho Oracle Dependency

### Description
One collateral (`0xcf62F905562626CfcDD2261162a51fd02Fc9c5b6`) uses a Morpho Oracle (`0x0cb7dee76aa916a3666f50b044f16051c325b3e2`) with `normalizationFactor = 1000000`.

### Formula
```
oracleValue = MorphoOracle.price() / 1000000
```

### Risk
- Parallel's collateral valuation depends on Morpho Oracle's price feed
- If Morpho Oracle is compromised or returns incorrect prices, Parallel's collateral valuation would be affected
- This is an external dependency risk

### Severity
**Low** — Depends on external protocol security

---

## Finding 4: Deviation Threshold Analysis

### Description
The deviation thresholds for oracle reads are very small:
- `userDeviation`: 500000000000000 (0.05%)
- `burnRatioDeviation`: 500000000000000 (0.05%)

### Analysis
- For `readMint`: If oracle value is within 0.05% of target price, it's set to target price
- For `readBurn`: If oracle value is within 0.05% below target, ratio is set to 1:1

### Impact
- Very tight thresholds mean small oracle manipulations could affect collateral valuation
- However, 0.05% is extremely small and unlikely to be exploitable in practice

### Severity
**Informational** — Well-calibrated thresholds

---

## Summary

| Finding | Severity | Status |
|---------|----------|--------|
| Non-Monotonic Redemption Fee Curve | Medium | Documented |
| External Manager Not Currently Used | Informational | Documented |
| Morpho Oracle Dependency | Low | Documented |
| Deviation Threshold Analysis | Informational | Documented |

## Recommendation
The most actionable finding is the non-monotonic redemption fee curve. This should be reviewed by the protocol team to ensure it matches their intended design. If not intentional, it should be corrected to a monotonic curve.
