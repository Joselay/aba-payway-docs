# Cancel Pre-Auth Transaction API

Releases temporary holds on customer payment methods before final transaction completion.

## Endpoint

```
POST /api/merchant-portal/merchant-access/online-transaction/pre-auth-cancellation
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Important Notes

- Pre-authorizations can only be cancelled while pending
- Each transaction can be cancelled only once
- Successful cancellations update status to `CANCELLED`
- ABA PAY and Card transactions release funds instantly
- KHQR transactions process refunds

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `request_time` | string | — | Yes | Request date/time in UTC format: `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | Unique merchant key |
| `merchant_auth` | string | — | Yes | RSA-encrypted, Base64-encoded JSON object |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash of `merchant_id` + `merchant_auth` + `request_time` |

### `merchant_auth` Object (before encryption)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mc_id` | string | Yes | Same as `merchant_id` |
| `tran_id` | string | Yes | Pre-auth transaction ID to cancel |

## Response

**HTTP 200**

| Field | Type | Description |
|-------|------|-------------|
| `grand_total` | number | Original pre-auth authorized amount |
| `currency` | string | Original transaction currency |
| `transaction_status` | string | `CANCELLED` |
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
| `PTL59` | Unable to cancel pre-auth |
| `PTL60` | Pre-auth amount exceeds limit |
| `PTL61` | Invalid action type |
| `PTL62` | Invalid merchant information |
| `PTL63` | Missing security configuration |
| `PTL157` | Unexpected error |
| `PTL168` | Concurrent requests not allowed |
| `PTL169` | Settlement account closed |
| `USD-NOT-ALLOW` | USD amount not permitted |
| `KHR-LESS-100` | KHR must exceed 100 |
| `KHR-CONTAIN-DECIMAL` | KHR cannot include decimals |
