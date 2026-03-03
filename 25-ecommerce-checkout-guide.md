# eCommerce Checkout Integration Guide

## Overview

PayWay eCommerce Checkout enables merchants to accept payments on websites and mobile apps through credit/debit cards, ABA Pay, KHQR, WeChat Pay, Alipay, and Google Pay.

## Supported Use Cases

- Online shopping for products and services
- Wallet top-ups and digital services
- Subscriptions and bills (recurring payments, utility bills)
- On-demand services (food delivery, ride-hailing)
- Event bookings and ticket/reservation payments

## Payment Flow

1. Customer selects product/service and clicks "Pay"
2. Customer chooses payment method; checkout modal appears
3. Customer completes payment using selected method
4. System receives pushback notification with payment status
5. System verifies payment and confirms order

> For eCommerce platform sellers, see the eCommerce Checkout Plugins instead.

## Payment Selection UI

Merchants must follow PayWay eCommerce checkout guidelines. The checkout UI should include:

- A section for customers to choose payment options
- A "We Accept..." area displaying offered payment methods

UI guidelines are available via Figma:

- **Web UI Guidelines** — For website payments
- **Mobile UI Guidelines** — For app/web app payments

## Setup Requirements

Before integration, you need:

- **PayWay Sandbox Account** — Register at `sandbox.payway.com.kh/register-sandbox/`
- **Sandbox Merchant ID**
- **API Key** — Delivered via email after registration

## Integration Steps

### Step 1: Create a Payment Transaction

When the customer selects "Pay" and chooses a payment method, call the [Purchase API](./01-purchase.md) to create a transaction:

```
POST https://checkout-sandbox.payway.com.kh/api/payment-gateway/v1/payments/purchase
```

#### Sample HTML Request

```html
<html lang="en">
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no" />
    <script src="https://checkout.payway.com.kh/plugins/checkout2-0.js" defer></script>
  </head>
  <body>
    <form method="POST" target="aba_webservice" id="aba_merchant_request"
      action="https://checkout-sandbox.payway.com.kh/api/payment-gateway/v1/payments/purchase" >
      <input type="hidden" name="hash" value="D8SaUWAA/AhxNro00wAykb4ibeo9kM3if7ioN7cnBfihXP/38anLGwGUxHK+J6HvaiUEV8Ho+nz5nkQrzowm7g==" />
      <input id="tran_id" type="hidden" name="tran_id" value="17536691884" /><br />
      <input type="hidden" name="amount" value="0.10" />
      <input type="hidden" name="merchant_id" value="ec000002" />
      <input type="hidden" name="req_time" value="20250728022056" />
      <input type="hidden" id="payment_option" name="payment_option" value="" />
      <input type="hidden" name="currency" value="" />
      <input type="hidden" name="firstname" value="sina" />
      <input type="hidden" name="lastname" value="chhum" />
      <input type="hidden" name="phone" value="093939399" />
      <input type="submit" value="submit" />
    </form>

    <script>
      var form = document.getElementById('aba_merchant_request')
      form.addEventListener('submit', function (event) {
        event.preventDefault()
        AbaPayway.checkout()
      })
    </script>
  </body>
</html>
```

> The PayWay plugin script (`checkout2-0.js`) is valid for both Sandbox and Production environments.

PayWay returns an HTML response containing the checkout interface for rendering on the merchant's platform.

#### Web Implementation

The response returns HTML with responsive design. Payment method checkout UI variants for web:

- `cards` — Credit/debit card interface
- `abapay` — ABA Pay interface
- `bakong` — KHQR (Bakong) interface
- `alipay` — Alipay interface
- `wechat` — WeChat Pay interface
- `google_pay` — Google Pay interface

#### Mobile Implementation

Key parameters for mobile:

- `view_type=hosted` — Returns hosted checkout page for mobile
- `return_deeplink` — Handles redirection for native iOS/hybrid apps for return after payment in ABA Mobile (Base64-encoded, mandatory for mobile)

#### Plugin vs. Hosted Mode

- **Plugin Mode**: Use `AbaPayway.checkout()` for bottom sheet on mobile or modal popup on desktop
- **Hosted Mode**: Use `document.getElementById(form_id).submit()` for hosted view

### Step 2: Verify Payment Status

Use the [Check Transaction API](./02-check-transaction.md) with the transaction ID to confirm payment success.

> Rate limit: 600 requests per second. Stop checking once a result is returned.

### Step 3: Handle Callback URL (Optional but Recommended)

After the customer completes payment, the payment gateway sends transaction details to the merchant's `return_url` via HTTP POST with `Content-Type: application/json`.

- Optional field; defaults to merchant profile's `return_url` if empty
- If provided, the domain must be whitelisted in the merchant profile

> We highly recommend securing this URL to ensure that only PayWay has access to it.

#### Callback Payload

```json
{
    "tran_id": "17425401324",
    "apv": "619195",
    "status": "0",
    "return_params": "xxxxxxxxxx"
}
```

| Field | Type | Description |
|---|---|---|
| `tran_id` | string | Transaction ID sent during initial payment process |
| `apv` | string | Transaction approval code |
| `status` | string | Payment status indicator |
| `return_params` | string | Extra information from payment initiation request |
