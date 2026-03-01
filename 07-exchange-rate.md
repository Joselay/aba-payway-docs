# Exchange Rate API

Fetches the latest exchange rates from ABA Bank. Rates match those on [ABA Bank Forex Exchange](https://www.ababank.com/en/forex-exchange).

## Endpoint

```
POST /api/payment-gateway/v1/exchange-rate
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Request Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `req_time` | string | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | Yes | Unique merchant key provided by ABA Bank |
| `hash` | string | Yes | Base64-encoded HMAC-SHA512 hash |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $req_time . $merchant_id;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Response

**HTTP 200**

### Exchange Rates Object

Each currency object contains `sell` and `buy` rates as strings.

### Supported Currencies

| Code | Currency |
|------|----------|
| `aud` | Australian Dollar |
| `sgd` | Singapore Dollar |
| `eur` | Euro |
| `gbp` | Pound Sterling |
| `myr` | Malaysian Ringgit |
| `thb` | Thai Baht |
| `hkd` | Hong Kong Dollar |
| `cny` | Chinese Yuan |
| `cad` | Canadian Dollar |
| `krw` | South Korean Won |
| `jpy` | Japanese Yen |
| `vnd` | Vietnamese Dong |

### Example Response Structure

```json
{
  "status": {
    "code": "00",
    "message": "Success"
  },
  "exchange_rates": {
    "aud": { "sell": "...", "buy": "..." },
    "sgd": { "sell": "...", "buy": "..." },
    "eur": { "sell": "...", "buy": "..." }
  }
}
```

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success |
| `1` | Wrong hash |
| `26` | Invalid merchant profile |
