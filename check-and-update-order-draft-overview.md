# Check and Update Order Draft System Overview

## Introduction

The Check and Update Order Draft system is a core component of the Chataigne application that validates and processes order drafts before they can be converted into actual orders. This system ensures that all order information is complete, valid, and properly calculated before finalizing an order through the AI agent.

## Core Purpose

The system serves as a comprehensive validation and processing pipeline that:

1. **Validates order completeness** - Ensures all required information is present
2. **Calculates accurate pricing** - Computes delivery fees, service fees, discounts, and taxes
3. **Handles payment setup** - Prepares payment methods and processing
4. **Manages inventory constraints** - Validates item availability and restrictions
5. **Applies business rules** - Enforces location-specific policies and settings

## Architecture Overview

The system is built around a **processor chain architecture** where each processor handles a specific aspect of order validation and processing. This modular approach ensures:

- **Separation of concerns** - Each processor has a single responsibility
- **Maintainability** - Individual processors can be modified independently
- **Extensibility** - New processors can be easily added
- **Testability** - Each processor can be unit tested in isolation

## Main Components

### 1. CheckAndUpdateOrderDraftUsecase

The main orchestrator that:
- Loads the order draft context (location, customer, catalog, etc.)
- Initializes the validation state
- Executes the processor chain
- Returns the final validation result

### 2. ValidationState

A shared state object that flows through all processors containing:
- **draft**: The order draft being processed
- **context**: Related entities (location, customer, catalog)
- **errors**: Validation errors collected during processing
- **validationData**: Additional data computed during validation
- **patchLog**: Audit trail of all changes made

### 3. OrderDraftContext

Contains all related entities needed for validation:
- **initialOrderDraft**: The original order draft
- **location**: Restaurant/business location settings
- **catalog**: Available products and pricing
- **customer**: Customer information
- **conversation**: Chat context
- **timezone**: Location timezone for date calculations

## Processor Chain

The processors execute in a specific order to ensure dependencies are resolved:

1. **FormatDealsProcessor** - Formats and validates deal structures
2. **LocationStateProcessor** - Validates location availability and settings
3. **DiscountsProcessor** - Applies available discounts and promotions
4. **ItemsProcessor** - Validates items, quantities, and availability
5. **DeliveryProcessor** - Calculates delivery options and costs
6. **CustomerUpdateProcessor** - Handles customer information updates
7. **ComputeOrderFeesProcessor** - Calculates all fees using smart algorithms
8. **PaymentProcessor** - Sets up payment methods and validation
9. **OrderTimeProcessor** - Validates and sets order timing

### Error Handling

The system uses an **early exit strategy**:
- If any processor encounters validation errors, the chain stops immediately
- Errors are collected and returned to provide user feedback through the AI agent
- The order draft is saved with current progress for potential retry

### Validation Types

The system can exit with different validation states:

- **`validationError`** - Order has validation issues that must be resolved
- **`waitingForPaymentSetup`** - Payment method needs to be configured
- **`validOrderDraft`** - Order is valid and ready for placement

## Key Features

### 1. Context Loading
Before processing begins, the system loads all necessary context:
- Order draft from database
- Location settings and catalog
- Customer information
- Business organization details

### 2. State Management
The ValidationState object maintains:
- Current order draft state
- Audit trail of all changes
- Validation errors
- Computed data for UI display

### 3. Fee Calculation
Advanced fee calculation includes:
- Delivery fee estimation based on distance
- Service fee calculation with smart splitting
- Payment processing fees
- Tax calculations
- Discount applications

### 4. Business Rule Enforcement
Each processor enforces specific business rules:
- Opening hours and availability
- Minimum order amounts
- Product restrictions
- Delivery area limitations

## Integration with AI Agent

The Check and Update Order Draft system is primarily used by Chataigne's AI agent to validate customer orders during conversations:

### Real-time Order Validation
- The AI agent calls this system whenever customers modify their order
- Provides immediate feedback on order validity
- Enables dynamic pricing updates during conversation

### Error Communication
- Validation errors are formatted for natural language responses
- The AI agent can explain issues and guide customers to solutions
- Supports multi-language error messaging

### Order Finalization
- Ensures orders are complete before payment processing
- Validates all business rules before order submission
- Provides accurate pricing for customer confirmation

## Flow Diagram

```
AI Agent Order Update
         ↓
   Load Context
         ↓
   Initialize State
         ↓
   Apply Initial Updates
         ↓
   Check Ongoing Orders
         ↓
   Execute Processor Chain
   ┌─────────────────────┐
   │ FormatDeals         │
   │ LocationState       │
   │ Discounts          │
   │ Items              │
   │ Delivery           │
   │ CustomerUpdate     │
   │ ComputeOrderFees   │
   │ Payment            │
   │ OrderTime          │
   └─────────────────────┘
         ↓
   Save Updated Draft
         ↓
   Return Validation Result
         ↓
   AI Agent Response to Customer
```

## Error Recovery

The system is designed for graceful error handling:

1. **Validation errors** preserve the current state for user correction
2. **Context loading failures** return appropriate error messages
3. **Processing errors** include enough information for debugging
4. **Partial updates** are saved to prevent data loss

## Performance Considerations

- **Lazy loading** of expensive computations
- **Caching** of frequently accessed data
- **Parallel processing** where dependencies allow
- **Early exit** on validation failures

## Next Steps

For detailed information about specific processors, refer to:
- [Items Processor Documentation](./processors/items-processor.md)
- [Delivery Processor Documentation](./processors/delivery-processor.md)
- [Fee Calculation Documentation](./processors/fees-processor.md)
- [Smart Fee Splitting Algorithm](./smart-fee-splitting-algorithm.md)
- [Payment Processor Documentation](./processors/payment-processor.md)
- [Time Processor Documentation](./processors/time-processor.md)