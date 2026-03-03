# Refund API

You can use the Refund API to issue full or partial refunds within 30 days after the transaction was created. ABA PAY and KHQR refunds are immediate, while Card, WeChat, and Alipay refunds follow your agreement with PayWay. This API works both for instore transaction and online transaction.

## Endpoint

```
POST /api/merchant-portal/merchant-access/online-transaction/refund
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

**Rate Limit**: Request limit 500 requests/second

## Constraints

- **Eligible Transactions**: Only transactions with a status of COMPLETED can be refunded.
- **Time Frame**: Refunds must be requested within 30 days of the payment created date.
- **Pending Settlements**: Refunds can be issued even if the settlement is still pending (Alipay, WeChat, Card).
- **Partial Refunds**: Multiple partial refunds can be issued until the total amount paid is refunded.

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `request_time` | string | — | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | A unique merchant key which provided by ABA Bank |
| `merchant_auth` | string | — | Yes | The JSON-encoded object containing `mc_id`, `tran_id`, and `refund_amount` using RSA public key encryption in chunks. The encrypted data is then concatenated and encoded in Base64 format |
| `hash` | string | — | Yes | Base64 encode of hash hmac sha512 encryption of concatenates values `request_time`, `merchant_id` and `merchant_auth` with `public_key` |

### `merchant_auth` Object (before encryption)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mc_id` | string | Yes | A unique merchant key which provided by ABA Bank. Same value as `merchant_id` |
| `tran_id` | string | Yes | Purchase transaction id to refund |
| `refund_amount` | decimal | Yes | Amount to refund back to payer |

### RSA Encryption (PHP)

```php
$data_object = json_encode([
    'mc_id' => $merchant_id,
    'tran_id' => $tran_id,
    'refund_amount' => $amount
]);
$rsa_public_key = "RSA PUBLIC KEY PROVIDED BY ABA BANK";
$maxlength = 117;
$encrypted_output = '';
while ($data_object !== '') {
    $chunk = substr($data_object, 0, $maxlength);
    $data_object = substr($data_object, $maxlength);
    if (openssl_public_encrypt($chunk, $encrypted_chunk, $rsa_public_key)) {
        $encrypted_output .= $encrypted_chunk;
    } else {
        throw new Exception('Encryption failed for a data chunk.');
    }
}
$merchant_auth = base64_encode($encrypted_output);
```

### Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $request_time . $merchant_id . $merchant_auth;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

### Request Example

```json
{
  "request_time": "20200728093403",
  "merchant_id": "ec000002",
  "merchant_auth": "884113079983a...2c3e460be35f2a3",
  "hash": "3nd/2Z4g45...wnA2WA/M/Qg=="
}
```

## Response

**HTTP 200**

Success response:

```json
{
    "grand_total": 1.5,
    "total_refunded": 0.09,
    "currency": "USD",
    "transaction_status": "REFUNDED",
    "status": {
        "code": "00",
        "message": "Success!"
    }
}
```

Exception response:

```json
{
    "status": {
        "code": "PTL02",
        "message": "Wrong Hash"
    }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `grand_total` | number | Example: A customer pays 20USD by card. A 2USD discount is applied, so the final amount is 18USD |
| `total_refunded` | number | Total of refunded amount. If the purchase transactions has refunded 3 times, and each time was 1USD, to total_refunded = 3USD |
| `currency` | string | Original currency of the transaction |
| `transaction_status` | string | Either it's full refund or partial refund the status here is `REFUNDED` |
| `status.message` | string | Please see the property response `code` for the details |

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success! |
| `PTL02` | Wrong Hash |
| `PTL04` | Parameter validation required |
| `PTL05` | Parameter invalid format |
| `PTL06` | The `request_time` value is missing or incorrectly formatted |
| `PTL37` | Refund amount cannot exceed the original purchase amount |
| `PTL57` | Unable to refund |
| `PTL58` | Fail to refund |
| `PTL62` | Invalid merchant information |
| `PTL63` | Merchant have no security config file |
| `PTL168` | Concurrent requests are not allowed for this operation. Please try again in a few seconds |
| `PTL169` | The merchant profile cannot accept payment because its settlement account is closed |
| `PTL181` | The available balance is not enough to refund the customer |
| `PTL186` | Invalid amount format |
| `PTL187` | Amount is below the minimum allowed |
