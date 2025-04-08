# Services

This document provides detailed information about the various services available through the XashPay API system.

## Service Overview

XashPay offers a range of digital services that vendors can provide to their customers. Each service has specific parameters, commission structures, and processing requirements.

## Available Services

The following services are currently available through the XashPay API:

### Taura Top-up

Taura Top-up allows users to add airtime credit to Taura mobile accounts.

**Service Code:** `taura`

**Category:** Airtime

**Requires Initiation:** No

**Parameters:**
- `phone_number` (string, required): The recipient's 10-digit phone number
- `amount` (numeric, required): The amount to top up (minimum 5)

**Commission Structure:**
- Type: Percentage
- Default Value: 1.5%

**Processing Flow:**
1. Submit a payment request with the required parameters
2. XashPay processes the top-up request with Taura
3. The recipient receives the airtime credit immediately
4. Commission is held and released on the next commission release date

**Example Request:**
```json
{
  "service_code": "taura",
  "phone_number": "0123456789",
  "amount": 50
}
```

**Example Response:**
```json
{
  "success": true,
  "message": "Taura top-up to 0123456789 was completed",
  "data": {
    "reference": "txn_123456",
    "amount": "50.00",
    "units": "50.00",
    "charge": "1.00",
    "commission": "0.75",
    "balance": "949.00",
    "timestamp": "2025-04-07T23:30:00+00:00"
  }
}
```

### DSTV Payment

DSTV Payment allows users to pay their DSTV subscription bills.

**Service Code:** `dstv`

**Category:** TV

**Requires Initiation:** Yes

**Initiation Parameters:**
- `account_number` (string, required): The DSTV account number (8-12 digits)

**Payment Parameters:**
- `amount` (numeric, required): The amount to pay (minimum 10)

**Commission Structure:**
- Type: Percentage
- Default Value: 1.0%

**Processing Flow:**
1. Initiate the service with the account number to validate the account and get the outstanding amount
2. Submit a payment request with the amount
3. XashPay processes the payment with DSTV
4. The customer's DSTV account is credited
5. Commission is held and released on the next commission release date

**Example Initiation Request:**
```json
{
  "service_code": "dstv",
  "account_number": "12345678"
}
```

**Example Initiation Response:**
```json
{
  "success": true,
  "message": "Service initiated successfully",
  "data": {
    "init_id": "init_123456",
    "customer_name": "John Doe",
    "account_status": "active",
    "outstanding_amount": "150.00",
    "expiry": "2025-05-01T00:00:00+00:00"
  }
}
```

**Example Payment Request:**
```json
{
  "service_code": "dstv",
  "amount": 150
}
```

**Example Payment Response:**
```json
{
  "success": true,
  "message": "DSTV payment for account 12345678 was completed",
  "data": {
    "reference": "txn_123457",
    "amount": "150.00",
    "charge": "2.25",
    "commission": "1.50",
    "balance": "796.25",
    "timestamp": "2025-04-07T23:35:00+00:00"
  }
}
```

### OTT Payout

OTT Payout allows vendors to process payouts to customers through the OTT platform.

**Service Code:** `ott_payout`

**Category:** Payout

**Requires Initiation:** No

**Parameters:**
- `recipient_id` (string, required): The recipient's OTT ID
- `amount` (numeric, required): The amount to pay out (minimum 10)
- `reference` (string, required): Your unique reference for this payout

**Commission Structure:**
- Type: Percentage
- Default Value: 0.5%

**Processing Flow:**
1. Submit a payment request with the required parameters
2. XashPay processes the payout request with OTT
3. The recipient receives the funds in their OTT account
4. Commission is held and released on the next commission release date

**Example Request:**
```json
{
  "service_code": "ott_payout",
  "recipient_id": "OTT123456",
  "amount": 200,
  "reference": "payout_ref_001"
}
```

**Example Response:**
```json
{
  "success": true,
  "message": "OTT payout to OTT123456 was completed",
  "data": {
    "reference": "txn_123458",
    "amount": "200.00",
    "charge": "1.00",
    "commission": "1.00",
    "balance": "598.00",
    "timestamp": "2025-04-07T23:40:00+00:00"
  }
}
```

## Other Services

Additional services are regularly added to the XashPay platform. To get the most up-to-date list of available services, use the [List Available Services](api-reference.md#list-available-services) endpoint.

## Service Parameters

Each service has specific parameters that must be provided when making a payment request. These parameters are divided into two categories:

1. **Initiation Parameters**: Required for services that need a two-step process (like DSTV)
2. **Payment Parameters**: Required for processing the actual payment

To get the parameters for a specific service, check the `parameters` array in the service details from the [List Available Services](api-reference.md#list-available-services) endpoint.

## Commission and Charges

Each service has its own commission and charge structure:

- **Commission**: The amount you earn for processing the service
- **Charge**: The fee charged by XashPay for processing the service

Both commission and charges can be either:
- **Percentage**: A percentage of the transaction amount
- **Fixed**: A fixed amount regardless of the transaction amount

Commission is held until the next commission release date, at which point it becomes available in your wallet balance.

## Service Status

You can check the status of any service payment using the [Check Payment Status](api-reference.md#check-payment-status) endpoint with the transaction reference.

## Error Handling

Service-specific errors are returned in the API response with appropriate error codes and messages. Common service-related errors include:

- Invalid service parameters
- Insufficient funds
- Service provider unavailable
- Account validation failed

For more information on error handling, see the [Error Handling](error-handling.md) guide.
