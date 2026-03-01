# Complete Pre-Auth Transaction API

Captures funds following an initial pre-authorization, typically when products or services are delivered.

## Endpoint

```
POST /api/merchant-portal/merchant-access/online-transaction/pre-auth-completion
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Key Conditions

- Pre-auth completion can occur only **once** per transaction
- Expired or canceled transactions cannot be completed
- Card payments allow completion at up to **110%** of the original pre-auth amount
- If not completed or cancelled within **30 days** (default), the transaction auto-cancels

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `request_time` | string | — | Yes | Request timestamp in UTC format: `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | Unique merchant identifier |
| `merchant_auth` | string | — | Yes | RSA-encrypted, Base64-encoded JSON object |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash of `merchant_auth` + `request_time` + `merchant_id` |

### `merchant_auth` Object (before encryption)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mc_id` | string | Yes | Same as `merchant_id` |
| `tran_id` | string | Yes | Pre-auth transaction ID to complete |
| `complete_amount` | decimal | Yes | Amount to capture |

## Response

**HTTP 200**

| Field | Type | Description |
|-------|------|-------------|
| `grand_total` | number | Original pre-auth authorized amount |
| `currency` | string | Transaction currency |
| `transaction_status` | string | `COMPLETED` on success |
| `status.code` | string | Response code |
| `status.message` | string | Descriptive message |

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success |
| `PTL02` | Invalid hash |
| `PTL04` | Parameter validation failed |
| `PTL06` | Request expired |
| `PTL36` | Invalid transaction |
| `PTL59` | Unable to complete pre-authorization |
| `PTL60` | Completion amount exceeds authorized limit |
| `PTL61` | Invalid action type |
| `PTL62` | Invalid merchant information |
| `PTL63` | Missing security configuration |
| `PTL153` | Multiple settlement accounts prevent completion |
| `PTL157` | Unexpected error |
| `PTL168` | Concurrent requests blocked |
| `PTL169` | Settlement account closed |
| `USD-NOT-ALLOW` | USD amount not permitted |
| `KHR-LESS-100` | KHR must exceed 100 |
| `KHR-CONTAIN-DECIMAL` | KHR cannot include decimals |
