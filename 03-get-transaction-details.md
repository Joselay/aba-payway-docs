# Get Transaction Details API

Retrieves details of a purchase transaction, including its history, for both online and in-store payments. This API does not support real-time payment status checks during processing — use [Check Transaction](./02-check-transaction.md) for that.

## Endpoint

```
POST /api/payment-gateway/v1/payments/transaction-detail
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

**Rate Limit**: 10 requests per minute (cannot be increased)

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `req_time` | string | — | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | Unique merchant key provided by ABA Bank |
| `tran_id` | string | 20 | Yes | The purchase transaction ID |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $req_time . $merchant_id . $tran_id;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

```json
{
  "data": { ... },
  "status": { ... }
}
```

### `data` Object

| Field | Type | Description |
|-------|------|-------------|
| `transaction_id` | string | Your `tran_id` |
| `payment_status_code` | number | `0` = APPROVED/PRE-AUTH, `2` = PENDING, `3` = DECLINED, `4` = REFUNDED, `7` = CANCELLED |
| `payment_status` | string | `APPROVED`, `PRE-AUTH`, `PENDING`, `DECLINED`, `REFUNDED`, or `CANCELLED` |
| `original_amount` | number | Original transaction amount before discount |
| `original_currency` | string | Original transaction currency |
| `payment_amount` | number | Amount the customer paid |
| `payment_currency` | string | Currency customer used to pay |
| `total_amount` | number | Amount customer supposed to pay after discount |
| `refund_amount` | number | Total refunded amount |
| `discount_amount` | number | Discount amount in original currency |
| `apv` | string | Transaction approval code |
| `transaction_date` | string | Creation date in payment gateway database |
| `first_name` | string | Payer's first name |
| `last_name` | string | Payer's last name |
| `email` | string | Payer's email |
| `phone` | string | Payer's phone number |
| `bank_ref` | string | Unique booking entry reference from ABA Core banking |
| `payment_type` | string | See payment types below |
| `payer_account` | string | Masked ABA Account Number or Masked Card PAN |
| `bank_name` | string | ABA Bank for ABA PAY, or issuer bank name for KHQR |
| `card_source` | string | `ONUS` (ABA card), `OFFUS_DOMESTIC` (local card), `OFFUS_INTERNATIONAL` (international card) |
| `transaction_operations` | array | History of payment operations |

### `transaction_operations` Array Items

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | `Completed`, `Pre-Auth`, `Completed Pre-Auth`, `Cancelled Pre-Auth`, or `Refunded` |
| `amount` | number | Amount based on operation type |
| `transaction_date` | string | Operation date and time |
| `bank_ref` | string | Unique booking entry ID (ABA PAY only) |

### Payment Types

| Value | Description |
|-------|-------------|
| `ABA Pay` | Transaction with ABA Account (ABA Mobile) |
| `Alipay` | Transaction with Alipay |
| `Wechat` | Transaction with WeChat Pay |
| `KHQR` | Transaction with KHQR |
| `VISA` | Transaction with Visa card |
| `MC` | Transaction with Mastercard |
| `JCB` | Transaction with JCB card |
| `CUP` | Transaction with UPI card |

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success |
| `5` | Wrong hash |
| `6` | Transaction not found |
| `8` | Invalid merchant profile |
| `11` | Internal server error |
| `429` | Rate limit exceeded |
