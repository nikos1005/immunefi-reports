# Nado Protocol — Smart Contract Analysis

## Date
2026-06-08

## Analyst
nikos1005 / Empire of Agents

## Bounty
Up to $500,000 on HackenProof

---

## Summary

Nado Protocol analyzed: 47 Solidity files across core contracts, libraries, and interfaces.

## Architecture

### Core Contracts
- **Clearinghouse** (~500 lines): Main contract for deposits, withdrawals, NLP minting/burning, liquidations
- **ClearinghouseLiq** (~380 lines): Liquidation logic with spread and non-spread support
- **OffchainExchange** (~600 lines): Order matching, fee calculation, isolated subaccounts
- **Endpoint** (~400 lines): Main entry point, transaction routing
- **EndpointTx** (~536 lines): Transaction execution, signature validation
- **SpotEngine** (~293 lines): Spot trading engine
- **PerpEngine** (~200 lines): Perpetual trading engine

### Libraries
- **MathSD21x18**: Fixed-point math with overflow checks
- **RiskHelper**: Risk calculations
- **ERC20Helper**: Token transfers

## Analysis

### 1. Clearinghouse - Liquidation Logic

**`liquidateSubaccountImpl`**:
- Checks `isUnderMaintenance` before liquidation
- Validates liquidation amounts via `_assertLiquidationAmount`
- Handles liquidation payments via `_handleLiquidationPayment`
- Finalizes subaccounts if fully liquidated

**Potential Issue - Health Check Timing**:
- Health is checked at the beginning of liquidation
- If health changes during liquidation (e.g., oracle update), the outcome could be affected
- **Severity**: Low - requires oracle manipulation during liquidation

### 2. OffchainExchange - Order Matching

**`matchOrders`**:
- Matches taker and maker orders at maker's price
- Validates signatures and order parameters
- Applies fees based on order type and amount

**Potential Issue - Frontrunning**:
- Orders are matched at maker's price
- If maker's price is manipulated, the match could be unfavorable
- **Severity**: Low - requires oracle manipulation

### 3. ClearinghouseLiq - Liquidation Price

**`getLiqPriceX18`**:
- Calculates liquidation price with penalty
- Penalty is `(weight - ONE) / 5` with minimum of 0.5%

**`getSpreadLiqPriceX18`**:
- Calculates spread liquidation price
- Penalty is `(weight - ONE) / 10` with minimum of 0.25%

**Observation**:
- Spread liquidations have lower minimum penalty (0.25%) than non-spread (0.5%)
- This could make spread liquidations less profitable for liquidators

### 4. Constants Analysis

- `LIQUIDATION_FEE_FRACTION`: 50% - High fee incentivizes liquidators
- `MIN_SPREAD_LIQ_PENALTY_X18`: 0.25% - Small minimum for spreads
- `MIN_NON_SPREAD_LIQ_PENALTY_X18`: 0.5% - Small minimum for non-spreads
- `NLP_LOCK_PERIOD`: 4 days - Long lock period
- `TAKER_FEE_ACCRUAL_RATE_X18`: -3bps - Negative fee (rebate)

## Findings

### Finding 1: Spread Liquidation Lower Penalty (Low)
**Description**: Spread liquidations have a lower minimum penalty (0.25%) than non-spread liquidations (0.5%). This could make spread liquidations less profitable, potentially reducing liquidator incentive.

**Impact**: Liquidators may avoid spread liquidations, leading to delayed liquidations and potential bad debt.

**Recommendation**: Consider equalizing minimum penalties or adjusting the fee structure.

### Finding 2: NLP Lock Period (Informational)
**Description**: NLP tokens have a 4-day lock period. This is a design choice, not a vulnerability.

**Impact**: Users must wait 4 days to redeem NLP tokens after minting.

**Recommendation**: No action needed - this is by design.

## Conclusion

**No critical vulnerabilities found.**

The Nado Protocol codebase is well-structured with proper security checks:
- Health checks before liquidations
- Signature validation for orders
- Overflow checks in math libraries
- Minimum penalty enforcement for liquidations

The main risks are:
1. Oracle manipulation during liquidations (Low)
2. Frontrunning in order matching (Low)

These are standard DeFi risks and not specific to Nado.

## Recommendation

The Nado Protocol appears to be a well-designed DEX with proper security measures. The $500K bounty is justified by the complexity and value at risk, but finding novel vulnerabilities requires deep protocol-specific knowledge beyond standard patterns.
