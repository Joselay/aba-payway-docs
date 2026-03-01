# Update Beneficiary Status API

Toggles a beneficiary between active and inactive status.

## Endpoint

```
POST /api/merchant-portal/merchant-access/whitelist-account/update-whitelist-status
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Use Cases

- Prevent whitelisted beneficiaries from receiving future funds
- Resume previously disabled beneficiaries

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `request_time` | string | — | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | Unique merchant key from ABA Bank |
| `merchant_auth` | string | — | Yes | RSA-encrypted, Base64-encoded JSON object |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash of `request_time` + `merchant_auth` |

### `merchant_auth` Object (before encryption)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mc_id` | string | Yes | Same as `merchant_id` |
| `payee` | string | Yes | Beneficiary identifier: MID or ABA account |
| `status` | integer | Yes | `0` = disable, `1` = activate |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $request_time . $merchant_auth;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

### `data` Object

| Field | Type | Max Length | Description |
|-------|------|-----------|-------------|
| `name` | string | 255 | Beneficiary name |
| `payee` | string | 250 | MID or ABA Account number |
| `currency` | string | 10 | Currency |
| `type` | string | 20 | `Merchant` or `ABA Account` |
| `status` | integer | — | `1` = Active, `0` = Inactive |
| `created_at` | string | — | Date and time created |

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success |
| `PTL02` | Wrong hash |
| `PTL04` | Parameter validation required |
| `PTL46` | Merchant not found |
| `PTL149` | Invalid whitelist account |
| `PTL150` | Business profile not found |
