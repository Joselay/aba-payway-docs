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
| `4` | Duplicated transaction ID |
| `11` | Something went wrong |
| `24` | Cannot decrypt data |
| `25` | Maximum 10 beneficiaries per request |
| `26` | Invalid merchant profile |
| `36` | Payout account or amount is invalid |
| `37` | Payout accounts are not in whitelist |
| `44` | Purchase amount has reached transaction limit |
| `48` | Something went wrong with requested parameters |
| `70` | Total purchase amount reached daily limit |
| `79` | Payment rejected |
| `80` | Custom fields invalid |
| `81` | Total amount must be greater than 0 |
| `82` | Invalid transaction currency (only USD or KHR) |
| `83` | Transaction is duplicated |
| `84` | Unable to access merchant's account details |
| `85` | Transaction currency doesn't match merchant's currency |
| `86` | Unable to debit merchant's account |
| `87` | Unable to retrieve beneficiary's account details |
| `88` | Unable to retrieve beneficiary's MID details |
| `89` | Unable to retrieve beneficiary's account details |
| `90` | Merchant and beneficiary currencies don't align |
| `91` | Unable to credit beneficiary's account |
| `92` | Total payout amount doesn't match total transaction amount |
| `93` | Insufficient balance |
| `400` | Bad request |
| `LAM01` | Daily limit reached |
| `LAM02` | Monthly limit reached |
