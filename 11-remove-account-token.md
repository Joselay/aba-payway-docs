# Remove Account Token API

Removes a customer's linked ABA account token. **This action is irreversible** — the token becomes permanently invalid for future transactions.

## Endpoint

```
POST /api/aof/remove-account
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `req_time` | string | — | Yes | UTC timestamp as `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | Unique merchant identifier |
| `ctid` | string | 250 | Yes | Customer token (unique per customer) |
| `pwt` | string | 250 | Yes | Payment gateway-generated token |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $merchant_id . $req_time . $ctid . $pwt;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

```json
{
  "status": {
    "code": "00",
    "message": "Success"
  }
}
```

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success |
| `04` | Missing required parameter |
| `29` | Invalid ctid or pwt |
| `500` | Server error |
