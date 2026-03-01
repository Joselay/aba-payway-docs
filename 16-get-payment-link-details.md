# Get Payment Link Details API

Retrieves details of a specific payment link.

## Endpoint

```
POST /api/merchant-portal/merchant-access/payment-link/detail
```

**Base URL**: `https://checkout-sandbox.payway.com.kh/` (sandbox) | `https://checkout.payway.com.kh/` (production)

**Content-Type**: `application/json`

## Request Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `request_time` | string | Yes | Request date and time in UTC format as `YYYYMMDDHHmmss` |
| `merchant_id` | string | Yes | Unique merchant key from ABA Bank |
| `merchant_auth` | string | Yes | RSA-encrypted, Base64-encoded JSON object containing `mc_id` and `id` |
| `hash` | string | Yes | Base64-encoded HMAC-SHA512 hash of `request_time` + `merchant_id` + `merchant_auth` |

### `merchant_auth` Object (before encryption)

```json
{
  "mc_id": "merchant_id_value",
  "id": "payment_link_id"
}
```

## Response

**HTTP 200**

### `data` Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique payment link ID |
| `title` | string | Payment link title |
| `image` | object | `{ image: string, filename: string, size: number }` |
| `amount` | number | Payment link amount |
| `currency` | string | `KHR` or `USD` |
| `status` | string | `OPEN` (can still be paid) or `PAID` (limit reached) |
| `payment_limit` | number | Maximum transactions allowed |
| `total_amount` | number | Total amount after refund |
| `total_trxn` | number | Completed payment transactions |
| `created_at` | string | Creation timestamp |
| `updated_at` | string | Last update timestamp |
| `expired_date` | number | Expiration timestamp |
| `pushback_url` | string | URL for payment status updates |
| `payment_link` | string | Full payment link URL |
| `description` | string | Payment link description |
| `total_amount_org` | string | Total before refund |
| `total_refund` | string | Total refund amount |

## Status Codes

| Code | Description |
|------|-------------|
| `PTL02` | Wrong hash |
| `PTL132` | Invalid payment link |
