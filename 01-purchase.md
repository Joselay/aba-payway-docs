# Purchase API

Creates a new payment transaction.

## Endpoint

```
POST /api/payment-gateway/v1/payments/purchase
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `multipart/form-data`

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `req_time` | string | — | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | 30 | Yes | Unique merchant key provided by ABA Bank |
| `tran_id` | string | 20 | Yes | Unique transaction identifier for the payment |
| `amount` | number | — | Yes | Purchase amount |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash |
| `firstname` | string | 100 | No | Buyer's first name |
| `lastname` | string | 100 | No | Buyer's last name |
| `email` | string | 50 | No | Buyer's email |
| `phone` | string | 20 | No | Buyer's phone |
| `type` | string | 20 | No | Type of the transaction, default value is `purchase`. Supported values: `pre-auth` (for pre purchase), `purchase` (for full purchase) |
| `payment_option` | string | 20 | No | One of: `cards`, `abapay_khqr`, `abapay_khqr_deeplink`, `alipay`, `wechat`, `google_pay` |
| `items` | string | 500 | No | A base64-encoded JSON array describing the items being purchased |
| `shipping` | number | — | No | Shipping fee |
| `currency` | string | — | No | `KHR` or `USD`. Defaults to merchant profile setting |
| `return_url` | string | — | No | URL to receive callbacks upon payment completion, Base64-encoded |
| `cancel_url` | string | — | No | The URL to redirect to after the user closes the payment dialog or when user cancel the payment |
| `skip_success_page` | integer | — | No | `0` (don't skip) or `1` (skip success page). Defaults to profile config |
| `continue_success_url` | string | — | No | URL to redirect after successful payment |
| `return_deeplink` | string | — | No | The deep link for redirecting to the app after a successful payment from ABA Mobile. Must be base64-encoded and include both iOS and Android schemes |
| `custom_fields` | string | — | No | Additional information that you want to attach to the transaction. This information will appear in the transaction list, transaction details and export report |
| `return_params` | string | — | No | Information included when PayWay calls your return URL |
| `view_type` | string | — | No | `hosted_view` or `popup` |
| `payment_gate` | integer | — | No | Set to `0` for Checkout service |
| `payout` | string | — | No | Base64-encoded JSON string with payout details |
| `additional_params` | string | — | No | Currently supports WeChat Mini Program. Supported value keys: `wechat sub_appid` and `wechat_sub_openid` |
| `lifetime` | integer | — | No | Payment lifetime in minutes (min: 3, max: 43200 for 30 days). Payments after exceeding lifetime are rejected |
| `google_pay_token` | string | — | No | Required if `payment_option` is `google_pay` |

### Items Example

```php
$item = base64_encode(json_encode([
    ["name" => "product 1","quantity" => 1,"price" => 1.00],
    ["name" => "product 2","quantity" => 2, "price" => 4.00]
]));
```

### Return Deeplink Example

```php
$return_deeplink = base64_encode(json_encode([
    "ios_scheme" => "DEEPLINK TO RETURN TO YOUR IOS APP",
    "android_scheme" => "DEEPLINK TO RETURN TO YOUR ANDROID APP"
]));
```

### Custom Fields Example

```php
$custom_field = base64_encode(json_encode([
    "field1" => "myvalue1",
    "field2" => "myvalue2"
]));
```

### Payout Example

```php
$payout = base64_encode(json_encode([
    ["acc" => "000133879","amt"=> 1],
    ["acc" => "000133880","amt" => 1]
]));
```

### Additional Params Example

```php
$additional_params = base64_encode(json_encode([
    'wechat_sub_appid' => 'YOUR WECHAT APP ID',
    'wechat_sub_openid' => 'YOUR WECHAT OPEN ID'
]));
```

## Hash Generation

Concatenate all parameter values then HMAC-SHA512 with API key and Base64-encode.

> **Note:** The hash field order differs from the parameter table order above. The explicit concatenation order from the remote docs is:
>
> `req_time, merchant_id, tran_id, amount, items, shipping, firstname, lastname, email, phone, type, payment_option, return_url, cancel_url, continue_success_url, return_deeplink, currency, custom_fields, return_params, payout, lifetime, additional_params, google_pay_token, skip_success_page`
>
> Note: `view_type` and `payment_gate` are **not** included in the hash.

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $req_time . $merchant_id . $tran_id . $amount . $items . $shipping . $firstname . $lastname . $email . $phone . $type . $payment_option . $return_url . $cancel_url . $continue_success_url . $return_deeplink . $currency . $custom_fields . $return_params . $payout . $lifetime . $additional_params . $google_pay_token . $skip_success_page;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

```json
{
    "status": {
        "code": "00",
        "message": "Success!",
        "tran_id": "trx-20201019130949"
    },
    "qr_string": "00020101021230510016abaakhppxxx@abaa01153250212100849350208ABA Bank520410165...",
    "abapay_deeplink": "abamobilebank://ababank.com?type=payway&qrcode=00020101021230510016...",
    "checkout_qr_url": "https://checkout-uat.payway.com.kh/eyJzdGF0dXMiOnsiY29kZSI6IjAwIiw..."
}
```

## Error Codes

| Code | Message |
|------|---------|
| `1` | Wrong hash |
| `2` | Invalid transaction ID |
| `3` | Invalid transaction amount |
| `4` | Duplicated transaction ID |
| `5` | Transaction not found |
| `6` | Requested domain is not in whitelist |
| `7` | Wrong return param |
| `8` | Something went wrong while saving data |
| `10` | Wrong shipping price |
| `11` | Something went wrong |
| `12` | Payment currency is not allowed |
| `13` | Invalid items |
| `14` | Invalid credit multi acc |
| `15` | Invalid or missing channel values from smart merchant |
| `16` | Invalid first name |
| `17` | Invalid last name |
| `18` | Invalid phone number |
| `19` | Invalid email |
| `20` | Something went wrong. Please contact merchant |
| `21` | End of API lifetime |
| `22` | Pre-auth transaction is not enabled |
| `23` | Selected payment option is not enabled for this merchant profile |
| `24` | Cannot decrypt data |
| `25` | Allow maximum 10 payout per requests |
| `26` | Invalid merchant profile |
| `27` | Invalid ctid |
| `28` | Invalid pwt |
| `29` | Invalid pwt or ctid |
| `30` | Merchant is not enabled COF |
| `31` | Unsecure 3Ds page |
| `33` | Cannot identify cardOrigin |
| `34` | Exchange rate data is invalid |
| `35` | Payout info is invalid |
| `36` | Payout account or amount is invalid |
| `37` | Payout accounts are not in whitelist |
| `38` | Payout contain invalid transaction ID |
| `39` | Payout contain duplicated account |
| `40` | Payout contain duplicated transaction ID |
| `41` | Payout info contain mid not link with any merchant profile |
| `42` | Payout info contain account invalid status |
| `43` | Merchant profile's MID is missing |
| `44` | Purchase amount has reached transaction limit |
| `45` | Purchase with zero amount is not allowed |
| `46` | Purchase amount for KHR currency could not contain decimal place |
| `47` | KHR amount must be greater than 100 KHR |
| `48` | Something went wrong with requested parameters |
| `49` | Invalid start date |
| `50` | Invalid end date |
| `51` | Invalid date range |
| `52` | Maximum date range is allowed only 3 days |
| `53` | Invalid amount range |
| `54` | Transaction is expired |
| `55` | Unable to request QR from Wechat system |
| `56` | Unable to validate transaction with Wechat system |
| `57` | Unable to validate card source |
| `58` | Invalid card number |
| `59` | Payout info can not be fixed with MID and ABA account |
| `60` | Something went wrong with QR String |
| `61` | Something went wrong |
| `62` | QR is already in used |
| `63` | Transaction already exists in core banking |
| `64` | Payer's account is same as merchant profile's account |
| `65` | Merchant profile's MID not found in core banking |
| `66` | Something went wrong |
| `67` | QR on invoice not available for this merchant profile |
| `68` | Transaction is expired |
| `69` | Transaction lifetime cannot be less than 3 minutes |
| `70` | Total purchase amount has reached daily limit |
| `71` | Payout for card payment is not allowed to ABA account |
| `72` | Merchant profile cannot accept payment — settlement account closed |
| `73` | Invalid transaction status |
| `74` | Invalid tran_id or merchant_id |
| `75` | tran_id not found |
| `76` | Invalid additional parameters |
| `77` | Merchant transactions do not support transaction fees |
| `78` | Card payout transactions not compatible with discount program |
| `79` | Payment token missing in Google Pay |
| `80` | Failed to decrypt Google Pay payment token |
| `81` | The return URL is not in the whitelist |
| `82` | Payment has exceeded the maximum allowable amount per transaction |
| `83` | Payment credential is disabled |
| `84` | Payment credential is expired |
| `85` | Purchase reach limit amount per transaction |
| `86` | Unsupported merchant purchase mode |
| `87` | Payment credential is removed |
| `200` | Payment was canceled |
| `201` | Payment was declined |
| `401` | Unauthorized access |
| `403` | Something went wrong |
| `429` | Too many requests, please try again in 1 minute |
| `503` | System under maintenance |
