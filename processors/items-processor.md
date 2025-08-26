# Items Processor Documentation

## Overview

The ItemsProcessor is responsible for validating all items and deals in an order draft. It ensures that every item in the order is valid, available, and properly configured according to the restaurant's catalog and business rules. This processor is crucial for the AI agent to provide accurate order validation feedback to customers.

**Location**: `apps/server/src/application/use-cases/order-draft/check-and-update/processors/items/items.processor.ts`

## Purpose

The ItemsProcessor performs comprehensive validation of:

1. **Item existence** - Verifies SKUs exist in the catalog
2. **Deal validation** - Ensures deal structures are valid and complete
3. **Option validation** - Validates selected options for each item
4. **Product restrictions** - Checks time, date, and service type restrictions
5. **Availability rules** - Ensures items are available for the requested service

## Processing Flow

### 1. Empty Order Check

```typescript
if ((draft.items?.length ?? 0) === 0 && (draft.deals?.length ?? 0) === 0) {
  state.errors.push({ code: 'NO_ITEMS_IN_ORDER_DRAFT' });
  return;
}
```

The processor first checks if the order has any items or deals. If neither exists, it immediately returns a validation error that the AI agent can communicate to the customer.

### 2. Individual Item Validation

For each item in the order, the processor performs the following validations:

#### SKU Existence Check

```typescript
const sku = validateItemSkuExists(item, catalog);
if (!sku) {
  state.errors.push({
    code: 'SKU_NOT_FOUND',
    skuInternalId: item.skuInternalId,
  });
  continue;
}
```

Verifies that the item's SKU exists in the restaurant's catalog.

#### Deal-Only SKU Check

```typescript
if (sku.dealOnly) {
  state.errors.push({
    code: 'SKU_ONLY_AVAILABLE_IN_DEALS',
    skuInternalId: item.skuInternalId,
  });
  continue;
}
```

Ensures that SKUs marked as "deal-only" are not ordered individually.

#### Product Restrictions Validation

```typescript
const restrictionErrors = validateProductRestrictions({
  item,
  product,
  catalog,
  location: state.context.location!,
  orderDraft: draft,
  timeData: state.context,
});
state.errors.push(...restrictionErrors);
```

Checks various product-level restrictions:
- **Time restrictions** - Product availability during specific hours
- **Date restrictions** - Seasonal or special date availability
- **Service type restrictions** - Delivery vs collection limitations
- **Location-specific rules** - Restaurant-specific availability rules

#### Option Validation

```typescript
const optionErrors = validateItemOptions({ item, sku, catalog });
state.errors.push(...optionErrors);
```

Validates that:
- Selected options exist for the SKU
- Required options are selected
- Option combinations are valid
- Option pricing is correct

### 3. Deal Validation

```typescript
if (draft.deals?.length) {
  const dealErrors = validateDeals({ deals: draft.deals, catalog });
  state.errors.push(...dealErrors);
}
```

For orders containing deals, validates:
- Deal structure and completeness
- Deal line assignments
- Pricing calculations
- Deal-specific restrictions

## Validation Helpers

### validateItemSkuExists

**Location**: `./lib/sku.ts`

Checks if a given SKU internal ID exists in the catalog and returns the SKU object if found.

### validateProductRestrictions  

**Location**: `./lib/product-restrictions.ts`

Comprehensive validation of product-level restrictions including:
- Time-based availability
- Service type restrictions
- Date-based restrictions
- Location-specific rules

### validateItemOptions

**Location**: `./lib/options.ts`

Validates option selections for an item:
- Ensures all required options are selected
- Validates option existence in catalog
- Checks option compatibility
- Verifies option pricing

### validateDeals

**Location**: `./lib/deals.ts`

Validates deal structures:
- Deal completeness
- Line item assignments
- Pricing calculations
- Deal-specific business rules

## Error Types for AI Agent

The ItemsProcessor can generate several types of validation errors that the AI agent uses to provide feedback:

| Error Code | Description | AI Agent Response |
|------------|-------------|------------------|
| `NO_ITEMS_IN_ORDER_DRAFT` | Order contains no items or deals | "Your order is empty. Please add some items to continue." |
| `SKU_NOT_FOUND` | SKU doesn't exist in catalog | "Sorry, that item is no longer available. Let me show you similar options." |
| `SKU_ONLY_AVAILABLE_IN_DEALS` | Item can only be ordered as part of a deal | "This item is only available as part of our special deals. Would you like to see our current offers?" |
| Product restriction errors | Various time/date/service restrictions | "This item is not available for [delivery/collection] at this time. It's available during [hours]." |
| Option validation errors | Missing or invalid option selections | "Please select your preferred [option type] for this item." |
| Deal validation errors | Invalid deal structure or content | "There's an issue with this deal configuration. Let me help you fix it." |

## Processing Characteristics

### Performance

- **Early exit** - Stops processing at first critical error
- **Batch validation** - Collects multiple validation errors when possible
- **Catalog lookup optimization** - Efficient SKU and product lookups

### Dependencies

- **Catalog** - Requires fully loaded product catalog
- **Location settings** - Needs location-specific rules and restrictions
- **Time context** - Requires current time for time-based validations

### Side Effects

- **No mutations** - Only adds validation errors, doesn't modify the order draft
- **No database operations** - Pure validation logic
- **Context preservation** - Maintains all validation context for downstream processors

## Integration with AI Agent

### Real-time Validation

The AI agent uses this processor to:
- Validate items as customers add them to their order
- Provide immediate feedback on availability
- Suggest alternatives when items are unavailable
- Guide customers through option selections

### Conversational Error Handling

When validation fails, the AI agent:
- Translates error codes into natural language
- Provides specific guidance on how to fix issues
- Offers alternative products or configurations
- Maintains conversation flow despite validation errors

### Order Building Support

The processor helps the AI agent:
- Ensure order completeness before checkout
- Validate complex deal configurations
- Handle product restrictions transparently
- Support multi-language error messaging

## Integration Points

### Input Requirements

- Valid order draft with items or deals
- Complete catalog with products, SKUs, and options
- Location context with business rules
- Time context for restriction validation

### Output Guarantees

- All validation errors are collected in `state.errors`
- Order draft remains unchanged
- Validation context preserved for subsequent processors

### Downstream Impact

- **Delivery Processor** - Relies on validated items for weight/size calculations
- **Fee Processor** - Uses validated items for pricing calculations
- **Payment Processor** - Depends on valid order total

This processor is critical for ensuring order integrity and enabling the AI agent to provide accurate, helpful guidance to customers during the ordering process.