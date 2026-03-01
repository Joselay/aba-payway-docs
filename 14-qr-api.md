# QR API (Generate QR Code)

Generates dynamic QR codes for payments via ABA KHQR, WeChat Pay, or Alipay.

## Endpoint

```
POST /api/payment-gateway/v1/payments/generate-qr
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Use Cases

- Cashier screens & POS systems
- Self-service kiosks
- QR codes on invoices/receipts

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `req_time` | string | — | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | 30 | Yes | Unique merchant key provided by ABA Bank |
| `tran_id` | string | 20 | Yes | Unique transaction identifier |
| `amount` | number | — | Yes | Total amount (min: 100 KHR or 0.01 USD) |
| `currency` | string | 3 | Yes | `KHR` or `USD` |
| `payment_option` | string | 20 | Yes | `abapay_khqr`, `wechat` (USD only), or `alipay` (USD only) |
| `lifetime` | integer | — | Yes | Transaction lifetime in minutes (min: 3, max: 43200 / 30 days). Default: 30 days |
| `qr_image_template` | string | 20 | Yes | QR image template: `template1`, `template1_color`, `template2`, `template2_color`, `template3_color`, `template4`, `template4_color`, `template5`, `template5_color`, `template6_color` |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash |
| `first_name` | string | 20 | No | Payer's first name |
| `last_name` | string | 20 | No | Payer's last name |
| `email` | string | 50 | No | Payer's email address |
| `phone` | string | 20 | No | Payer's phone number |
| `purchase_type` | string | 20 | No | `pre-auth` or `purchase` (default) |
| `items` | string | 500 | No | Base64-encoded JSON item list (max 10 items) |
| `callback_url` | string | 255 | No | Base64-encoded URL to receive payment callbacks |
| `return_deeplink` | string | 255 | No | Base64-encoded deeplink with Android and iOS schemes |
| `custom_fields` | string | 255 | No | Base64-encoded additional custom fields |
| `return_params` | string | — | No | Additional info included in pushback on payment completion |
| `payout` | string | 255 | No | Base64-encoded JSON payout instructions |

> **Note:** Alipay & WeChat do not support pre-auth.

## Hash Generation

Concatenate all parameter values then HMAC-SHA512 with API key and Base64-encode.

> **Note:** The hash field order differs from the parameter table order above. The explicit concatenation order from the remote docs is:
>
> `req_time, merchant_id, tran_id, amount, items, first_name, last_name, email, phone, purchase_type, payment_option, callback_url, return_deeplink, currency, custom_fields, return_params, payout, lifetime, qr_image_template`

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $req_time . $merchant_id . $tran_id . $amount . $items . $first_name . $last_name . $email . $phone . $purchase_type . $payment_option . $callback_url . $return_deeplink . $currency . $custom_fields . $return_params . $payout . $lifetime . $qr_image_template;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Field Format Examples

**items** (Base64-encoded JSON):
```json
[
  {"name": "Item 1", "quantity": "1", "price": "0.01"},
  {"name": "Item 2", "quantity": "1", "price": "0.02"}
]
```

**payout** (Base64-encoded JSON):
```json
[
  {"account": "003811397", "amount": "1.00", "tran_id": "payout_trx_001"}
]
```

**return_deeplink** (Base64-encoded JSON):
```json
{"android": "payway://pay", "ios": "payway://pay"}
```

**custom_fields** (Base64-encoded JSON):
```json
{"key1": "value1", "key2": "value2"}
```

**callback_url** (Base64-encoded): The URL that receives payment notifications.

**return_params**: Additional info included in pushback on payment completion (plain string, not Base64-encoded).

## Request Example

```json
{
  "hash": "base64-encoded-hmac-sha512-hash",
  "req_time": "20250301120000",
  "merchant_id": "your_merchant_id",
  "tran_id": "trx-20250301-001",
  "amount": 1.00,
  "currency": "USD",
  "payment_option": "abapay_khqr",
  "lifetime": 30,
  "qr_image_template": "qr_template1",
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "phone": "012345678",
  "items": "base64-encoded-items-json"
}
```

## Response

**HTTP 200**

| Field | Type | Description |
|-------|------|-------------|
| `status.code` | string | Response code |
| `status.message` | string | Descriptive message |
| `status.trace_id` | string | Unique request trace identifier |
| `amount` | number | Transaction amount |
| `currency` | string | Transaction currency |
| `qrString` | string | QR content as string |
| `qrImage` | string | QR as base64 image (can be ~0.5 MB for high resolution) |
| `abapay_deeplink` | string | ABA Mobile deep link |
| `app_store` | string | App Store download link |
| `play_store` | string | Play Store download link |

> **Note**: For bandwidth-constrained environments, use `qrString` to generate QR codes locally instead of using `qrImage`.

```json
{
  "status": {
    "code": "0",
    "message": "Success",
    "trace_id": "abc-123-def"
  },
  "amount": 1.00,
  "currency": "USD",
  "qrString": "00020101021229...",
  "qrImage": "data:image/png;base64,...",
  "abapay_deeplink": "abamobilebank://ababank.com?type=payway&qrcode=...",
  "app_store": "https://apps.apple.com/kh/app/aba-mobile/id...",
  "play_store": "https://play.google.com/store/apps/details?id=..."
}
```

## Callback Notification (POST to `callback_url`)

| Field | Type | Max Length | Description |
|-------|------|-----------|-------------|
| `tran_id` | string | 20 | Payment gateway transaction ID |
| `apv` | integer | 6 | Transaction approval code |
| `status` | string | — | Request status `"00"` |
| `merchant_ref_no` | string | — | Your payment link reference |

## Status Codes

| Code | Description |
|------|-------------|
| `0` | Success |
| `1` | Wrong hash |
| `6` | Requested domain is not in whitelist |
| `8` | Something went wrong |
| `12` | Payment currency is not allowed |
| `16` | Invalid first name |
| `17` | Invalid last name |
| `18` | Invalid phone number |
| `19` | Invalid email |
| `21` | End of API lifetime |
| `23` | Selected payment option not enabled |
| `32` | Service is not enabled |
| `35` | Payout info is invalid |
| `44` | Purchase amount reached transaction limit |
| `47` | KHR amount must be greater than 100 KHR |
| `48` | Something went wrong with requested parameters |
| `96` | Invalid merchant data |
| `102` | The URL is not in the whitelist |
| `403` | Duplicated transaction ID |
| `429` | Maximum attempt limit reached |
