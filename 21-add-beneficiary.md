# Add a beneficiary to whitelist

Use this API to whitelist accounts that you intend to split payment and payout. You'll first have to whitelist the accounts before you can use those accounts to request on payout request.

## Endpoint

```
POST /api/merchant-portal/merchant-access/whitelist-account/add-whitelist-payout
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `request_time` | string | — | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | A unique merchant key which provided by ABA Bank |
| `merchant_auth` | string | — | Yes | The JSON-encoded object containing `mc_id`, `payee` encrypted using RSA public key encryption in chunks |
| `hash` | string | — | Yes | Base64 encode of hash hmac sha512 encryption of concatenates values `request_time` and `merchant_auth` with `public_key` |

### `merchant_auth` Object (before encryption)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mc_id` | string | Yes | A unique merchant key which provided by ABA Bank. Same value as `merchant_id` |
| `payee` | string | Yes | Beneficiary identifier: It can be either a MID or an ABA account |

### RSA Encryption

The `merchant_auth` JSON object must be encrypted using the RSA public key provided by ABA Bank. Data is encrypted in 117-byte chunks, then concatenated and Base64-encoded.

## Example Request

```json
{
  "request_time": "20200728093403",
  "merchant_id": "ec000002",
  "merchant_auth": "39aaa43...c00a",
  "hash": "EVDFA21....t+sWw=="
}
```

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $request_time . $merchant_auth;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

### `data` Object

| Field | Type | Max Length | Description |
|-------|------|-----------|-------------|
| `name` | string | 255 | The name of the beneficiary: if the type is MID, it will be the outlet name; if it is an account, it will be the account holder's name |
| `payee` | string | 250 | This value represent the destination beneficiary it can be MID or ABA Account number |
| `currency` | string | 10 | If payee is MID, the value here will be merchant's currency and if the payee is an ABA Account holder it will return account currency |
| `type` | string | 20 | If payee is MID, the value here is `Merchant` if the payee is an ABA Account holder it will return `ABA Account` |
| `status` | integer | — | The current status of the beneficiary. `1` = Active, `0` = Inactive |
| `created_at` | string | — | Date and time that the beneficiary was created or added to the list |

### Example Response

```json
{
  "data": {
    "name": "Sample Outlet",
    "payee": "318111358120004",
    "currency": "USD",
    "type": "Merchant",
    "status": 1,
    "created_at": "2020-07-28T09:34:03Z"
  },
  "status": {
    "code": "200",
    "message": "Success"
  }
}
```

## Status Codes

| Code | Description |
|------|-------------|
| `PTL02` | Wrong hash |
| `PTL04` | Parameter validation required |
| `PTL25` | Invalid account class |
| `PTL99` | Merchant invalid currency |
| `PTL134` | Account not found |
| `PTL146` | Payee is invalid |
| `PTL147` | Currency of the payee does not the same as merchant currency |
| `PTL148` | Payee already exist |
| `PTL150` | Business profile is not found |
| `PTL151` | Failed to whitelist account |
