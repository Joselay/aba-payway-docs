# Link Account API

Links a customer's ABA Bank account for future payments. Generates a QR code for desktop or a deeplink for mobile that the customer uses to authorize the account linking via ABA Mobile.

## Endpoint

```
POST /api/aof/request-qr
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Prerequisites

Account on File must be enabled on your merchant profile. Contact `digitalsupport@ababank.com` (sandbox) or `paywaysales@ababank.com` (production).

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `req_time` | string | — | Yes | Request timestamp in UTC format: `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | Unique merchant identifier |
| `return_param` | string | — | Yes | Custom data included in gateway callback |
| `return_url` | string | — | No | URL receiving linked account details; defaults to merchant profile's `pushback_url` |
| `return_deeplink` | string | — | No | Base64-encoded JSON with `ios_scheme` and `android_scheme` for post-linking redirect |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $merchant_id . $req_time . $return_deeplink;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

| Field | Type | Description |
|-------|------|-------------|
| `deeplink` | string | Mobile app deep link for Android/iOS |
| `qr_string` | string | QR code data string for web browser integration |
| `qr_image` | string | Complete URL to QR image file |
| `expire_in` | number | Token expiration timestamp |
| `status.code` | string | Response code |
| `status.message` | string | Status description |

## Callback Response (POST to `return_url`)

After the customer links their account, PayWay sends:

| Field | Description |
|-------|-------------|
| `tran_id` | Transaction identifier |
| `ctid` | Consumer Token Identification (unique per customer) |
| `pwt` | PayWay Token (identifies the ABA account) |
| `mask_account` | Masked account showing last 4 digits |
| `expired_in` | Expiration timestamp |

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success |
| `04` | Missing required parameter |
| `11` | Server error |
