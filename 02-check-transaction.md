# Check Transaction API

This API allow you to get the transaction status of a transaction, you can only check the transaction that created within 7 days only. To get a details of a transaction which is older than 7 days please use [Get a transaction details](./03-get-transaction-details.md) API.

## Endpoint

```
POST /api/payment-gateway/v1/payments/check-transaction-2
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

**Rate Limit**: Request limit 600 reqeusts/second

## Request Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `req_time` | string | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | Yes | A unique merchant key which provided by ABA Bank |
| `tran_id` | string | Yes | Your purchase transaction id |
| `hash` | string | Yes | Base64 encode of hash hmac sha512 encryption of concatenates values `merchant_id`, and `tran_id` with `public_key` |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $req_time . $merchant_id . $tran_id;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

### Request Example

```json
{
  "req_time": "20250213065545",
  "merchant_id": "ec000002",
  "tran_id": "17394277693",
  "hash": "4slqXzgVig09Hf...2vgALgdENA=="
}
```

## Response

**HTTP 200**

```json
{
    "data": {
        "payment_status_code": 0,
        "total_amount": 0.1,
        "original_amount": 0.1,
        "refund_amount": 0,
        "discount_amount": 0,
        "payment_amount": 0.1,
        "payment_currency": "USD",
        "apv": "753786",
        "payment_status": "APPROVED",
        "transaction_date": "2025-02-13 13:55:25"
    },
    "status": {
        "code": "00",
        "message": "Success!",
        "tran_id": "17394277693"
    }
}
```

### `data` Object

| Field | Type | Description |
|-------|------|-------------|
| `payment_status_code` | integer | Transaction status code: `0` = APPROVED/PRE-AUTH, `2` = PENDING, `3` = DECLINDED, `4` = REFUNDED, `7` = CANCELLED |
| `payment_status` | string | `APPROVED` – Transaction successfully completed with the full purchase amount, `PRE-AUTH` – Transaction successfully processed with a pre-authorization hold on funds pending final capture, `REFUNDED` – Transaction has been fully or partially refunded, `PENDING` – Transaction is awaiting payment completion by the payer, `DECLINED` – Transaction has been declined, `CANCELLED` – Merchant canceled the pre-authorization or closed the transaction |
| `total_amount` | number | Amount that customer suppose to pay after discount |
| `original_amount` | number | Original purchase amount |
| `refund_amount` | number | Total of all refunded amount |
| `discount_amount` | number | Discounted amount and its currency follow the original transaction currency |
| `payment_amount` | number | The amount that customer has paid |
| `payment_currency` | string | The payment currency that the customer has paid |
| `apv` | string | Transaction approval code |
| `transaction_date` | string | Date and time of the transaction created in payment gateway |

### `status` Object

| Field | Type | Description |
|-------|------|-------------|
| `code` | string | Response code. See status codes below |
| `message` | string | Please see the property reponse `code` for the details |
| `tran_id` | string | Unique request id auto nenerated by payment gateway |

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success! |
| `5` | Invalid hash |
| `6` | Transaction not found |
| `8` | Invalid merchant profile |
| `11` | Internal server error |
| `429` | Reach request limit |
