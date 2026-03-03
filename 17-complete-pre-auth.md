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
| `00` | Transaction successful. |
| `PTL02` | Invalid hash value. |
| `PTL04` | Parameter validation failed. |
| `PTL06` | The request has expired. |
| `PTL36` | Invalid transaction. |
| `PTL59` | Unable to complete or cancel the pre-authorization. |
| `PTL60` | Pre-authorization completion amount exceeds the authorized limit. |
| `PTL61` | Invalid action type. |
| `PTL62` | Merchant information is invalid. |
| `PTL63` | The merchant does not have a security configuration file. |
| `PTL153` | Completing pre-authorization fees for a merchant with multiple settlement accounts is not allowed. |
| `PTL157` | An unexpected error occurred. Please try again later or contact our digital support team. |
| `PTL168` | Concurrent requests are not allowed for this operation. Please try again in a few seconds. |
| `PTL169` | The merchant profile cannot accept payments because the settlement account is closed. |
| `USD-NOT-ALLOW` | The requested amount is not allowed for USD transactions. |
| `KHR-LESS-100` | The transaction amount in KHR must be at least 100 KHR. |
| `KHR-CONTAIN-DECIMAL` | KHR transaction amounts cannot contain decimal places. |
