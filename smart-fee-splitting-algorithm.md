# Smart Fee Splitting Algorithm Documentation

## Overview

The Smart Fee Splitting Algorithm is a sophisticated financial optimization system that automatically calculates optimal fee distribution for third-party delivery orders. It ensures restaurants earn at least as much as they would on competing platforms while providing transparent, distance-based pricing to customers through the AI agent.

**Location**: `apps/server/src/application/use-cases/order-draft/check-and-update/processors/fees/smart-fee-splitting-algorithm.ts`

## Core Objectives

The algorithm is designed to achieve:

1. **Restaurant profitability** - Ensure restaurants never earn less than on Uber Eats
2. **Distance-based pricing** - Delivery fees determined solely by distance, never adjusted post-calculation
3. **Transparent cost structure** - Clear breakdown of all fees and their purposes
4. **Customer predictability** - Consistent pricing that customers can understand
5. **Platform sustainability** - Fair compensation for the platform's services

## Algorithm Philosophy: Fixed-Delivery Approach

### Key Principle: "No Cap-Aware, No Post-Adjustments"

The algorithm follows a **Fixed-Delivery** approach where:
- **Delivery fee depends ONLY on distance** and is never modified later
- **No cap logic** - Service fees can exceed informal targets on small baskets at long distances
- **No post-tuning** - Delivery fees are calculated once and remain fixed
- **Transparent pricing** - All fees serve clear, explainable purposes

## Algorithm Parameters

### Configurable Parameters

These parameters are stored in `location.deliverySettings.feeSharing.smartParameters`:

| Parameter | Default | Description | Impact |
|-----------|---------|-------------|--------|
| `MENU_UPLIFT` | 0.20 | Uber menu markup vs restaurant prices | Higher = more restaurant protection |
| `UBER_COMMISSION` | 0.30 | Uber commission on uplifted menu | Restaurant economics baseline |
| `TARGET_LIFT` | 0.00 | Additional percentage vs Uber baseline | Higher = restaurant earns more |
| `COVERAGE_FRACTION` | 1.00 | Fraction of safe cover to use | Lower = more service fee, less subsidy |

### System Constants

These are defined in the codebase:

| Constant | Value | Purpose |
|----------|-------|---------|
| `PLATFORM_FEE` | 1.00 € | Chataigne fee paid by restaurant |
| `STRIPE_PCT` | 0.015 | Stripe variable percentage |
| `STRIPE_FIXED` | 0.25 € | Stripe fixed fee |
| `LADDER_FLOOR` | 1.99 € | Delivery ladder starting price |
| `LADDER_STEP` | 0.50 € | Delivery fee increment steps |

## Algorithm Flow

### 1. Input Collection

```typescript
const { menuUplift, coverageFraction, targetLift, uberCommission } = 
  state.context.location.deliverySettings.feeSharing.smartParameters;
const courierCostTTC = state.draft.realDeliveryFees;
const displayedDeliveryFee = state.draft.displayedDeliveryFees;
const basket = state.draft.computeTotal(state.context.location.whatsappCatalog).amount;
const platformFee = state.context.location.billingSettings?.whatsappDeliveryOrderFee?.amount ?? 1;
```

### 2. Shortfall Calculation

```typescript
const shortfall = Math.max(0, courierCostTTC - displayedDeliveryFee);
```

**Shortfall** represents the gap between actual delivery cost and what customers pay.

### 3. Coverage Coefficient Calculation

```typescript
const coeff = 1 - (1 - uberCommission) * (1 + menuUplift) * (1 + targetLift);
```

**Coverage Coefficient** determines how much restaurants can safely subsidize delivery:
- With 30% commission & 20% uplift: `coeff = 1 - 0.70 * 1.20 = 0.16`
- This means restaurants can subsidize up to 16% of basket value while matching Uber profitability

### 4. Safe Cap Determination

```typescript
const safeCap = Math.max(0, coeff * basket - platformFee);
```

**Safe Cap** is the maximum amount restaurants can subsidize without earning less than on Uber.

### 5. Cover Usage Calculation

```typescript
const coverUsed = Math.min(shortfall, coverageFraction * safeCap);
```

**Cover Used** is the actual restaurant subsidy applied, limited by safe cap and coverage fraction.

### 6. Fee Distribution

```typescript
const stripe = StripeFeeCalculatorService.calculateProcessingFee(
  new Price(basket, state.context.location.currency)
);
const variable = Math.max(0, shortfall - coverUsed);
const service = Math.round(stripe + variable);
const total = Math.round(basket + displayedDeliveryFee + service);
```

**Service Fee** combines:
- **Stripe processing fee** - Payment processing cost
- **Variable component** - Remaining shortfall not covered by restaurant

### 7. Restaurant Economics

```typescript
const restNetChataigne = Math.round(basket - platformFee - coverUsed);
const restNetUber = Math.round((1 - uberCommission) * (1 + menuUplift) * basket);
const deltaVsUber = Math.round(restNetChataigne - restNetUber);
```

**Restaurant Net Earnings**:
- **Chataigne**: Basket minus platform fee minus delivery subsidy
- **Uber**: Basket times (1 - commission) times (1 + uplift)
- **Delta**: Guaranteed to be ≥ 0 by design

## Return Structure

The algorithm returns a comprehensive result object used throughout the system:

### Customer Lines
```typescript
customerLines: {
  subTotal: basket,      // Order subtotal
  delivery: displayedDeliveryFee,  // Distance-based delivery fee
  service: service,      // Payment + variable fees
  total: total,         // Final amount charged
}
```

### Transparency Data
```typescript
transparency: {
  courierActual: courierCostTTC,     // Real delivery cost
  shortfall: shortfall,              // Cost gap to fill
  restaurantCovers: coverUsed,       // Restaurant subsidy amount
  stripeInService: stripe,           // Payment processing part
  variableInService: variable,       // Remaining shortfall part
}
```

### Restaurant Economics
```typescript
restaurant: {
  netChataigne: restNetChataigne,    // Earnings on Chataigne
  netUber: restNetUber,              // Comparable Uber earnings
  deltaVsUber: deltaVsUber,          // Advantage vs Uber (≥ 0)
}
```

## Integration with Order Draft Validation

### validationData.splitAlgorithmResult

The complete algorithm result is stored in `state.validationData.splitAlgorithmResult`, making it available to:

1. **AI Agent responses** - For explaining fees to customers
2. **Order confirmation** - Final pricing breakdown
3. **Restaurant dashboard** - Economics transparency
4. **Analytics** - Fee optimization insights
5. **Customer support** - Detailed fee explanations

### Usage in Fee Processor

```typescript
if (smartFeesResult) {
  state.validationData.splitAlgorithmResult = smartFeesResult;
}
```

The fee processor stores the complete result, enabling downstream systems to access:
- Customer-facing pricing
- Internal cost breakdowns
- Restaurant profitability metrics
- Transparency data for support

## AI Agent Integration

### Customer Fee Explanations

The AI agent uses `splitAlgorithmResult` to explain fees:

```
"Your delivery fee of €3.49 is based on distance. The service fee of €1.25 covers payment processing and any additional delivery costs."
```

### Transparency Features

When customers ask about fees, the AI agent can access:
- Exact delivery cost breakdown
- Service fee component explanation
- Platform fee transparency
- Restaurant support information

### Dynamic Updates

As orders change, the algorithm recalculates:
- New basket totals affect safe cap
- Address changes affect delivery distance
- Item modifications update subsidy calculations
- Payment method switches update processing fees

## Example Calculation

### Scenario
- **Basket**: €25.00
- **Distance**: 3.5 km
- **Courier cost**: €6.50
- **Displayed delivery fee**: €2.99
- **Platform settings**: 30% commission, 20% uplift, 0% target lift

### Calculation Steps

1. **Shortfall**: €6.50 - €2.99 = €3.51
2. **Coverage coefficient**: 1 - (1-0.30) × (1+0.20) × (1+0.00) = 0.16
3. **Safe cap**: 0.16 × €25.00 - €1.00 = €3.00
4. **Cover used**: min(€3.51, 1.00 × €3.00) = €3.00
5. **Stripe fee**: €0.62 (1.5% + €0.25)
6. **Variable**: €3.51 - €3.00 = €0.51
7. **Service fee**: €0.62 + €0.51 = €1.13
8. **Total**: €25.00 + €2.99 + €1.13 = €29.12

### Result Breakdown

**Customer sees:**
- Subtotal: €25.00
- Delivery: €2.99  
- Service: €1.13
- **Total: €29.12**

**Restaurant economics:**
- Chataigne net: €25.00 - €1.00 - €3.00 = €21.00
- Uber net: (1-0.30) × (1+0.20) × €25.00 = €21.00
- Delta: €0.00 (matches Uber exactly)

## Edge Cases and Handling

### Small Baskets, Long Distances

- Service fees may exceed informal targets
- Transparency prevents customer confusion
- Restaurant protection remains priority
- AI agent can explain cost structure

### Large Baskets, Short Distances

- Restaurant covers most delivery costs
- Service fees approach minimum (Stripe only)
- Customer gets excellent value
- Restaurant profitability protected

### Zero Shortfall Scenarios

- When displayed fee ≥ actual cost
- Service fee = Stripe processing only
- Restaurant pays no delivery subsidy
- Maximum profitability for restaurant

### Maximum Coverage Scenarios

- When shortfall > safe cap
- Service fee includes significant variable component
- Customer pays fair share of delivery costs
- Restaurant protection maintained

## Configuration Guidelines

### menuUplift Tuning
- **Higher values** (0.25-0.30): More restaurant protection, higher service fees
- **Lower values** (0.15-0.20): Less protection, more competitive service fees
- **Recommendation**: Match local competitor markup patterns

### targetLift Optimization
- **0.00**: Restaurant earns same as Uber (fair baseline)
- **0.03**: Restaurant earns 3% more than Uber (competitive advantage)
- **0.05**: Restaurant earns 5% more (strong incentive)
- **Recommendation**: Start at 0.00, increase based on market response

### coverageFraction Adjustment
- **1.00**: Use full safe coverage (maximum restaurant support)
- **0.80**: Use 80% of safe coverage (more service fees)
- **0.60**: Use 60% of safe coverage (customer pays more)
- **Recommendation**: Use 1.00 unless service fees become problematic

## Performance and Scaling

### Computational Efficiency
- Simple arithmetic operations only
- No complex iterations or recursive calls
- Constant-time execution regardless of order size
- Minimal memory allocation

### Real-time Suitability
- Sub-millisecond calculation time
- Suitable for real-time order updates
- Handles high-frequency recalculations
- No external API dependencies

## Monitoring and Analytics

### Key Metrics to Track

1. **Average service fee percentage** - Should remain reasonable
2. **Restaurant subsidy frequency** - How often restaurants cover shortfalls
3. **Customer total satisfaction** - Price competitiveness
4. **Platform fee coverage** - Ensure sustainable operations

### Success Indicators

- **Restaurant retention** - Restaurants continue using the platform
- **Customer satisfaction** - Acceptable total order values
- **Order frequency** - Customers place repeat orders
- **Profitability balance** - All stakeholders benefit appropriately

This algorithm provides a mathematically sound, transparent, and fair approach to fee distribution that ensures all parties benefit while maintaining clear, understandable pricing for customers interacting with the AI agent.