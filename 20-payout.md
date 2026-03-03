# Payout API

Distributes funds from a merchant's settlement account to beneficiaries (ABA Account holders or ABA Merchants).

## Endpoint

```
POST /api/payment-gateway/v2/direct-payment/merchant/payout
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Prerequisites

Beneficiaries must be whitelisted before payouts can be made. Use the [Add Beneficiary to Whitelist](./21-add-beneficiary.md) API.

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `merchant_id` | string | 255 | Yes | Unique merchant key from ABA Bank |
| `tran_id` | string | 20 | Yes | Unique transaction ID |
| `beneficiaries` | string | 1000 | Yes | RSA-encrypted, Base64-encoded JSON array of beneficiary objects |
| `amount` | number (float) | — | Yes | Total payout amount (sum of all beneficiary amounts). KHR >= 100, USD >= 0.01 |
| `currency` | string | 3 | Yes | `KHR` or `USD` |
| `custom_fields` | string | 255 | No | Additional JSON information associated with the transaction |
| `hash` | string | 512 | Yes | Base64-encoded HMAC-SHA512 hash |

### Beneficiaries Array (before encryption)

```json
[
  {"account": "200030000", "amount": 100},
  {"account": "012538302", "amount": 200}
]
```

> Maximum 10 beneficiaries per request.

### RSA Encryption (PHP)

```php
$data_object = json_encode([
    ['account' => '200030000', 'amount' => 100],
    ['account' => '012538302', 'amount' => 200]
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
$beneficiaries = base64_encode($encrypted_output);
```

### Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $merchant_id . $tran_id . $beneficiaries . $amount . $custom_fields . $currency;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

| Field | Type | Description |
|-------|------|-------------|
| `transaction_id` | string | Your `tran_id` |
| `transaction_date` | string | Approved date and time |
| `external_reference` | string | Unique reference from core banking |
| `apv` | string | 6-digit approval code |
| `transaction_amount` | number | Total transaction amount |
| `transaction_currency` | string | Transaction currency |
| `beneficiaries` | array | Array of beneficiary results |
| `status` | object | Status information |

### Beneficiary Object

| Field | Type | Description |
|-------|------|-------------|
| `payout_id` | string | Unique ID from payment gateway |
| `name` | string | Beneficiary name |
| `mid_acccount` | string | MID or ABA account number |
| `amount` | number | Payout amount |
| `currency` | string | Follows transaction currency |

### Status Object

| Field | Type | Description |
|-------|------|-------------|
| `code` | string | Response code |
| `message` | string | Associated message |
| `tran_id` | string | Your transaction ID |
| `trace_id` | string | Trace ID for debugging |

## Status Codes

| Code | Description |
|------|-------------|
| `0` | Success |
| `4` | Duplicated Transaction ID |
| `11` | Something went wrong. Try again or contact the merchant for help |
| `24` | Can not decrypt data |
| `25` | Allow maximum 10 beneficiaries per requests |
| `26` | Invalid Merchant Profile |
| `36` | Payout account or amount is invalid |
| `37` | Payout accounts are not in whitelist |
| `44` | Purchase amount has reached transaction limit |
| `48` | Something went wrong with requested parameters. Please try again or contact the merchant for help |
| `70` | Total purchase amount has reached daily limit. Please use difference account |
| `79` | Payment Rejected! |
| `80` | Custom fields invalid |
| `81` | The total amount must be greater than 0. Please double check and try again. |
| `82` | Invalid transaction currency. We only support USD or KHR. Please double check and try again. |
| `83` | Transaction is duplicated. |
| `84` | Unable to access the merchant's account details. Please verify that your settlement account is still active. |
| `85` | Transaction currency does not match the merchant's currency. Please review your details. |
| `86` | Unable to debit the merchant's account. |
| `87` | Unable to retrieve the beneficiary's account details. |
| `88` | Unable to retrieve the beneficiary's MID details. |
| `89` | Unable to retrieve the beneficiary's account details. |
| `90` | The currencies for the merchant and beneficiary do not align. Please review your details. |
| `91` | Unable to credit the beneficiary's account. |
| `92` | The total payout amount does not match the total transaction amount. Please review your details. |
| `93` | Insufficient balance. |
| `400` | Bad request |
| `LAM01` | Total purchase amount has reached daily limit. Please use difference account |
| `LAM02` | Total purchase amount has reached monthly limit. Please use difference account |
