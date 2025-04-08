# API Reference

This comprehensive reference documents all endpoints available in the XashPay API system.

## Base URL

```
https://api.xashpay.com/v1
```

## Authentication

All API requests require authentication using a bearer token. Include your API token in the Authorization header:

```
Authorization: Bearer YOUR_API_TOKEN
```

For more details on authentication, see the [Authentication](authentication.md) guide.

## Response Format

All API responses follow a consistent format:

```json
{
  "success": true,
  "message": "Operation successful message",
  "data": {
    // Response data varies by endpoint
  }
}
```

Error responses:

```json
{
  "success": false,
  "message": "Error message describing what went wrong",
  "code": 422
}
```

## User and Profile

### Get User Profile

Retrieves the authenticated user's profile information.

**Endpoint:** `GET /profile`

**Response:**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Vendor Name",
    "email": "vendor@example.com",
    "is_approved": true,
    "created_at": "2025-01-01T00:00:00.000000Z",
    "updated_at": "2025-01-01T00:00:00.000000Z"
  }
}
```

## Wallet

### Get Wallet Balance

Retrieves the current wallet balance and commission information.

**Endpoint:** `GET /wallet`

**Response:**

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

## Services

### List Available Services

Retrieves all services available to the authenticated user.

**Endpoint:** `GET /services`

**Response:**

```json
{
  "success": true,
  "services": [
    {
      "code": "taura",
      "name": "Taura Top-up",
      "category": "airtime",
      "requires_initiation": false,
      "parameters": [
        {
          "name": "phone_number",
          "type": "string",
          "required": true,
          "validation": "required|string|min:10|max:10",
          "step": "pay"
        },
        {
          "name": "amount",
          "type": "numeric",
          "required": true,
          "validation": "required|numeric|min:5",
          "step": "pay"
        }
      ],
      "commission": {
        "type": "percentage",
        "value": 1.5
      },
      "charge": {
        "type": "percentage",
        "value": 2.0
      }
    },
    {
      "code": "dstv",
      "name": "DSTV Payment",
      "category": "tv",
      "requires_initiation": true,
      "parameters": [
        {
          "name": "account_number",
          "type": "string",
          "required": true,
          "validation": "required|string|min:8|max:12",
          "step": "initiate"
        },
        {
          "name": "amount",
          "type": "numeric",
          "required": true,
          "validation": "required|numeric|min:10",
          "step": "pay"
        }
      ],
      "commission": {
        "type": "percentage",
        "value": 1.0
      },
      "charge": {
        "type": "percentage",
        "value": 1.5
      }
    }
  ],
  "balance": "1000.00"
}
```

### Initiate Service

Initiates a service that requires a two-step process (like DSTV payments).

**Endpoint:** `POST /initiate`

**Request Body:**

```json
{
  "service_code": "dstv",
  "account_number": "12345678"
}
```

**Response:**

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

## Payments

### Process Payment

Processes a payment for a service.

**Endpoint:** `POST /pay/{service_init?}`

**Request Body (Direct Payment):**

```json
{
  "service_code": "taura",
  "phone_number": "0123456789",
  "amount": 50
}
```

**Request Body (After Initiation):**

```json
{
  "service_code": "dstv",
  "amount": 150
}
```

**Response:**

```json
{
  "success": true,
  "message": "Payment processed successfully",
  "data": {
    "reference": "txn_123456",
    "amount": "50.00",
    "charge": "1.00",
    "commission": "0.75",
    "balance": "949.00",
    "timestamp": "2025-04-07T23:30:00+00:00"
  }
}
```

### Check Payment Status

Checks the status of a previous payment.

**Endpoint:** `GET /status/{reference}`

**Response:**

```json
{
  "success": true,
  "message": "Transaction status retrieved",
  "data": {
    "reference": "txn_123456",
    "status": "completed",
    "amount": "50.00",
    "service": "Taura Top-up",
    "recipient": "0123456789",
    "timestamp": "2025-04-07T23:30:00+00:00"
  }
}
```

## Vouchers

### List Voucher Providers

Retrieves a list of supported voucher providers.

**Endpoint:** `GET /vouchers/providers`

**Response:**

```json
{
  "success": true,
  "message": "Active voucher providers retrieved",
  "data": {
    "providers": [
      {
        "name": "OTT Vouchers",
        "code": "OTT",
        "fee_percentage": 5.0,
        "required_fields": ["voucher_number", "voucher_pin"]
      },
      {
        "name": "Blu Voucher",
        "code": "BLU",
        "fee_percentage": 3.5,
        "required_fields": ["voucher_number"]
      }
    ]
  }
}
```

### Validate Voucher

Validates a voucher without redeeming it.

**Endpoint:** `POST /vouchers/validate`

**Request Body:**

```json
{
  "provider_code": "OTT",
  "voucher_number": "123456789",
  "voucher_pin": "1234"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Voucher is valid",
  "data": {
    "status": "VALID",
    "is_valid": true,
    "amount": 100.00,
    "provider": "OTT Vouchers"
  }
}
```

### Redeem Voucher

Redeems a voucher and optionally performs an action with the funds.

**Endpoint:** `POST /vouchers/redeem`

**Request Body (Simple Redemption):**

```json
{
  "provider_code": "OTT",
  "voucher_number": "123456789",
  "voucher_pin": "1234",
  "client_id": "OPTIONAL_CLIENT_REFERENCE"
}
```

**Request Body (With Action):**

```json
{
  "provider_code": "OTT",
  "voucher_number": "123456789",
  "voucher_pin": "1234",
  "client_id": "OPTIONAL_CLIENT_REFERENCE",
  "voucher_action": {
    "service_code": "taura",
    "phone_number": "0123456789"
  }
}
```

**Response (Simple Redemption):**

```json
{
  "success": true,
  "message": "Voucher redeemed successfully",
  "data": {
    "redeemed": true,
    "amount": "100.00",
    "provider": "OTT Vouchers",
    "currency": "ZAR",
    "fee": "5.00",
    "balance": "195.00",
    "timestamp": "2025-04-07T23:30:00+00:00"
  }
}
```

**Response (With Action):**

```json
{
  "success": true,
  "message": "Voucher redeemed successfully",
  "data": {
    "redeemed": true,
    "amount": "100.00",
    "provider": "OTT Vouchers",
    "currency": "ZAR",
    "fee": "5.00",
    "balance": "195.00",
    "timestamp": "2025-04-07T23:30:00+00:00",
    "action_id": "1",
    "action_status": "pending",
    "action_reference": "VAct6071a2b3e4f5c"
  }
}
```

### Check Voucher Redemption Status

Checks the status of a voucher redemption.

**Endpoint:** `GET /vouchers/status/{reference}`

**Response:**

```json
{
  "success": true,
  "message": "Redemption status retrieved",
  "data": {
    "reference": "rdm_123456",
    "status": "completed",
    "amount": "100.00",
    "provider": "OTT Vouchers",
    "timestamp": "2025-04-07T23:30:00+00:00"
  }
}
```

### Check Voucher Action Status

Retrieves the status of a voucher action.

**Endpoint:** `GET /vouchers/actions/{actionId}`

**Response:**

```json
{
  "success": true,
  "message": "Action status retrieved",
  "data": {
    "action_id": "1",
    "action_type": "taura",
    "status": "completed",
    "amount": "100.00",
    "attempts": 1,
    "last_attempt": "2025-04-07T23:31:00+00:00",
    "created_at": "2025-04-07T23:30:00+00:00",
    "logs": [
      {
        "id": 1,
        "voucher_action_id": 1,
        "status": "pending",
        "message": "Action created and queued for processing",
        "created_at": "2025-04-07T23:30:00+00:00"
      },
      {
        "id": 2,
        "voucher_action_id": 1,
        "status": "processing",
        "message": "Starting to process the action",
        "created_at": "2025-04-07T23:30:05+00:00"
      },
      {
        "id": 3,
        "voucher_action_id": 1,
        "status": "completed",
        "message": "Action processed successfully",
        "created_at": "2025-04-07T23:31:00+00:00"
      }
    ]
  }
}
```

### Retry Failed Action

Retries a failed voucher action.

**Endpoint:** `POST /vouchers/actions/{actionId}/retry`

**Response:**

```json
{
  "success": true,
  "message": "Action retry initiated",
  "data": {
    "action_id": "1",
    "status": "pending"
  }
}
```

## Error Codes

| Code | Description |
|------|-------------|
| 400 | Bad Request - The request was malformed or contains invalid parameters |
| 401 | Unauthorized - Authentication failed or token is invalid |
| 403 | Forbidden - The authenticated user doesn't have permission for the requested operation |
| 404 | Not Found - The requested resource doesn't exist |
| 409 | Conflict - The request couldn't be completed due to a conflict (e.g., voucher already redeemed) |
| 422 | Unprocessable Entity - Validation failed or the request couldn't be processed |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error - Something went wrong on the server |

## Rate Limits

The API implements rate limiting to protect against abuse:

- 100 requests per minute per API key
- 5,000 requests per day per API key

Rate limit headers are included in all responses:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1617840000
```
