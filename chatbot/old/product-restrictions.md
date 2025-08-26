---
title: Product Restrictions
description: Understanding and implementing product availability restrictions in your chatbot
---

# Product Restrictions

Product restrictions allow restaurant owners to control product availability based on various criteria such as days of the week, time ranges, date ranges, and service type (delivery vs pickup). This feature ensures that customers can only order products when they are actually available.

## Overview

The product restrictions feature works by adding optional restriction rules to products in the catalog. When a customer attempts to order a product, the chatbot validates these restrictions against the current context (time, date, service type) to ensure the product is available.

## Data Structure

### Restriction Object

The restriction object contains the following fields:

| Field                  | Type                      | Required | Description                                          |
| ---------------------- | ------------------------- | -------- | ---------------------------------------------------- |
| `dow`                  | [DaysOfWeek](#daysofweek) | Yes      | Days of the week when the product is available       |
| `startTime`            | string                    | No       | Start time for availability (HH:MM format, 24-hour)  |
| `endTime`              | string                    | No       | End time for availability (HH:MM format, 24-hour)    |
| `startDate`            | string                    | No       | Start date for availability (ISO 8601 date string)   |
| `endDate`              | string                    | No       | End date for availability (ISO 8601 date string)     |
| `availableForDelivery` | boolean                   | Yes      | Whether the product is available for delivery orders |
| `availableForPickup`   | boolean                   | Yes      | Whether the product is available for pickup orders   |

### DaysOfWeek Object

| Field       | Type    | Required | Description            |
| ----------- | ------- | -------- | ---------------------- |
| `monday`    | boolean | Yes      | Available on Monday    |
| `tuesday`   | boolean | Yes      | Available on Tuesday   |
| `wednesday` | boolean | Yes      | Available on Wednesday |
| `thursday`  | boolean | Yes      | Available on Thursday  |
| `friday`    | boolean | Yes      | Available on Friday    |
| `saturday`  | boolean | Yes      | Available on Saturday  |
| `sunday`    | boolean | Yes      | Available on Sunday    |

## Default Behavior

When no restriction is specified for a product:

- The product is available all days of the week
- No time restrictions apply
- No date restrictions apply
- Available for both delivery and pickup

## Implementation Examples

### Example 1: Weekend-Only Product

```json
{
  "internalId": "weekend_special_001",
  "name": "Weekend Special Pizza",
  "categoryName": "Specials",
  "description": "Our special weekend pizza",
  "isAvailable": true,
  "restriction": {
    "dow": {
      "monday": false,
      "tuesday": false,
      "wednesday": false,
      "thursday": false,
      "friday": false,
      "saturday": true,
      "sunday": true
    },
    "availableForDelivery": true,
    "availableForPickup": true
  }
}
```

### Example 2: Lunch-Time Only Product

```json
{
  "internalId": "lunch_menu_001",
  "name": "Business Lunch",
  "categoryName": "Lunch",
  "description": "Quick lunch option",
  "isAvailable": true,
  "restriction": {
    "dow": {
      "monday": true,
      "tuesday": true,
      "wednesday": true,
      "thursday": true,
      "friday": true,
      "saturday": false,
      "sunday": false
    },
    "startTime": "11:30",
    "endTime": "14:30",
    "availableForDelivery": true,
    "availableForPickup": true
  }
}
```

### Example 3: Pickup-Only Product

```json
{
  "internalId": "fresh_bread_001",
  "name": "Fresh Daily Bread",
  "categoryName": "Bakery",
  "description": "Freshly baked bread, pickup only",
  "isAvailable": true,
  "restriction": {
    "dow": {
      "monday": true,
      "tuesday": true,
      "wednesday": true,
      "thursday": true,
      "friday": true,
      "saturday": true,
      "sunday": true
    },
    "availableForDelivery": false,
    "availableForPickup": true
  }
}
```

### Example 4: Seasonal Product

```json
{
  "internalId": "summer_special_001",
  "name": "Summer Ice Cream Special",
  "categoryName": "Desserts",
  "description": "Available during summer months",
  "isAvailable": true,
  "restriction": {
    "dow": {
      "monday": true,
      "tuesday": true,
      "wednesday": true,
      "thursday": true,
      "friday": true,
      "saturday": true,
      "sunday": true
    },
    "startDate": "2024-06-01T00:00:00.000Z",
    "endDate": "2024-08-31T23:59:59.999Z",
    "availableForDelivery": true,
    "availableForPickup": true
  }
}
```

## Chatbot Integration Guidelines

### 1. Validation Logic

When processing an order, the chatbot should validate each product against its restrictions:

```typescript
function isProductAvailable(
  product: ChatbotProductDTO,
  orderContext: OrderContext
): boolean {
  // If no restrictions, product is always available
  if (!product.restriction) {
    return product.isAvailable;
  }

  const restriction = product.restriction;
  const now = new Date(); // Should be in restaurant's timezone

  // Check day of week
  const dayOfWeek = now.toLocaleDateString("en-US", { weekday: "lowercase" });
  if (!restriction.dow[dayOfWeek]) {
    return false;
  }

  // Check time restrictions
  if (restriction.startTime || restriction.endTime) {
    const currentTime = now.toTimeString().slice(0, 5); // HH:MM format

    if (restriction.startTime && currentTime < restriction.startTime) {
      return false;
    }

    if (restriction.endTime && currentTime > restriction.endTime) {
      return false;
    }
  }

  // Check date restrictions
  if (restriction.startDate && now < new Date(restriction.startDate)) {
    return false;
  }

  if (restriction.endDate && now > new Date(restriction.endDate)) {
    return false;
  }

  // Check service type
  if (
    orderContext.serviceType === "delivery" &&
    !restriction.availableForDelivery
  ) {
    return false;
  }

  if (
    orderContext.serviceType === "pickup" &&
    !restriction.availableForPickup
  ) {
    return false;
  }

  return product.isAvailable;
}
```

### 2. Error Handling

When a product is not available due to restrictions, provide clear error messages:

- **Day restrictions**: "Sorry, [Product Name] is only available on [available days]."
- **Time restrictions**: "Sorry, [Product Name] is only available between [start time] and [end time]."
- **Date restrictions**: "Sorry, [Product Name] is only available from [start date] to [end date]."
- **Service type restrictions**: "Sorry, [Product Name] is only available for [pickup/delivery]."

### 3. Proactive Filtering

Filter out unavailable products from menus and suggestions:

```typescript
function getAvailableProducts(
  catalog: ChatbotCatalogDTO,
  orderContext: OrderContext
): ChatbotProductDTO[] {
  return catalog.products.filter((product) =>
    isProductAvailable(product, orderContext)
  );
}
```

### 4. Time Zone Considerations

**Critical**: Always use the restaurant's local timezone for time-based validations:

```typescript
// Get restaurant's timezone from location data
const restaurantTimezone = location.timezone; // e.g., "Europe/Zurich"

// Convert current time to restaurant's timezone
const restaurantTime = new Date().toLocaleString("en-CA", {
  timeZone: restaurantTimezone,
  hour12: false,
});
```

## Best Practices

### 1. Clear Communication

- Always inform customers about availability restrictions
- Suggest alternative products when restrictions apply
- Provide clear timeframes for when products will be available again

### 2. Graceful Degradation

- If restriction data is missing or invalid, default to allowing the product
- Log validation errors for debugging without blocking orders

### 3. Caching Considerations

- Cache availability results for performance
- Invalidate cache when time-sensitive restrictions change
- Consider pre-calculating availability for common time periods

### 4. User Experience

- Show availability information proactively in product listings
- Use visual indicators (badges, colors) to highlight restrictions
- Group products by availability status when helpful

## Common Use Cases

1. **Breakfast Items**: Available only until 11:00 AM
2. **Dinner Specials**: Available only after 5:00 PM
3. **Weekend Brunch**: Available only on Saturday and Sunday, 10:00 AM - 3:00 PM
4. **Happy Hour**: Available only on weekdays, 3:00 PM - 6:00 PM
5. **Seasonal Items**: Available only during specific months
6. **Fresh Items**: Pickup only to ensure quality
7. **Delivery-Only**: Items that require special packaging

## Technical Notes

- All times are in 24-hour format (HH:MM)
- Dates use ISO 8601 format for consistency
- Time comparisons should account for the restaurant's timezone
- Boolean flags default to `true` when not specified for maximum availability
- The `isAvailable` flag acts as a master switch that can override all restrictions

This documentation provides a complete guide for implementing and working with product restrictions in the Châtaigne chatbot system.
