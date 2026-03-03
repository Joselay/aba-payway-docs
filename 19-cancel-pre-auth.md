# Cancel Pre-Purchase Transaction

Cancel pre-auth (or cancel pre-authorization) is the process of releasing a temporary hold on funds placed on a customer's payment method before the final transaction is completed.

## Endpoint

```
POST /api/merchant-portal/merchant-access/online-transaction/pre-auth-cancellation
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Important Notes

- You can only cancel a pre-authorization if the transaction is still pending; if the pre-auth has already been completed or previously cancelled, it cannot be cancelled again.
- Each transaction's pre-authorization can be cancelled only once.
- Once the cancellation is successfully processed, the transaction status will update to 'CANCELLED.'
- For ABA PAY and Card transactions, funds are instantly released back to the payer, whereas for KHQR transactions, the funds will be refunded to the payer.

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `request_time` | string | — | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss`. |
| `merchant_id` | string | 20 | Yes | A unique merchant key which provided by ABA Bank. |
| `merchant_auth` | string | — | Yes | The JSON-encoded object containing `mc_id` and `tran_id` using RSA public key encryption in chunks. |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash of concatenated values: `merchant_id`, `merchant_auth`, and `request_time` with `public_key`. |

### `merchant_auth` Object (before encryption)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mc_id` | string | Yes | A unique merchant key which provided by ABA Bank. Same value as `merchant_id`. |
| `tran_id` | string | Yes | Pre-auth purcahse transaction id to cancel. |

## Response

**HTTP 200**

| Field | Type | Description |
|-------|------|-------------|
| `grand_total` | number | The original amount authorized for pre-auth transactions. |
| `currency` | string | Original transaction currency |
| `transaction_status` | string | Status of the transaction. After successfully cancelling, its status is `CANCELLED` |
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
| `PTL62` | Invalid merchant information. Verify your merchant ID and try again. |
| `PTL63` | Merchant does not have a security configuration file. Contact support for assistance. |
| `PTL59` | Unable to complete or cancel Pre-auth. Check the transaction status before retrying. |
| `PTL60` | Pre-auth amount exceeds the allowed limit. Reduce the amount and try again. |
| `PTL61` | Invalid action type. Ensure you are using a valid operation type. |
| `PTL157` | An unexpected error occurred. Please try again later or contact our digital support team. |
| `PTL168` | Concurrent requests are not allowed. Wait a few seconds and retry. |
| `PTL169` | The merchant profile cannot accept payments. Settlement account is closed. |
| `USD-NOT-ALLOW` | The requested amount is not permitted. Choose a valid amount. |
| `KHR-LESS-100` | KHR amount must be greater than 100 KHR. |
| `KHR-CONTAIN-DECIMAL` | Amount for KHR currency must be a whole number (no decimals allowed). |
