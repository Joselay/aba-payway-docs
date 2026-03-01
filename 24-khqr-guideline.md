# KHQR Guideline

KHQR is a standardized QR code payment system overseen by the National Bank of Cambodia that enables seamless digital transactions across the country's financial ecosystem.

## Compliance Requirement

Merchants implementing KHQR must make adjustments upon request by ABA or in compliance with the National Bank of Cambodia's requirements. Failure to comply may result in service suspension or termination.

## QR Code Specification

### Notation Convention

| Abbreviation | Meaning |
|-------------|---------|
| ans | Alphanumeric Special |
| C | Conditional |
| M | Mandatory |
| N | Numeric |
| S | String |

### Root Level Data Objects

| Data Element | ID | Format | Length | Required |
|-------------|-----|--------|--------|----------|
| Payload Format Indicator | 00 | N | 02 | M |
| Point of Initiation Method | 01 | N | 02 | M |
| Merchant Account Information | 30 | N | var. up to 99 | M |
| Merchant Category Code | 52 | ans | var. up to 99 | M |
| Transaction Currency | 53 | N | 03 | M |
| Transaction Amount | 54 | ans | var. up to 13 | C |
| Country Code | 58 | ans | 02 | M |
| Merchant Name | 59 | ans | var. up to 25 | M |
| Merchant City | 60 | ans | var. up to 15 | M |
| Additional Data Field Template | 62 | S | var. up to 99 | M |
| Additional Data Field | 99 | S | var. up to 99 | M |
| CRC | 63 | ans | 04 | M |

### Point of Initiation Method

| Value | Type |
|-------|------|
| `11` | Static QR (without amount) |
| `12` | Dynamic QR (with amount) |

### Currency Codes

| Code | Currency |
|------|----------|
| `116` | KHR (Cambodian Riel) |
| `840` | USD (US Dollar) |

### Merchant City Valid Values

Battambang, BMC (Banteay MeanChey), Kampong Cham, Kampong Chhnang, Kampong Speu, Kampong Thom, Kandal, Kep, Koh Kong, Kratie, Mondolkiri, Oddor Meanchey, Pailin, Pady Paet, Phnom Penh, Preah Vihear, Prey Veng, Pursat, Ratanakiri, Siem Reap, Sihanouk Ville, Steung Treng, Svay Rieng, Takeo, Tboung Khmum.

### Merchant Name

The display name shown when a mobile banking app scans the QR. Must match the name on your ABA registered profile.

### Additional Data Field Template Objects

| Name | ID | Format | Length | Required |
|------|-----|--------|--------|----------|
| Merchant Reference Number | 01 | ans | var. up to 25 | M |
| PayWay Data Field Template | 68 | S | var. up to 99 | M |

### Multi-Payment

QR codes can be paid multiple times, allowing single QR codes to process repeated transactions.

## QR Code Example

```
00020101021230510016abaakhppxxx@abaa01151250212145328460208ABA Bank52045987530311654031005802KH5925OLD ME 25 CHAR WINNER IP26010Phnom Penh62570115MC-REF-KH-1500068340010PAYWAY@ABA0208104514230604A2279934001317598053453370113175980552533763049FBD
```

### Example Breakdown

| ID | Sub Tag | Length | Value | Description |
|----|---------|--------|-------|-------------|
| 00 | — | 02 | 01 | Version 1 |
| 01 | — | 02 | 12 | Dynamic QR |
| 30 | — | 51 | — | Merchant information |
| — | 00 | 16 | abaakhppxxx@abaa | Acquiring Bakong ID |
| — | 01 | 15 | 125021214532846 | Merchant ID from ABA Bank |
| — | 02 | 08 | ABA Bank | Acquiring bank name |
| 52 | — | 04 | 5987 | Merchant Category Code |
| 53 | — | 03 | 116 | KHR currency |
| 54 | — | 03 | 100 | Transaction amount |
| 58 | — | 02 | KH | Country code |
| 59 | — | 25 | OLD ME 25 CHAR WINNER IP2 | Merchant name |
| 60 | — | 10 | PHNOM PENH | Merchant city |
| 62 | — | 57 | — | Additional data |
| — | 01 | 15 | MC-REF-KH-15000 | Merchant reference number |
| — | 68 | 34 | 0010PAYWAY@ABA0208104514230604A227 | PayWay data |
| 99 | — | 34 | — | Bakong-specific data |
| — | 00 | 13 | 1759805345337 | Creation timestamp |
| — | 01 | 13 | 1759805345337 | Expiry timestamp |
| 63 | — | 04 | 9FBD | CRC checksum |

## Webhook Payment Notifications

Merchants must supply webhook URLs to PayWay for receiving payment success notifications.

### Requirements

- **Method**: `POST`
- **Protocol**: HTTPS required

### Webhook Payload

```json
{
   "transaction_id": "3309DCD5BCB94CBD820046CE9",
   "transaction_date": "2025-10-10 16:03:26",
   "original_currency": "KHR",
   "original_amount": 100,
   "bank_ref": "100FT30153179430",
   "apv": "341136",
   "payment_status_code": 0,
   "payment_status": "APPROVED",
   "payment_currency": "KHR",
   "payment_amount": 100.0,
   "payment_type": "ABA Pay",
   "payer_account": "*898",
   "bank_name": "ABA Bank",
   "merchant_ref": "3309DCD5BCB94CBD820046CE9"
}
```

### Webhook Fields

| Field | Description |
|-------|-------------|
| `transaction_id` | Unique identifier from PayWay |
| `merchant_ref` | Data from QR subtag 62.01 |
| `transaction_date` | Transaction date and time |
| `bank_ref` | Core banking booking entry reference |
| `payment_status_code` | `0` = successful payment |
| `payment_status` | Status description (e.g., `APPROVED`) |
| `apv` | 6-digit approval code |
| `original_amount` | Amount received by merchant |
| `original_currency` | Merchant's currency |
| `payment_amount` | Amount paid by customer |
| `payment_currency` | Payer's currency (`KHR` or `USD`) |
| `payment_type` | `ABA PAY` or `KHQR` |
| `payer_account` | Masked payer account number |
| `bank_name` | Issuing bank name |
