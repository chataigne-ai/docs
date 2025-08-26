# Payment Processor Documentation

## Overview

The PaymentProcessor manages payment method validation and setup for order drafts. It handles different payment types, creates Stripe customers, generates payment URLs, and determines the appropriate payment flow for order completion. This processor is essential for the AI agent to guide customers through secure payment processes.

**Location**: `apps/server/src/application/use-cases/order-draft/check-and-update/processors/payment/payment.processor.ts`

## Purpose

The PaymentProcessor is responsible for:

1. **Payment method validation** - Ensures selected payment methods are available
2. **Stripe customer creation** - Sets up customers for online payments
3. **Payment URL generation** - Creates secure checkout and setup links
4. **Payment flow determination** - Decides between instant, checkout, or in-store payment
5. **Payment method setup** - Handles saved card and one-time payment configurations

## Processing Flow

### 1. Payment Method Auto-Selection

```typescript
if (paymentMethods.length === 1) {
  patchDraft(state, 
    { set: { paymentMethod: paymentMethods[0] } },
    {
      processor: 'Payment',
      summary: 'Set the payment method to the only available payment method',
    }
  );
}
```

When only one payment method is available, it's automatically selected for user convenience.

### 2. Payment Method Validation

```typescript
if (!state.draft.paymentMethod) {
  state.errors.push({ code: 'NO_PAYMENT_METHOD' });
  state.validationData.availablePaymentMethods = paymentMethods;
  return;
}

if (!paymentMethods.includes(state.draft.paymentMethod)) {
  state.errors.push({ code: 'INVALID_PAYMENT_METHOD' });
  state.validationData.availablePaymentMethods = paymentMethods;
  return;
}
```

Validates that a payment method is selected and that it's available for the location.

### 3. Stripe Customer Setup

```typescript
if (state.draft.paymentMethod !== 'HandledByStore' && !state.context.customer.stripeId) {
  const customer = await this.createStripeCustomerUsecase.execute(
    state.context.customer.id,
  );
  state.context.customer = customer;
}
```

Creates Stripe customer records for online payment methods when needed.

## Payment Method Handling

### HandledByStore

For in-store payment handling:

```typescript
case 'HandledByStore':
  state.validationType = 'readyForPayment';
  state.validationData.paymentType = 'RegisterWithoutPayment';
  break;
```

- No online payment processing
- Order ready for kitchen preparation
- Payment handled at pickup/delivery

### PersistentCard (Saved Cards)

For customers with saved payment methods:

```typescript
case 'PersistentCard':
  const hasPaymentMethod = state.context.customer.paymentMethods.some(
    (paymentMethod) => paymentMethod.isActive === true,
  );

  if (!hasPaymentMethod) {
    state.validationType = 'noPaymentMethod';
    const shortUrl = await createSetupSessionUrl(state, this.createShortLinkUsecase);
    state.validationData.paymentMethodSetupUrl = shortUrl;
    return;
  }

  const paymentMethod = state.context.customer.paymentMethods.filter(
    (paymentMethod) => paymentMethod.isActive === true,
  )[0];

  state.validationData.paymentMethod = DtoPaymentMethodAdapter.paymentMethodToDTO(paymentMethod);
  state.validationType = 'readyForPayment';
  state.validationData.paymentType = 'InstantPayment';
  break;
```

**Flow:**
1. Check if customer has active saved payment methods
2. If no saved method: Create setup URL for card registration
3. If saved method exists: Prepare for instant payment
4. Set appropriate validation type

### TemporaryCard (One-time Payments)

For single-use card payments:

```typescript
case 'TemporaryCard':
  state.validationType = 'readyForPayment';
  
  const shortCardUrl = await createCheckoutSessionUrl(
    state,
    'TemporaryCard',
    this.createShortLinkUsecase,
  );
  
  state.validationData.paymentMethodSetupUrl = shortCardUrl;
  state.validationData.paymentType = 'CheckoutPayment';
  break;
```

Creates Stripe checkout session for one-time card entry.

### RevolutPay

For Revolut payment integration:

```typescript
case 'RevolutPay':
  state.validationType = 'readyForPayment';
  
  const shortRevolutUrl = await createCheckoutSessionUrl(
    state,
    'RevolutPay',
    this.createShortLinkUsecase,
  );
  
  state.validationData.paymentMethodSetupUrl = shortRevolutUrl;
  state.validationData.paymentType = 'CheckoutPayment';
  break;
```

Integrates with Revolut's payment system through Stripe.

### Twint

For Swiss Twint mobile payments:

```typescript
case 'Twint':
  state.validationType = 'readyForPayment';
  
  const shortTwintUrl = await createCheckoutSessionUrl(
    state,
    'Twint',
    this.createShortLinkUsecase,
  );
  
  state.validationData.paymentMethodSetupUrl = shortTwintUrl;
  state.validationData.paymentType = 'CheckoutPayment';
  break;
```

Supports Swiss mobile payment method through Stripe integration.

## Payment Flow Types

### RegisterWithoutPayment

For in-store payment:
- Order is registered in system
- Kitchen receives order immediately
- Payment handled at fulfillment time
- No online payment processing

### InstantPayment

For saved payment methods:
- Immediate charge to saved card
- Fastest checkout experience
- Order confirmed instantly
- Kitchen receives order immediately

### CheckoutPayment

For new card entry and alternative payments:
- Customer redirected to secure payment page
- Card details entered on Stripe-hosted page
- Payment processed before order confirmation
- Order sent to kitchen after payment success

### Payment Setup (noPaymentMethod)

For customers without saved cards:
- Setup session created for card registration
- Customer saves card for future use
- No immediate payment taken
- Returns to order flow after card setup

## URL Generation

### Setup Session URLs

```typescript
const shortUrl = await createSetupSessionUrl(state, this.createShortLinkUsecase);
```

Creates secure URLs for:
- Adding new payment methods
- Card verification and storage
- Future payment setup

### Checkout Session URLs

```typescript
const shortCardUrl = await createCheckoutSessionUrl(
  state,
  'TemporaryCard',
  this.createShortLinkUsecase,
);
```

Creates secure payment URLs for:
- One-time payments
- Alternative payment methods
- Immediate order payment

## Validation Data for AI Agent

The processor provides payment information for AI agent communication:

```typescript
state.validationData.availablePaymentMethods = paymentMethods;
state.validationData.paymentMethodSetupUrl = shortUrl;
state.validationData.paymentType = 'InstantPayment';
state.validationData.paymentMethod = paymentMethodDTO;
```

## AI Agent Integration

### Payment Method Selection

The AI agent uses this processor to:
- Present available payment options to customers
- Auto-select single payment methods
- Guide customers through payment method selection
- Handle payment method availability issues

### Payment Flow Communication

When payment setup is required, the AI agent:
- Explains the payment process clearly
- Provides secure payment links to customers
- Manages expectations about payment timing
- Handles payment method setup failures

### Error Handling for Conversations

Payment errors are communicated through:

| Error Code | AI Agent Response |
|------------|------------------|
| `NO_PAYMENT_METHOD` | "Please choose how you'd like to pay: [list available methods]" |
| `INVALID_PAYMENT_METHOD` | "That payment method isn't available. Here are your options: [methods]" |

### Payment Status Updates

The AI agent uses validation types to:
- **`readyForPayment`** - Proceed with order confirmation
- **`noPaymentMethod`** - Request payment method setup
- **`validationError`** - Handle payment configuration issues

## Security Considerations

### Stripe Integration

- Customer data handled securely through Stripe APIs
- No sensitive card data stored in application
- PCI compliance through Stripe's infrastructure
- Secure tokenization of payment methods

### URL Security

- Short links prevent URL manipulation
- Time-limited checkout sessions
- Secure redirect handling
- Session validation

## Error Recovery

### Payment Setup Failures

- Clear error messaging through AI agent
- Alternative payment method suggestions
- Retry mechanisms for temporary issues
- Fallback to in-store payment when appropriate

### Stripe Customer Creation Issues

- Graceful degradation to one-time payments
- Error logging for debugging
- Customer notification through AI agent
- Manual customer creation fallback

## Dependencies

- **CreateStripeCustomerUsecase** - Stripe customer management
- **CreateShortLinkUsecase** - Secure URL generation
- **Payment method configuration** - Location-specific settings
- **Stripe API integration** - Payment processing infrastructure

## Configuration Requirements

### Location Payment Settings

Each location must configure:
- Available payment methods
- Stripe account connectivity
- Payment method priorities
- Currency settings

### Stripe Configuration

- Valid Stripe API keys
- Webhook endpoint configuration
- Payment method support settings
- Currency and region settings

This processor ensures secure, flexible payment handling while enabling the AI agent to provide smooth, conversational payment experiences for customers.