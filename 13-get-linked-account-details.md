# Get Linked Account Details API

Retrieves linked account information when pushback notifications fail to deliver. This API only works for retrieving **account tokens** — it is not applicable for card tokens.

## Endpoint

```
POST /api/aof/pushback-status
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `req_time` | string | — | Yes | Request timestamp in UTC format: `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | Unique merchant identifier |
| `return_param` | string | 255 | Yes | Unique transaction request identifier |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $merchant_id . $req_time . $return_param;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

### `data` Object

| Field | Type | Description |
|-------|------|-------------|
| `ctid` | string | Customer token (unique per customer) |
| `pwt` | string | Payment gateway-generated token |
| `mask_account` | string | Masked account displaying final 4 digits |
| `expired_in` | number | Token expiration timestamp |

### `status` Object

| Field | Type | Description |
|-------|------|-------------|
| `code` | string | Response code |
| `message` | string | Details corresponding to the code |
| `tran_id` | string | Transaction identifier |

## Status Codes

| Code | Description |
|------|-------------|
| `403` | Not Found |
| `PW59` | Invalid Merchant Profile |
| `04` | Parameter Validation Required |
