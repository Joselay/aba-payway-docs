# Link Card API

Links a customer's credit/debit card (Visa, Mastercard, JCB, UPI) for future payments. Returns HTML content containing a card entry form. A $0.01 verification charge is processed and immediately refunded.

## Endpoint

```
POST /api/payment-gateway/v1/cof/initial
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `multipart/form-data`

## Prerequisites

Card on File must be enabled on your merchant profile. Contact `digitalsupport@ababank.com` (sandbox) or `paywaysales@ababank.com` (production).

## Request Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `merchant_id` | string | Yes | Unique merchant identifier from ABA Bank |
| `ctid` | string | No | Consumer identification number |
| `return_param` | string | Yes | Supplementary data included in callback |
| `firstname` | string | No | Consumer's first name |
| `lastname` | string | No | Consumer's last name |
| `email` | string | No | Consumer's email address |
| `phone` | string | No | Consumer's phone number |
| `return_url` | string | No | URL receiving token/details post-linking; defaults to merchant `pushback_url`. Domain must be whitelisted |
| `continue_add_card_success_url` | string | No | Redirect URL after success screen confirmation |
| `hash` | string | Yes | Base64-encoded HMAC-SHA512 hash |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $merchant_id . $ctid . $return_param;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200** — Returns HTML content with the card entry form rendered via PayWay Plugin JS.

## Callback Response (POST to `return_url`)

After the card is linked, PayWay sends:

| Field | Description |
|-------|-------------|
| `pwt` | PayWay Token for the card |
| `mask_pan` | Masked PAN (last 4 digits) |
| `card_type` | `Visa`, `MC`, `JCB`, or `CUP` |
| `ctid` | Consumer Token Identification |
