# Add Beneficiary to Whitelist API

Registers a beneficiary (ABA Account holder or ABA Merchant) for payouts. Beneficiaries must be whitelisted before they can receive funds.

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
| `merchant_id` | string | 20 | Yes | Unique merchant key from ABA Bank |
| `merchant_auth` | string | — | Yes | RSA-encrypted, Base64-encoded JSON object |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash of `request_time` + `merchant_auth` |

### `merchant_auth` Object (before encryption)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mc_id` | string | Yes | Same as `merchant_id` |
| `payee` | string | Yes | Beneficiary identifier: MID or ABA account number |

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
| `name` | string | 255 | Beneficiary name (outlet name for MID, account holder name for account) |
| `payee` | string | 250 | MID or ABA Account number |
| `currency` | string | 10 | Merchant currency (MID) or account currency |
| `type` | string | 20 | `Merchant` or `ABA Account` |
| `status` | integer | — | `1` = Active, `0` = Inactive |
| `created_at` | string | — | Date and time added to whitelist |

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
