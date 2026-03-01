# ABA PayWay API Documentation

Offline-ready reference documentation for the [ABA PayWay](https://developer.payway.com.kh) payment gateway API. Structured for SDK development across all languages.

## Contents

### Overview & Guides

- [00 - Overview](./00-overview.md) — API overview, authentication, environments, test cards
- [24 - KHQR Guideline](./24-khqr-guideline.md) — QR code specification & webhook notifications
- [25 - eCommerce Checkout Guide](./25-ecommerce-checkout-guide.md) — Integration steps for web & mobile
- [26 - Resources](./26-resources.md) — Test cards, checkout plugins

### Checkout APIs

- [01 - Purchase](./01-purchase.md) — Create a payment transaction
- [02 - Check Transaction](./02-check-transaction.md) — Get transaction status (last 7 days)
- [03 - Get Transaction Details](./03-get-transaction-details.md) — Full transaction details with history
- [04 - Get Transaction List](./04-get-transaction-list.md) — List transactions with filters & pagination
- [05 - Close Transaction](./05-close-transaction.md) — Cancel a pending transaction
- [06 - Refund](./06-refund.md) — Full or partial refund
- [07 - Exchange Rate](./07-exchange-rate.md) — ABA Bank exchange rates

### Credentials on File APIs

- [08 - Link Account](./08-link-account.md) — Link ABA account via QR/deeplink
- [09 - Link Card](./09-link-card.md) — Link credit/debit card
- [10 - Purchase Using Token](./10-purchase-using-token.md) — Charge stored credentials
- [11 - Remove Account Token](./11-remove-account-token.md) — Remove linked account
- [12 - Remove Card Token](./12-remove-card-token.md) — Remove linked card
- [13 - Get Linked Account Details](./13-get-linked-account-details.md) — Retrieve account token info

### QR & Payment Link APIs

- [14 - QR API](./14-qr-api.md) — Generate dynamic QR codes
- [15 - Create Payment Link](./15-create-payment-link.md) — Create shareable payment links
- [16 - Get Payment Link Details](./16-get-payment-link-details.md) — Retrieve payment link info

### Pre-Auth APIs

- [17 - Complete Pre-Auth](./17-complete-pre-auth.md) — Capture pre-authorized funds
- [18 - Complete Pre-Auth with Payout](./18-complete-pre-auth-with-payout.md) — Capture & distribute funds
- [19 - Cancel Pre-Auth](./19-cancel-pre-auth.md) — Release held funds

### Payout APIs

- [20 - Payout](./20-payout.md) — Distribute funds to beneficiaries
- [21 - Add Beneficiary](./21-add-beneficiary.md) — Whitelist a payout recipient
- [22 - Update Beneficiary Status](./22-update-beneficiary-status.md) — Activate/deactivate beneficiary
- [23 - Get Transactions by Ref](./23-get-transactions-by-ref.md) — Look up transactions by merchant reference

## Related SDKs

- [aba-payway](https://github.com/Joselay/aba-payway) — TypeScript SDK
