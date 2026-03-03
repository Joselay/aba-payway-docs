# Link Card API

The API returns HTML, allowing users to enter their credit/debit card details (Visa, Mastercard, JCB, and UPI) to link their card to your platform. Once the user has completed the linking process, PayWay will send the account details and token to the merchant via the `return_url`.

## Endpoint

```
POST /api/payment-gateway/v1/cof/initial
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `multipart/form-data`

## Prerequisites

Before using this API, please ensure that your profile has the Card on File feature enabled. If your merchant profile has not enabled this feature yet, please contact our merchant digital support at `digitalsupport@ababank.com` for a sandbox profile. For a production merchant profile, please contact our merchant acquisition team at `paywaysales@ababank.com`.

## Request Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `merchant_id` | string | Yes | A unique merchant key which provided by ABA Bank |
| `ctid` | string | No | Your consumer identification number |
| `return_param` | string | Yes | Extra information that you want to include when payment gateway calls your `return_url` |
| `firstname` | string | No | Your consumer first name |
| `lastname` | string | No | Your consumer last name |
| `email` | string | No | Your consumer email |
| `phone` | string | No | Your consumer phone |
| `return_url` | string | No | Once the user has linked their card, the details of the token and other important information will be sent via this URL. This is an optional field. If left empty, it will default to the merchant profile's `pushback_url`. If you provide a value, ensure that your domain is whitelisted in your merchant profile |
| `continue_add_card_success_url` | string | No | After linking their card, the user will see a success screen with a **Done** button. Your `continue_add_card_success_url` will be embedded in this button. When the user taps **Done**, they will be redirected to your platform |
| `hash` | string | Yes | Base64-encoded HMAC-SHA512 hash of the concatenated values: `merchant_id`, `ctid`, and `return_param` with `public_key` |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $merchant_id . $ctid . $return_param;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200** — Returns HTML content with the card entry form.
