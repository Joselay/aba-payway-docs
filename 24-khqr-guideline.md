# KHQR Guideline

KHQR is a standardized QR code payment system overseen by the National Bank of Cambodia that enables seamless digital transactions across the country's financial ecosystem.

## Compliance Requirement

Merchants implementing KHQR are required to make adjustments or changes upon request by ABA or in compliance with the National Bank of Cambodia's requirements. Failure to comply may result in service suspension or termination.

## Benefits

### Customer Benefits

- Single KHQR code eliminates multiple QR confusion at checkout
- Look for the KHQR label when paying
- Use preferred payment app including Bakong App at any KHQR location

### Merchant Benefits

- Save counter space with one KHQR stand
- Simple, fast, secure solution
- Accept payments from any bank app without bilateral contracts

## QR Code Specification

### Notation Convention

| Abbreviation | Description |
|---|---|
| ans | Alphanumeric Special (96 characters including numerics and punctuation) |
| C | Conditional |
| CDCVM | Consumer Device Cardholder Verification Method |
| CRC | Cyclic Redundancy Check |
| ECI | Extended Channel Interpretation |
| ID | Identifier of the data object |
| ISO | International Standards Organization |
| M | Mandatory |
| N | Numeric (digits "0" to "9") |
| QR Code | Quick Response Code |
| RFU | Reserved for Future Use |
| S | String (Unicode characters) |
| Var. | Variable |

### Data Objects Under Root QR Code

| Name | ID | Format | Length | Presence | Comment |
|---|---|---|---|---|---|
| Payload Format Indicator | 00 | N | 02 | M | 01 |
| Point of Initial Method | 01 | N | 02 | M | 11=Static QR (no amount); 12=Dynamic QR (with amount) |
| Merchant Account Information | 30 | N | var. up to 99 | M | Provided by ABA |
| Merchant Category Code | 52 | ans | var. up to 99 | M | At least one Merchant Account Information present |
| Transaction Currency | 53 | N | 03 | M | 116=KHR; 840=USD |
| Transaction Amount | 54 | ans | var. up to 13 | C | No decimal places for KHR amounts |
| Country Code | 58 | ans | 02 | M | — |
| Merchant Name | 59 | ans | var. up to 25 | M | Display name when scanned; must match ABA registered profile |
| Merchant City | 60 | ans | var. up to 15 | M | Valid values: Battambang, BMC (Banteay MeanChey), Kampong Cham, Kampong Chhnang, Kampong Speu, Kampong Thom, Kandal, Kep, Koh Kong, Kratie, Mondolkiri, Oddor Meanchey, Pailin, Pady Paet, Phnom Penh, Preah Vihear, Prey Veng, Pursat, Ratanakiri, Siem Reap, Sihanouk Ville, Steung Treng, Svay Rieng, Takeo, Tboung Khmum |
| Additional Data Field Template | 62 | S | var. up to 99 | M | Information provided by merchant |
| Additional Data Field | 99 | S | var. up to 99 | M | Additional info used by Bakong |
| CRC | 63 | ans | 04 | M | Cyclic Redundancy Check |

### Data Objects for Additional Data Field Template (ID "62")

| Name | ID | Format | Length | Presence | Comment |
|---|---|---|---|---|---|
| Merchant Reference Number | 01 | ans | var. up to 25 | M | — |
| PayWay Data Field Template | 68 | S | var. up to 99 | M | Provided by ABA |

### Multi-Payment

QR can be paid multiple times.

## QR Code Example

```
00020101021230510016abaakhppxxx@abaa01151250212145328460208ABA Bank52045987530311654031005802KH5925OLD ME 25 CHAR WINNER IP26010Phnom Penh62570115MC-REF-KH-1500068340010PAYWAY@ABA0208104514230604A2279934001317598053453370113175980552533763049FBD
```

### Example Breakdown

| ID | Sub Tag | Length | Value | Description |
|---|---|---|---|---|
| 00 | — | 02 | 01 | Version 1 |
| 01 | — | 02 | 12 | Dynamic QR indicator |
| 30 | — | 51 | — | Merchant info |
| — | 00 | 16 | abaakhppxxx@abaa | Acquiring Bakong ID |
| — | 01 | 15 | 125021214532846 | MID provided by ABA Bank |
| — | 02 | 08 | ABA Bank | Acquiring Bank name |
| 52 | — | 04 | 5987 | Merchant Category Code |
| 53 | — | 03 | 116 | KHR transaction currency |
| 54 | — | 03 | 100 | Transaction Amount |
| 58 | — | 02 | KH | Country Code |
| 59 | — | 25 | OLD ME 25 CHAR WINNER IP2 | Merchant Name |
| 60 | — | 10 | PHNOM PENH | Merchant City |
| 62 | — | 57 | — | Additional data |
| — | 01 | 15 | MC-REF-KH-15000 | Merchant reference # |
| — | 68 | 34 | 0010PAYWAY@ABA0208104514230604A227 | Provided by ABA Bank |
| 99 | — | 34 | — | Additional data for Bakong |
| — | 00 | 13 | 1759805345337 | Creation timestamp |
| — | 01 | 13 | 1759805345337 | Expiry timestamp |
| 63 | — | 04 | 9FBD | CRC |

## Webhook Payment Notifications

Merchants must provide webhook URL to PayWay for payment notifications.

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
|---|---|
| `transaction_id` | Unique transaction ID generated by PayWay |
| `merchant_ref` | Information from QR subtag 62.01 |
| `transaction_date` | Date and time of transaction |
| `bank_ref` | Core banking booking entry |
| `payment_status_code` | `0` represents successful payment |
| `payment_status` | Status description in word format |
| `apv` | Approval code (6 digits) |
| `original_amount` | Amount merchant received |
| `original_currency` | Merchant currency |
| `payment_amount` | Payer payment amount |
| `payment_currency` | Payer currency (`KHR` or `USD`) |
| `payment_type` | `ABA PAY` or `KHQR` |
| `payer_account` | Masked account number of payer |
| `bank_name` | Issuer bank name |
