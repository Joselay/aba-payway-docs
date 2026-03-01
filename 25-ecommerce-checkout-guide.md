# eCommerce Checkout Integration Guide

## Overview

PayWay eCommerce Checkout enables merchants to accept payments on websites and mobile apps through credit/debit cards, ABA Pay, KHQR, WeChat Pay, Alipay, and Google Pay.

## Supported Use Cases

- Online product and service sales
- Digital wallet top-ups
- Subscription and utility bill processing
- On-demand service payments (delivery, ride-hailing)
- Event ticket and reservation bookings

## Payment Flow

1. Customer selects a product and initiates payment
2. Payment method selection triggers modal display
3. Payment completion through chosen method
4. System receives pushback notification with status
5. Payment verification and order confirmation

## Integration Steps

### Step 1: Create Payment Transaction

Call the [Purchase API](./01-purchase.md):

```
POST https://checkout-sandbox.payway.com.kh/api/payment-gateway/v1/payments/purchase
```

#### Web Implementation

```javascript
var form = document.getElementById('aba_merchant_request');
form.addEventListener('submit', function (event) {
  event.preventDefault();
  AbaPayway.checkout();
});
```

#### Mobile Implementation

Use parameters:
- `view_type=hosted_view` — Returns hosted checkout page
- `return_deeplink` — Handles native app redirection (Base64-encoded, mandatory for mobile)

### Step 2: Verify Payment Status

Use the [Check Transaction API](./02-check-transaction.md) with the transaction ID.

> Rate limit: 600 requests per second. Stop checking once a result is returned.

### Step 3: Handle Callback URL (Optional but Recommended)

The `return_url` parameter receives post-payment transaction details via HTTP POST with `Content-Type: application/json`. The domain must be whitelisted in the merchant profile.

#### Callback Payload

```json
{
    "tran_id": "17425401324",
    "apv": "619195",
    "status": "0",
    "return_params": "xxxxxxxxxx"
}
```

| Field | Description |
|-------|-------------|
| `tran_id` | Transaction ID from initial payment request |
| `apv` | Transaction approval code |
| `status` | Payment outcome indicator |
| `return_params` | Additional merchant-supplied data |
