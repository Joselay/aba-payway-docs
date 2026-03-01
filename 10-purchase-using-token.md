# Purchase Using Token API

Processes a payment using stored payment credentials (card token or account token).

## Endpoint

```
POST /api/payment-gateway/v1/payments/purchase
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `req_time` | string | — | Yes | Request timestamp in UTC format: `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | Unique merchant identifier |
| `tran_id` | string | 20 | Yes | Unique transaction identifier |
| `amount` | number | — | Yes | Total purchase amount (excludes shipping) |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash |
| `ctid` | string | 255 | Yes* | Consumer token (unique per customer) |
| `pwt` | string | 255 | Yes* | PayWay-issued token |
| `firstname` | string | 100 | No | Buyer's first name |
| `lastname` | string | 100 | No | Buyer's last name |
| `email` | string | 50 | No | Buyer's email address |
| `phone` | string | 20 | No | Buyer's phone number |
| `type` | string | 20 | No | `pre-auth` or `purchase` (default) |
| `items` | string | 500 | No | Base64-encoded JSON array of items |
| `shipping` | number | — | No | Shipping fee |
| `return_url` | string | — | No | Base64-encoded callback URL |
| `custom_fields` | string | — | No | Base64-encoded JSON with additional data |
| `return_params` | string | — | No | Data included in return URL callback |
| `payout` | string | — | No | Base64-encoded JSON payout details (max 10 accounts) |

> *Both `ctid` and `pwt` must be provided for token-based purchases.

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $req_time . $merchant_id . $tran_id . $amount . $items . $shipping . $ctid . $pwt . $firstname . $lastname . $email . $phone . $type . $return_url . $currency . $custom_fields . $return_params . $payout;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Success Response

**HTTP 200**

```json
{
  "tran_id": "trx-20201019130949",
  "payment_status": {
    "status": "0",
    "code": "CDA00",
    "description": "OK",
    "pw_tran_id": "trx-20201019130949"
  }
}
```

## Error Response

```json
{
  "status": {
    "code": "26",
    "message": "Invalid Merchant Profile."
  }
}
```

## Error Codes

Same error codes as the [Purchase API](./01-purchase.md#error-codes).
