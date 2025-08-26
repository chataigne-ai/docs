# Fee Calculation Processor Documentation

## Overview

The ComputeOrderFeesProcessor is responsible for calculating all fees associated with an order draft, including payment fees, service fees, and delivery fees. It integrates with the smart fee splitting algorithm to ensure optimal pricing for both customers and restaurants while the AI agent manages customer interactions.

**Location**: `apps/server/src/application/use-cases/order-draft/check-and-update/processors/fees/compute-order-fees.processor.ts`

## Purpose

The ComputeOrderFeesProcessor handles:

1. **Payment fee calculation** - Stripe processing fees for card payments
2. **Service fee computation** - Platform and processing fees shown to customers
3. **Delivery fee determination** - Final delivery costs using smart algorithms
4. **Fee splitting logic** - Revenue optimization between customer, restaurant, and platform
5. **Payment method differentiation** - Different fee structures for various payment types

## Processing Flow

### 1. Payment Fee Calculation

```typescript
await computePaymentFee(state);
```

First computes basic payment processing fees based on:
- Payment method type (Stripe vs HandledByStore)
- Order total amount
- Currency and processing rates

### 2. Service and Delivery Fee Computation

```typescript
await this.computeServiceFeeAndDeliveryFee(state);
```

Main logic that determines final fees based on service type and payment method.

## Fee Calculation by Service Type

### Collection Orders

For collection orders, fee calculation is simplified:

**HandledByStore Payment:**
```typescript
if (state.draft.paymentMethod === 'HandledByStore') {
  serviceFee = 0;
  deliveryFee = 0;
}
```
No fees when restaurant handles payment directly.

**Stripe Payment Methods:**
```typescript
serviceFee = StripeFeeCalculatorService.calculateProcessingFee(
  new Price(orderDraft.computeTotal(catalog).amount, location.currency)
);
deliveryFee = 0;
```
Only Stripe processing fees apply, no delivery charges.

### Delivery Orders

Delivery orders use different logic based on payment method:

#### HandledByStore Delivery Payment

```typescript
switch (location.deliverySettings.mode) {
  case 'free':
    displayedDeliveryFee = 0;
    displayedServiceFee = 0;
    break;
  case 'chaskis':
  case 'uberDirect':
    smartFeesResult = computeFeesForThirdPartyDelivery(state);
    displayedDeliveryFee = smartFeesResult.customerLines.delivery;
    displayedServiceFee = smartFeesResult.customerLines.service;
    break;
  case 'postalCode':
    displayedServiceFee = 0;
    displayedDeliveryFee = state.draft.displayedDeliveryFees;
    break;
  case 'none':
    displayedDeliveryFee = 0;
    displayedServiceFee = 0;
    break;
}
```

#### Stripe Delivery Payment

```typescript
switch (location.deliverySettings.mode) {
  case 'free':
    displayedDeliveryFee = 0;
    displayedServiceFee = 0;
    break;
  case 'chaskis':
  case 'uberDirect':
    smartFeesResult = computeFeesForThirdPartyDelivery(state);
    displayedDeliveryFee = smartFeesResult.customerLines.delivery;
    displayedServiceFee = smartFeesResult.customerLines.service;
    break;
  case 'postalCode':
    displayedServiceFee = paymentFee;
    displayedDeliveryFee = state.draft.displayedDeliveryFees;
    break;
  case 'none':
    displayedDeliveryFee = 0;
    displayedServiceFee = paymentFee;
    break;
}
```

## Smart Fee Splitting Integration

For third-party delivery services (Chaskis, UberDirect), the processor calls the smart fee splitting algorithm:

```typescript
smartFeesResult = computeFeesForThirdPartyDelivery(state);
displayedDeliveryFee = smartFeesResult.customerLines.delivery;
displayedServiceFee = smartFeesResult.customerLines.service;
```

This returns detailed fee breakdown:
- **customerLines** - What customers see (subtotal, delivery, service, total)
- **transparency** - Internal cost breakdown
- **restaurant** - Restaurant economics vs competitors

## State Updates

The processor updates the order draft with calculated fees:

```typescript
patchDraft(state, {
  set: {
    serviceFee: serviceFee,
    deliveryFee: deliveryFee,
    smartFeeResult: smartFeesResult,
  }
}, {
  processor: 'ComputeOrderFees',
  summary: 'Computed order fees',
});
```

## Validation Data for AI Agent

The processor provides structured fee data for the AI agent:

```typescript
state.validationData.serviceFee = {
  amount: serviceFee,
  currency: location.currency,
};

state.validationData.deliveryFee = {
  amount: deliveryFee,
  currency: location.currency,
};

if (smartFeesResult) {
  state.validationData.splitAlgorithmResult = smartFeesResult;
}
```

## AI Agent Integration

### Price Communication

The AI agent uses fee calculation results to:
- Provide accurate total pricing to customers
- Explain fee breakdowns when asked
- Compare delivery vs collection costs
- Handle price changes when addresses or items change

### Transparency Features

When customers ask about fees, the AI agent can access:
- Service fee explanations
- Delivery cost breakdowns
- Payment processing fee details
- Smart algorithm transparency data

### Dynamic Pricing Updates

As customers modify orders, the processor enables the AI agent to:
- Recalculate fees in real-time
- Update pricing displays instantly
- Handle address changes affecting delivery fees
- Manage payment method switches

## Delivery Mode Handling

### Free Delivery Mode
- Zero delivery and service fees
- Used for promotional periods or local testing

### PostalCode Mode
- Fixed pricing by geographic zones
- Service fee only includes Stripe processing
- Predictable costs for customers

### Third-Party Delivery (Chaskis/UberDirect)
- Dynamic pricing using smart algorithms
- Complex fee splitting between stakeholders
- Optimized for restaurant profitability

### No Delivery Mode
- Collection only locations
- Service fees cover payment processing only

## Fee Validation

The processor ensures:
- **Non-negative fees** - All calculated fees are >= 0
- **Currency consistency** - All fees use location currency
- **Rounding accuracy** - Proper monetary rounding
- **Payment method compatibility** - Fees match selected payment type

## Error Handling

The processor handles edge cases:
- Missing delivery settings gracefully
- Invalid payment methods with appropriate errors
- Calculation failures with fallback logic
- Currency conversion issues

## Performance Considerations

### Calculation Efficiency
- Cached Stripe fee calculations
- Optimized smart algorithm calls
- Minimal database queries
- Efficient currency operations

### Real-time Updates
- Fast recalculation for order changes
- Immediate fee updates for address changes
- Responsive payment method switching
- Quick total updates for item modifications

## Dependencies

- **StripeFeeCalculatorService** - Payment processing fee calculation
- **Smart Fee Splitting Algorithm** - Advanced delivery fee optimization
- **Location delivery settings** - Configuration for fee modes
- **Payment method configuration** - Available payment types

## Integration Points

### Input Requirements
- Valid order draft with items and totals
- Configured payment methods
- Delivery settings for the location
- Customer payment preferences

### Output Guarantees
- Accurate fee calculations in correct currency
- Properly rounded monetary values
- Complete fee breakdown for transparency
- Smart algorithm results when applicable

### Downstream Impact
- **Payment Processor** - Uses calculated fees for payment setup
- **Order finalization** - Final pricing for order submission
- **AI Agent responses** - Accurate pricing communication

This processor ensures accurate, transparent, and optimized fee calculation that benefits customers, restaurants, and the platform while enabling the AI agent to provide clear pricing information.