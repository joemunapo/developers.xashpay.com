# Wallet Management

This document provides detailed information about wallet management in the XashPay API system.

## Overview

Every XashPay account has an associated wallet that holds funds for processing transactions and receives commission earnings. The wallet system allows you to track your balance, view transaction history, and manage commission earnings.

## Balance

Your wallet balance consists of two components:

1. **Available Balance**: Funds that are immediately available for transactions
2. **Commission Balance**: Earnings that are held until the next commission release date

### Checking Your Balance

You can check your current wallet balance using the [Get Wallet Balance](api-reference.md#get-wallet-balance) endpoint:

```http
GET /wallet
Authorization: Bearer YOUR_API_TOKEN
```

Response:

```json
{
  "success": true,
  "message": "Wallet retrieved successfully",
  "commission_release_date": "2025-05-01T00:00:00+00:00",
  "data": {
    "currency": "ZAR",
    "name": "South Africa Rand",
    "balance": "1000.00",
    "profit_on_hold": "50.00"
  }
}
```

## Transactions

All financial activities in your XashPay account are recorded as transactions in your wallet.

### Transaction Types

| Type | Description |
|------|-------------|
| deposit | Funds added to your wallet |
| withdrawal | Funds withdrawn from your wallet |
| payment | Payment for a service |
| commission | Commission earned from a transaction |
| voucher_redemption | Funds added from a voucher redemption |
| voucher_action | Service payment using voucher funds |
| refund | Refund of a previous transaction |

### Transaction Status

| Status | Description |
|--------|-------------|
| pending | Transaction has been initiated but not completed |
| success | Transaction has been successfully completed |
| failed | Transaction has failed |
| reversed | Transaction has been reversed |

### Viewing Transaction History

Transaction history can be viewed through the XashPay dashboard. The API endpoint for transaction history will be available in a future update.

## Commission

Commission is earned on every successful service transaction based on the commission rate for that service.

### Commission Calculation

Commission is calculated based on the commission structure defined for each service:

- **Percentage Commission**: `commission = transaction_amount * (commission_percentage / 100)`
- **Fixed Commission**: `commission = fixed_commission_amount`

### Commission Release

Commission earnings are held in your wallet until the next commission release date. The release schedule is typically monthly, but may vary based on your agreement with XashPay.

The next commission release date is included in the wallet balance response:

```json
{
  "commission_release_date": "2025-05-01T00:00:00+00:00"
}
```

### Commission Override

Some vendors may have custom commission rates for specific services. These overrides are managed by XashPay administrators and are automatically applied to your transactions.

## Funding Your Wallet

There are several ways to add funds to your XashPay wallet:

1. **Bank Transfer**: Deposit funds via bank transfer to XashPay's account
2. **Voucher Redemption**: Redeem vouchers to add funds to your wallet
3. **Commission Release**: Commission earnings are released to your available balance on the release date

For bank transfer details and instructions, please contact your account manager or XashPay support.

## Withdrawals

To withdraw funds from your XashPay wallet, you need to submit a withdrawal request through the XashPay dashboard. Withdrawals are processed according to the schedule specified in your agreement with XashPay.

## Security

XashPay implements several security measures to protect your wallet:

1. **Transaction Limits**: Maximum transaction amounts for different service types
2. **Suspicious Activity Monitoring**: Automated systems to detect unusual transaction patterns
3. **Two-Factor Authentication**: Required for high-value transactions and withdrawals
4. **IP Restrictions**: Option to restrict API access to specific IP addresses

## Best Practices

1. **Regular Balance Checks**: Monitor your wallet balance regularly
2. **Transaction Verification**: Verify all transactions against your internal records
3. **Sufficient Funds**: Ensure you have sufficient funds before initiating high-volume transactions
4. **Security Measures**: Implement proper security measures to protect your API credentials

## Troubleshooting

### Insufficient Funds

If you receive an "Insufficient funds" error, check your available balance and ensure you have enough funds to cover the transaction amount plus any applicable fees.

### Transaction Failed

If a transaction fails, check the error message for details. Common causes include:

- Invalid service parameters
- Service provider unavailable
- Network issues
- Account restrictions

### Commission Not Released

If your commission hasn't been released on the expected date, check the commission release date in your wallet balance response. If the date has passed, contact XashPay support.

## Support

For wallet-related issues or questions, contact XashPay support:

- Email: support@xashpay.com
- Phone: +27 12 345 6789
- Support Hours: Monday to Friday, 8:00 AM to 5:00 PM SAST
