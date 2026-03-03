# Purchase Using Token API

This API supports both card tokens and account tokens.

## Endpoint

```
POST /api/payment-gateway/v1/payments/purchase
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `req_time` | string | — | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | A unique merchant key provided by ABA Bank |
| `tran_id` | string | 20 | Yes | A unique transaction ID for the payment |
| `amount` | number | — | Yes | Total purchase amount (exclude shipping fee) |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash of concatenated values `req_time`, `merchant_id`, `tran_id`, `amount`, `items`, `shipping`, `ctid`, `pwt`, `firstname`, `lastname`, `email`, `phone`, `type`, `return_url`, `currency`, `custom_fields`, `return_params`, and `payout`, using `public_key` |
| `ctid` | string | 255 | Yes* | Your consumer token |
| `pwt` | string | 255 | Yes* | PayWay-issued token |
| `firstname` | string | 100 | No | Buyer's first name |
| `lastname` | string | 100 | No | Buyer's last name |
| `email` | string | 50 | No | Buyer's email |
| `phone` | string | 20 | No | Buyer's phone |
| `type` | string | 20 | No | The type of transaction. The default value is `purchase`. Supported values: `pre-auth` (pre-authorization), `purchase` (full purchase transaction) |
| `items` | string | 500 | No | A base64-encoded JSON array listing the items included in the transaction |
| `shipping` | number | — | No | Shipping fee |
| `return_url` | string | — | No | URL to receive callbacks upon payment completion, encrypted with Base64 |
| `custom_fields` | string | — | No | Additional information you want to attach to the transaction. This information appears in transaction details, lists, and export reports. Must be base64-encoded JSON |
| `return_params` | string | — | No | Information to include when PayWay calls your return URL after payment completed |
| `payout` | string | — | No | Base64-encoded JSON string representing payout details |

> *Both `ctid` and `pwt` are nullable but must be provided for token-based purchases.

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $req_time . $merchant_id . $tran_id . $amount . $items . $shipping . $ctid . $pwt . $firstname . $lastname . $email . $phone . $type . $return_url . $currency . $custom_fields . $return_params . $payout;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

### Request Example

```json
{
  "req_time": "20250312075529",
  "merchant_id": "xxxxx",
  "type": "pre-auth",
  "items": "Nlx1MTc5NFx1MTdiY...MDAwLjAwIn1d",
  "amount": 60000,
  "tran_id": "17417661239",
  "continue_success_url": "demo-payway-uat.ababank.com",
  "return_url": "aHR0cHM6Ly9kZW1vLXBheXdhe..NzY2MTIzOQ==",
  "return_param": "OTg0OQ==",
  "hash": "QRzyIlknvaVA..jXvkA==",
  "custom_fields": "eyJteV9jdXN0bMSI6I..lfY3VzdG9tX2ZpZWDE3NjYxMjl9",
  "firstname": "QA",
  "lastname": "Sakada",
  "phone": "017582717",
  "email": "sakadaqa@gmail.com"
}
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

**HTTP 403**

```json
{
  "status": {
    "code": "26",
    "message": "Invalid Merchant Profile."
  }
}
```

## Error Codes

| Code | Description |
|------|-------------|
| `1` | Wrong Hash |
| `2` | Invalid Transaction ID |
| `3` | Invalid Transaction Amount |
| `4` | Duplicated Transaction ID |
| `5` | Transaction not found |
| `6` | Requested Domain is not in whitelist |
| `7` | Wrong return param |
| `8` | Data saving error |
| `10` | Wrong shipping price |
| `11` | General error |
| `12` | Payment currency not allowed |
| `13` | Invalid items |
| `14` | Invalid Credit Multi Acc |
| `15` | Invalid channel values |
| `16` | Invalid First Name |
| `17` | Invalid Last Name |
| `18` | Invalid Phone |
| `19` | Invalid Email |
| `20` | Contact merchant |
| `21` | API lifetime ended |
| `22` | Pre-Auth not enabled |
| `23` | Payment option disabled |
| `24` | Decryption failed |
| `25` | Max 10 payouts exceeded |
| `26` | Invalid Merchant Profile |
| `27` | Invalid ctid |
| `28` | Invalid pwt |
| `29` | Invalid pwt or ctid |
| `30` | Merchant not COF enabled |
| `31` | Unsecure 3DS page |
| `32` | Cannot identify cardOrigin |
| `33` | Invalid exchange rate |
| `34` | Invalid payout info |
| `35` | Invalid payout account/amount |
| `36` | Payout accounts not whitelisted |
| `37` | Invalid Transaction ID in payout |
| `38` | Duplicate account in payout |
| `39` | Duplicate Transaction ID in payout |
| `40` | MID not linked |
| `41` | Invalid account status |
| `42` | Missing MID |
| `43` | Transaction limit reached |
| `44` | Zero amount not allowed |
| `45` | KHR cannot contain decimals |
| `46` | KHR must exceed 100 |
| `47` | Invalid parameters |
| `48` | Invalid Start Date |
| `49` | Invalid End Date |
| `50` | Invalid Date Range |
| `51` | Max 3-day range |
| `52` | Invalid Amount Range |
| `53` | Transaction expired |
| `54` | WeChat QR request failed |
| `55` | WeChat validation failed |
| `56` | Card validation failed |
| `57` | Invalid card number |
| `58` | Payout cannot be fixed |
| `59` | QR string error |
| `60` | General error |
| `61` | QR in use |
| `62` | Transaction exists in core |
| `63` | Same account error |
| `64` | MID not in core |
| `65` | General error |
| `66` | QR on invoice unavailable |
| `67` | Transaction expired |
| `68` | Lifetime too short |
| `69` | Daily limit reached |
| `70` | Transaction limit reached |
| `71` | Settlement account closed |
| `72` | Invalid Transaction Status |
| `73` | Invalid tran_id/merchant_id |
| `74` | tran_id not found |
| `75` | Invalid parameters |
| `76` | No transaction fees |
| `77` | Incompatible with discount |
| `78` | Google Pay token missing |
| `79` | Google Pay decryption failed |
| `80` | Return URL not whitelisted |
| `81` | Payout max amount exceeded |
| `82` | Credential disabled |
| `83` | Credential expired |
| `84` | Purchase limit reached |
| `85` | Unsupported mode |
| `86` | Credential removed |
| `200` | Canceled |
| `201` | Declined |
| `401` | Unauthorized |
| `403` | General error |
| `429` | Too many requests |
| `503` | Maintenance |
