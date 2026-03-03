# Remove Card Token API

Removes a customer's linked card token. **This action cannot be undone** — the token is permanently invalidated.

## Endpoint

```
POST /api/payment-gateway/v1/cof/remove
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `merchant_id` | string | 20 | Yes | Unique merchant identifier |
| `ctid` | string | 250 | Yes | Customer identifier (unique per customer) |
| `pwt` | string | 250 | Yes | Payment gateway-generated token |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $merchant_id . $ctid . $pwt;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Example Request

```json
{
  "merchant_id": "ec000002",
  "ctid": "522235F63E4998...ABC270FC9FE07409F6ACD716B88",
  "pwt": "5222359B5394FF7B...259BFE3868B08461F",
  "hash": "xShjXEibzGYj...u7fpx2SLfEu.HR0eyNKAbg=="
}
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
| `1` | Invalid hash value |
| `11` | An unexpected error occurred. Please try again later or contact ABA support for assistance. |
| `26` | Merchant profile is invalid |
| `29` | Invalid ctid or pwt |
