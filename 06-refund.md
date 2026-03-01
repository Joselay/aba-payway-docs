# Refund API

Processes full or partial refunds for completed transactions.

## Endpoint

```
POST /api/merchant-portal/merchant-access/online-transaction/refund
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

**Rate Limit**: 500 requests/second

## Rules

- Only transactions with status `COMPLETED` can be refunded
- Refunds must be requested within **30 days** of the payment creation date
- Refunds can be issued even if settlement is still pending (Alipay, WeChat, Card)
- Multiple partial refunds allowed until the total paid amount is fully refunded
- ABA PAY and KHQR refunds are immediate; Card, WeChat, and Alipay refunds follow your agreement with PayWay
- Works for both in-store and online transactions

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `request_time` | string | — | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | Unique merchant key provided by ABA Bank |
| `merchant_auth` | string | — | Yes | RSA-encrypted, Base64-encoded JSON object (see below) |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash of `request_time` + `merchant_id` + `merchant_auth` |

### `merchant_auth` Object (before encryption)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mc_id` | string | Yes | Same value as `merchant_id` |
| `tran_id` | string | Yes | Purchase transaction ID to refund |
| `refund_amount` | decimal | Yes | Amount to refund back to payer |

### RSA Encryption (PHP)

```php
$data_object = json_encode([
    'mc_id' => $merchant_id,
    'tran_id' => $tran_id,
    'refund_amount' => $refund_amount
]);
$rsa_public_key = "RSA PUBLIC KEY PROVIDED BY ABA BANK";
$maxlength = 117;
$encrypted_output = '';
while ($data_object !== '') {
    $chunk = substr($data_object, 0, $maxlength);
    $data_object = substr($data_object, $maxlength);
    openssl_public_encrypt($chunk, $encrypted_chunk, $rsa_public_key);
    $encrypted_output .= $encrypted_chunk;
}
$merchant_auth = base64_encode($encrypted_output);
```

### Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $request_time . $merchant_id . $merchant_auth;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

| Field | Type | Description |
|-------|------|-------------|
| `grand_total` | number | Final purchase amount after discounts |
| `total_refunded` | number | Cumulative total of all refunds |
| `currency` | string | Original currency of the transaction |
| `transaction_status` | string | `REFUNDED` (for both full and partial refunds) |

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success |
| `PTL02` | Invalid hash |
| `PTL04` | Parameter validation required |
| `PTL05` | Parameter invalid format |
| `PTL06` | Missing or incorrectly formatted `request_time` |
| `PTL37` | Refund amount cannot exceed original purchase amount |
| `PTL57` | Unable to refund |
| `PTL58` | Fail to refund |
| `PTL62` | Invalid merchant information |
| `PTL63` | Merchant has no security config file |
| `PTL168` | Concurrent requests not allowed. Try again in a few seconds |
| `PTL169` | Settlement account is closed |
| `PTL181` | Available balance not enough to refund |
| `PTL186` | Invalid amount format |
| `PTL187` | Amount is below the minimum allowed |
