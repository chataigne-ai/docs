# Order Time Processor Documentation

## Overview

The OrderTimeProcessor validates order timing against restaurant opening hours and service availability. It ensures that orders are placed within valid operating windows for both delivery and collection services, providing essential timing validation for the AI agent to communicate accurate availability to customers.

**Location**: `apps/server/src/application/use-cases/order-draft/check-and-update/processors/time/order-time.processor.ts`

## Purpose

The OrderTimeProcessor is responsible for:

1. **Opening hours validation** - Ensures orders are placed during service hours
2. **Service type timing** - Validates delivery vs collection specific hours
3. **ASAP order validation** - Confirms immediate orders can be fulfilled
4. **Scheduled order validation** - Verifies future orders are within operating windows
5. **Multi-timezone support** - Handles timezone-aware time validation

## Processing Flow

### 1. Service Type Validation

```typescript
if (!state.draft.serviceType) {
  return;
}

const serviceType = state.draft.serviceType as 'delivery' | 'pickup';
```

Ensures service type is defined before proceeding with time validation.

### 2. ASAP Order Validation

For immediate orders (`requestedTime === 'ASAP'`):

#### Current Time Validation

```typescript
const isNowOpen = state.context.location.openingHours.isOpen(
  new Date(),
  state.context.timezone,
  serviceType,
);
```

Checks if the restaurant is currently open for the requested service type.

#### Future Time Validation

```typescript
const relevantTime = 
  state.draft.serviceType === 'delivery'
    ? state.context.location.deliverySettings.mode === 'uberDirect' ||
      state.context.location.deliverySettings.mode === 'chaskis'
      ? state.draft.readyTime
      : state.draft.deliveryTime
    : state.draft.readyTime;

const isFutureTimeOpen = state.context.location.openingHours.isOpen(
  new Date(relevantTime),
  state.context.timezone,
  serviceType,
);
```

Validates that the calculated delivery/ready time falls within operating hours.

**Time Selection Logic:**
- **Third-party delivery** (UberDirect/Chaskis): Uses `readyTime` (when order is ready for pickup by courier)
- **Direct delivery** (PostalCode/etc.): Uses `deliveryTime` (when order reaches customer)
- **Collection orders**: Uses `readyTime` (when order is ready for customer pickup)

#### Combined Validation

```typescript
if (!isNowOpen && !isFutureTimeOpen) {
  if (state.draft.serviceType === 'delivery') {
    state.errors.push({ code: 'OUTSIDE_OF_DELIVERY_HOURS' });
  } else {
    state.errors.push({ code: 'OUTSIDE_OF_STORE_HOURS' });
  }
  return;
}
```

Rejects orders only when both current time AND future fulfillment time are outside operating hours.

### 3. Scheduled Order Validation

For future orders with specific requested times:

```typescript
const isStartTimeOpen = state.context.location.openingHours.isOpen(
  state.draft.startTime,
  state.context.timezone,
  serviceType,
);

if (!isStartTimeOpen) {
  if (state.draft.serviceType === 'delivery') {
    state.errors.push({ code: 'OUTSIDE_OF_DELIVERY_HOURS' });
  } else {
    state.errors.push({ code: 'OUTSIDE_OF_STORE_HOURS' });
  }
  return;
}
```

Validates that the calculated start time (when kitchen begins preparation) falls within operating hours.

## Timing Strategy

### ASAP Orders - Flexible Validation

The processor uses a **permissive approach** for ASAP orders:

- **Scenario 1**: Restaurant closed now, but will be open when order is ready ✅ **ALLOWED**
- **Scenario 2**: Restaurant open now, but will be closed when order is ready ❌ **REJECTED**  
- **Scenario 3**: Restaurant closed now and will still be closed when order is ready ❌ **REJECTED**
- **Scenario 4**: Restaurant open now and will still be open when order is ready ✅ **ALLOWED**

This flexibility allows customers to place orders slightly before opening that will be ready during operating hours.

### Scheduled Orders - Strict Validation

For scheduled orders, validation is stricter:
- Kitchen start time must be within operating hours
- No flexibility for orders that begin preparation outside hours
- Ensures restaurant staff availability for order preparation

## Service Type Differentiation

### Delivery Service Hours

```typescript
serviceType = 'delivery'
```

Validates against:
- Restaurant's delivery-specific opening hours
- May differ from collection hours
- Considers delivery logistics and driver availability

### Collection Service Hours

```typescript  
serviceType = 'pickup' // Note: code uses 'pickup' for collection
```

Validates against:
- Restaurant's collection/pickup hours
- May have different hours than delivery
- Considers staff availability for order handoff

## Error Types for AI Agent

| Error Code | Service Type | AI Agent Response |
|------------|-------------|------------------|
| `OUTSIDE_OF_DELIVERY_HOURS` | Delivery | "We're not delivering right now. Our delivery hours are [hours]. Would you like to schedule for later or choose collection instead?" |
| `OUTSIDE_OF_STORE_HOURS` | Collection | "We're closed for pickup right now. Our collection hours are [hours]. Would you like to schedule for when we're open?" |

## AI Agent Integration

### Real-time Availability Communication

The AI agent uses this processor to:
- Inform customers of current service availability
- Suggest alternative timing when orders are rejected
- Provide specific opening hours for planning
- Handle timezone differences for international customers

### Proactive Time Management

The processor enables the AI agent to:
- Warn customers about approaching closing times
- Suggest earlier ordering for scheduled deliveries
- Automatically propose valid time alternatives
- Handle edge cases around opening/closing transitions

### Dynamic Scheduling Support

When orders are rejected, the AI agent can:
- Calculate next available service window
- Propose specific alternative times
- Handle recurring weekly schedules
- Manage holiday and special hour exceptions

## Timezone Handling

### Multi-timezone Support

```typescript
state.context.location.openingHours.isOpen(
  timeValue,
  state.context.timezone,
  serviceType,
);
```

The processor:
- Uses location-specific timezone for all calculations
- Handles daylight saving transitions
- Supports international locations
- Manages UTC conversion accurately

### Customer Communication

The AI agent receives timezone-aware data to:
- Display times in customer's local timezone
- Show restaurant timezone for clarity
- Handle booking across timezone boundaries
- Manage scheduling confirmation

## Opening Hours Integration

### Service-Specific Hours

The processor respects different hours for:
- **Delivery hours** - When delivery orders are accepted
- **Collection hours** - When pickup orders are accepted  
- **Kitchen hours** - When food preparation can begin
- **Overall hours** - General restaurant operation

### Complex Schedule Support

Handles various scheduling patterns:
- Different weekend vs weekday hours
- Lunch/dinner service windows
- Seasonal hour changes
- Holiday schedules
- Temporary closures

## Performance Considerations

### Efficient Validation

- Minimal time calculations
- Cached timezone conversions  
- Fast opening hours lookups
- Optimized date arithmetic

### Real-time Accuracy

- Current time validation for ASAP orders
- Precise future time projections
- Account for processing delays
- Handle rapid status changes

## Edge Case Handling

### Transition Periods

- Orders placed right at opening/closing time
- Daylight saving transitions
- Midnight crossover scenarios
- Multi-day scheduled orders

### Service Availability

- Kitchen closure before restaurant closure
- Delivery driver availability
- Third-party service windows
- Emergency closures

## Dependencies

- **Location opening hours** - Service-specific schedule configuration
- **Timezone data** - Accurate timezone handling
- **Order timing calculations** - Preparation and delivery time estimates
- **Service type configuration** - Delivery vs collection availability

This processor ensures that all orders are placed within valid service windows, enabling the AI agent to provide accurate availability information and guide customers toward successful order placement.