# Get Transactions by Merchant Reference API

Retrieves purchase transactions using the `merchant_ref` identifier. Returns only the last 50 transactions.

## Endpoint

```
POST /api/payment-gateway/v1/payments/get-transactions-by-mc-ref
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

**Rate Limit**: 10 requests per minute (cannot be increased)

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `req_time` | string | — | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | Unique merchant key from ABA Bank |
| `merchant_ref` | string | 20 | Yes | Your merchant reference number |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash of `req_time` + `merchant_id` + `merchant_ref` |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $req_time . $merchant_id . $merchant_ref;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

### `data` Array — Transaction Objects

| Field | Type | Description |
|-------|------|-------------|
| `transaction_id` | string | Transaction identifier |
| `transaction_date` | string | Created date & time |
| `apv` | string | Transaction approval code |
| `payment_status` | string | `APPROVED` or `REFUNDED` |
| `payment_status_code` | integer | `0` = APPROVED, `4` = REFUNDED |
| `original_amount` | number | Original amount before discount |
| `original_currency` | string | `KHR` or `USD` |
| `total_amount` | number | Amount after discount |
| `discount_amount` | number | Discount applied |
| `refund_amount` | number | Total refunded amount |
| `payment_amount` | number | Actual customer payment |
| `payment_currency` | string | Currency customer paid with |
| `bank_ref` | string | Unique booking entry from ABA core banking |
| `payer_account` | string | Masked account number |
| `bank_name` | string | Originating bank name |
| `payment_type` | string | `ABA Pay` or `KHQR` |
| `merchant_ref` | string | Your reference number |

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success |
| `1` | Wrong hash |
| `8` | Invalid merchant profile |
| `11` | Server error |
| `429` | Rate limit exceeded |
