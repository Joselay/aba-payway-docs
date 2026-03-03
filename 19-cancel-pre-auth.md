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
| `00` | Success! |
| `PTL02` | Invalid hash provided. Ensure you are using the correct hash key. |
| `PTL04` | Parameter validation failed. Verify that all required fields are correctly formatted. |
| `PTL06` | The request has expired. Please generate a new request and retry. |
| `PTL36` | Invalid transaction. Ensure that the transaction ID is correct. |
| `PTL59` | Unable to complete or cancel Pre-auth. Check the transaction status before retrying. |
| `PTL60` | Pre-auth amount exceeds the allowed limit. Reduce the amount and try again. |
| `PTL61` | Invalid action type. Ensure you are using a valid operation type. |
| `PTL62` | Invalid merchant information. Verify your merchant ID and try again. |
| `PTL63` | Merchant does not have a security configuration file. Contact support for assistance. |
| `PTL157` | An unexpected error occurred. Please try again later or contact our digital support team. |
| `PTL168` | Concurrent requests are not allowed. Wait a few seconds and retry. |
| `PTL169` | The merchant profile cannot accept payments. Settlement account is closed. |
| `USD-NOT-ALLOW` | The requested amount is not permitted. Choose a valid amount. |
| `KHR-LESS-100` | KHR amount must be greater than 100 KHR. |
| `KHR-CONTAIN-DECIMAL` | Amount for KHR currency must be a whole number (no decimals allowed). |
