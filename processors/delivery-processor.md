# Delivery Processor Documentation

## Overview

The DeliveryProcessor handles all delivery and collection-related validation and setup for order drafts. It manages address resolution, distance calculation, delivery quote generation, and timing computation for both delivery and collection orders processed by Chataigne's AI agent.

**Location**: `apps/server/src/application/use-cases/order-draft/check-and-update/processors/delivery/delivery.processor.ts`

## Purpose

The DeliveryProcessor is responsible for:

1. **Service type validation** - Ensures valid service type (delivery/collection)
2. **Address resolution** - Converts customer addresses to deliverable locations
3. **Distance and duration calculation** - Computes delivery metrics
4. **Delivery condition checks** - Validates delivery area and constraints
5. **Quote generation** - Gets delivery pricing from various providers
6. **Timing computation** - Calculates order preparation and delivery times
7. **Fee calculation** - Determines displayed delivery fees for customers

## Processing Flow

### 1. Basic Validation

```typescript
if (!state.draft.serviceType) {
  state.errors.push({ code: 'SERVICE_TYPE_REQUIRED' });
  return;
}

if (!state.draft.requestedTime) {
  state.errors.push({ code: 'REQUESTED_TIME_REQUIRED' });
  return;
}
```

Validates that essential order properties are present before proceeding.

### 2. Collection Order Processing

For collection orders (`serviceType === 'collection'`):

```typescript
const orderTimings = await computeCollectionOrderTimings(state);

patchDraft(state, {
  set: {
    startTime: orderTimings.startTime,
    readyTime: orderTimings.readyTime,
    deliveryTime: orderTimings.deliveryTime,
  }
}, {
  processor: 'DeliveryProcessor',
  summary: 'Computed order timings for collection order',
});
```

Collection orders skip address resolution and delivery-specific validations, only computing preparation timings.

### 3. Delivery Order Processing

For delivery orders, the processor executes a comprehensive validation pipeline:

#### Address Resolution

```typescript
const addressCheckResult = await resolveAddress(state, this.geocoding);

if (!addressCheckResult.success) {
  state.errors.push(addressCheckResult.error);
  return;
}
```

Uses the geocoding service to:
- Validate the customer's delivery address
- Convert to precise coordinates
- Calculate distance from restaurant
- Compute estimated travel time

#### Delivery Conditions Validation

```typescript
const checkDeliveryConditionsResult = await checkDeliveryConditions(
  state,
  distance,
  duration,
);

if (checkDeliveryConditionsResult.errors.length > 0) {
  state.errors.push(...checkDeliveryConditionsResult.errors);
  return;
}
```

Validates delivery feasibility:
- Distance within delivery radius
- Address within delivery zones
- Delivery time within service hours
- Special location restrictions

#### Order Timing Computation

The processor computes optimal order timings based on the request type:

**ASAP Orders:**
```typescript
const startTime = DateTime.now().toJSDate();
const expectedReadyTime = DateTime.now()
  .plus({ minutes: Math.max(location.averagePreparationTime.minute, MIN_READY_MINUTES) })
  .toJSDate();
const expectedDeliveryTime = DateTime.fromJSDate(expectedReadyTime)
  .plus({ minutes: deliveryDuration })
  .toJSDate();
```

**Scheduled Orders:**
```typescript
const expectedDeliveryTime = DateTime.fromJSDate(state.draft.requestedTime).toJSDate();
const expectedReadyTime = DateTime.fromJSDate(expectedDeliveryTime)
  .minus({ minutes: deliveryDuration })
  .toJSDate();
const startTime = DateTime.fromJSDate(expectedReadyTime)
  .minus({ minutes: location.averagePreparationTime.minute })
  .toJSDate();
```

#### Quote Generation

```typescript
const quote = await this.getQuote(state, distance);
const displayedFee = await this.getDisplayedFee(state, distance);
```

Gets delivery quotes from configured providers:
- **UberDirect** - Uber's delivery platform integration
- **Chaskis** - Third-party delivery service
- **PostalCode** - Zone-based delivery pricing

### 4. Fee Validation

```typescript
if (state.context.location.deliverySettings.maxDeliveryFee &&
    (quote?.fee?.amount ?? 0) > state.context.location.deliverySettings.maxDeliveryFee?.amount) {
  state.errors.push({
    code: 'DELIVERY_FEE_TOO_HIGH',
    feeAmount: state.context.location.deliverySettings.maxDeliveryFee?.amount,
  });
  return;
}
```

Validates that delivery costs don't exceed configured limits.

## Timing Resolution

### ASAP Orders

For immediate orders, the processor:
1. Sets start time to current time
2. Adds preparation time to get ready time
3. Adds delivery duration to get delivery time
4. Ensures minimum preparation time is respected

### Scheduled Orders

For future orders, the processor:
1. Works backwards from requested delivery time
2. Subtracts delivery duration to get ready time
3. Subtracts preparation time to get start time
4. Handles edge cases where calculated times are in the past

### Past Time Edge Case Handling

When scheduled order calculations result in past start times:

```typescript
if (DateTime.fromJSDate(startTime, { zone: state.context.timezone }) < DateTime.now()) {
  // Treat as ASAP order and recalculate
  const updatedDeliveryDuration = await this.geocoding.computeTravelTime({
    origin: state.validationData.deliveryAddress,
    destination: state.context.location.address,
    country: state.context.location.country,
    departureTime: expectedReadyTime,
  });
  
  // Revalidate against opening hours
  if (!state.context.location.openingHours.isOpen(startTime, state.context.timezone, 'delivery')) {
    return { success: false, error: { code: 'OUTSIDE_OF_DELIVERY_HOURS' } };
  }
}
```

## Delivery Providers

### UberDirect Integration

Handles Uber's on-demand delivery platform:
- Real-time quote generation
- Dynamic pricing based on demand
- Integrated tracking and fulfillment

### Chaskis Integration  

Third-party delivery service integration:
- Distance-based pricing
- Custom fee calculations
- Regional availability

### PostalCode Provider

Zone-based delivery system:
- Fixed pricing by postal code areas
- Simplified geographic coverage
- Predictable delivery costs

## State Updates

The processor updates the order draft with:

```typescript
patchDraft(state, {
  set: {
    quoteId: quote.quoteId,           // Provider quote reference
    realDeliveryFees: quote.fee.amount,  // Actual delivery cost
    displayedDeliveryFees: displayedFee, // Customer-facing fee
    startTime: orderTimings.startTime,   // Order start time
    readyTime: orderTimings.readyTime,   // Order ready time  
    deliveryTime: orderTimings.deliveryTime, // Expected delivery time
  }
});
```

## Validation Data for AI Agent

The processor populates validation data that the AI agent uses:

```typescript
state.validationData.deliveryAddress = address;
state.validationData.deliveryTime = DateTime.fromJSDate(orderTimings.deliveryTime)
  .setZone(state.context.timezone)
  .toISO({ suppressMilliseconds: true });
```

## Error Types for AI Agent

| Error Code | Description | AI Agent Response |
|------------|-------------|------------------|
| `SERVICE_TYPE_REQUIRED` | Missing service type | "Would you like this delivered or for collection?" |
| `REQUESTED_TIME_REQUIRED` | Missing timing preference | "When would you like this order? ASAP or at a specific time?" |
| `ADDRESS_INVALID` | Address cannot be resolved | "I couldn't find that address. Could you provide a more specific address?" |
| `OUTSIDE_DELIVERY_AREA` | Address outside delivery zone | "We don't deliver to that area. Would you prefer collection instead?" |
| `OUTSIDE_OF_DELIVERY_HOURS` | Order outside delivery hours | "We're closed for delivery at that time. We deliver from [hours]." |
| `DELIVERY_FEE_TOO_HIGH` | Quote exceeds maximum fee | "The delivery fee is quite high for that distance. Collection is free!" |

## Integration with AI Agent

### Customer Address Processing

When customers provide delivery addresses, the AI agent uses this processor to:
- Validate address accuracy using geocoding
- Confirm deliverability to the location  
- Calculate accurate delivery fees upfront
- Provide timing estimates for customer expectations

### Dynamic Pricing Communication

The processor enables the AI agent to:
- Quote accurate delivery fees during conversation
- Explain delivery time estimates
- Handle address changes and re-quote automatically
- Manage customer expectations about delivery costs

### Conversational Flow Management

When validation fails, the processor provides specific error codes that the AI agent uses to:
- Generate contextual error messages in the customer's language
- Guide customers toward valid alternatives (collection vs delivery)
- Maintain conversation flow despite validation issues
- Collect missing information systematically

### Real-time Order Updates

The AI agent uses this processor to:
- Update delivery estimates as customers modify orders
- Recalculate fees when addresses change
- Validate delivery windows for scheduled orders
- Handle timezone conversions for international locations

## Performance Considerations

### Geocoding Optimization

- Caches geocoding results to avoid repeated API calls
- Uses efficient address normalization
- Handles geocoding service timeouts gracefully

### Quote Generation

- Supports multiple provider fallbacks
- Implements timeout handling for external APIs
- Caches quotes for identical requests

### Timing Calculations

- Optimized timezone handling with Luxon
- Efficient date arithmetic
- Minimal memory allocation for frequent calculations

## Dependencies

- **IGeocodingService** - Address resolution and distance calculation
- **Location settings** - Delivery configuration and business rules  
- **Opening hours** - Service availability validation
- **Provider integrations** - UberDirect, Chaskis, PostalCode APIs

## Configuration Requirements

### Delivery Settings

Each location must have configured:
- Delivery mode (UberDirect, Chaskis, PostalCode, etc.)
- Maximum delivery distance
- Delivery hours
- Maximum delivery fee (optional)

### Provider Credentials

- UberDirect API keys and configuration
- Chaskis integration settings
- PostalCode zone definitions

This processor is essential for ensuring accurate delivery logistics and pricing, enabling the AI agent to provide seamless delivery experience to customers.