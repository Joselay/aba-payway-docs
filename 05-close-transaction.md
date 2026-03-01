# Close Transaction API

Cancels a pending transaction. Upon closure, payment status becomes `CANCELLED` and no payment notification (callback) will be sent to the merchant.

## Endpoint

```
POST /api/payment-gateway/v1/payments/close-transaction
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Use Cases

- Flash sales cancellation
- Hotel booking cancellation
- Ticket sales cancellation

## Request Parameters

| Field | Type | Max Length | Required | Description |
|-------|------|-----------|----------|-------------|
| `req_time` | string | — | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | 20 | Yes | Unique merchant key provided by ABA Bank |
| `tran_id` | string | 20 | Yes | Original purchase transaction ID to close/cancel |
| `hash` | string | — | Yes | Base64-encoded HMAC-SHA512 hash |

## Hash Generation

```php
$api_key = "API KEY PROVIDED BY ABA BANK";
$b4hash = $req_time . $merchant_id . $tran_id;
$hash = base64_encode(hash_hmac('sha512', $b4hash, $api_key, true));
```

## Example Request

```json
{
  "req_time": "20241022053608",
  "merchant_id": "lavacafe",
  "tran_id": "1729573626",
  "hash": "ln9Td4JiGPc...R6y2tmUoF2NERNaQ=="
}
```

## Response

**HTTP 200**

```json
{
  "status": {
    "code": "00",
    "message": "string",
    "tran_id": "string"
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `status.code` | string | Response code |
| `status.message` | string | Details matching the response code |
| `status.tran_id` | string | Purchase transaction ID that has been cancelled |

## Status Codes

| Code | Description |
|------|-------------|
| `00` | Success |
| `1` | Wrong hash |
| `5` | Transaction not found |
| `26` | Invalid merchant profile |
