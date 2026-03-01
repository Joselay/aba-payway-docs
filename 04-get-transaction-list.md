# Get Transaction List API

Retrieves a list of transactions filtered by specific criteria such as transaction date, amount, payment type, and more. Supports pagination for both in-store and online profiles.

## Endpoint

```
POST /api/payment-gateway/v1/payments/transaction-list-2
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

**Rate Limit**: 50 requests per minute

## Constraints

- Outlet-specific only — cannot retrieve all transactions across outlets
- Maximum 3-day date range (inclusive of current day)
- Hash parameters must follow exact sequence

## Request Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `req_time` | string | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | Yes | Unique merchant key from ABA Bank |
| `from_date` | string | No | Start date filter: `YYYY-MM-DD HH:mm:ss` |
| `to_date` | string | No | End date filter: `YYYY-MM-DD HH:mm:ss` |
| `from_amount` | number (double) | No | Minimum purchase amount filter |
| `to_amount` | number (double) | No | Maximum purchase amount filter |
| `status` | string | No | Comma-separated: `APPROVED`, `PRE-AUTH`, `REFUNDED`, `PENDING`, `DECLINED`, `CANCELLED` |
| `page` | string | No | Current page index (default: `1`) |
| `pagination` | string | No | Records per page (default: `40`, max: `1000`) |
| `hash` | string | Yes | Base64-encoded HMAC-SHA512 hash |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $req_time . $merchant_id . $from_date . $to_date . $from_amount . $to_amount . $status . $page . $pagination;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

```json
{
  "data": [ ... ],
  "page": "1",
  "pagination": "40",
  "status": {
    "code": "00",
    "message": "Success!",
    "tran_id": "..."
  }
}
```

### `data` Array — Transaction Object

| Field | Type | Description |
|-------|------|-------------|
| `transaction_id` | string | Transaction ID |
| `transaction_date` | string | Created date & time |
| `apv` | string | Transaction approval code |
| `payment_status` | string | `APPROVED`, `PRE-AUTH`, `REFUNDED`, `PENDING`, `DECLINED`, or `CANCELLED` |
| `payment_status_code` | integer | `0` = APPROVED/PRE-AUTH, `2` = PENDING, `3` = DECLINED, `4` = REFUNDED, `7` = CANCELLED |
| `original_amount` | number | Original amount before discount |
| `original_currency` | string | `KHR` or `USD` |
| `total_amount` | number | Amount after discount |
| `discount_amount` | number | Discounted amount |
| `refund_amount` | number | Total refunded amount |
| `payment_amount` | number | Amount customer paid (may differ due to currency conversion) |
| `payment_currency` | string | Payment currency |
| `first_name` | string | Payer first name |
| `last_name` | string | Payer last name |
| `email` | string | Payer email |
| `phone` | string | Payer phone |
| `bank_ref` | string | Unique booking entry ID from ABA core banking |
| `payer_account` | string | Masked ABA Account Number or Card PAN |
| `bank_name` | string | ABA Bank, issuer bank name (KHQR), or other |
| `card_source` | string | `ONUS`, `OFFUS_DOMESTIC`, or `OFFUS_INTERNATIONAL` |
| `payment_type` | string | `N/A`, `ABA Pay`, `Alipay`, `Wechat`, `KHQR`, `VISA`, `MC`, `JCB`, or `CUP` |

### Pagination Fields

| Field | Type | Description |
|-------|------|-------------|
| `page` | string | Current page index |
| `pagination` | string | Records per page |

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success |
| `1` | Wrong hash |
| `8` | Invalid merchant profile |
| `11` | Internal server error |
| `429` | Rate limit exceeded |
