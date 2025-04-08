# Authentication

The XashPay API uses token-based authentication to secure all API endpoints. This document explains how to authenticate with the API and manage your authentication tokens.

## Authentication Method

XashPay uses Bearer Token authentication for all API requests. You must include an `Authorization` header with a valid API key in all your requests:

```
Authorization: Bearer YOUR_API_KEY
```

## Obtaining an API Key

### Registration and Account Setup

To use the XashPay API, you must first register for an account:

1. Contact our sales team at sales@xashpay.com to set up your account
2. Complete the registration process and verification requirements
3. Once approved, you'll be given access to the Filament dashboard

### API Keys from Filament Dashboard

API keys are generated and managed exclusively through the Filament dashboard:

1. Log in to your Filament dashboard at https://dashboard.xashpay.com
2. Navigate to the "API Keys" section
3. Click "Generate New API Key"
4. Provide a name for your key (e.g., "Production", "Testing")
5. Copy and securely store your API key immediately - it will only be shown once

**Important**: API keys are sensitive credentials that grant access to your account. Never share your API keys or expose them in client-side code.

## API Key Security

- **No Expiration**: API keys do not automatically expire and remain valid until revoked
- **Revocation**: You can revoke API keys at any time through the Filament dashboard
- **Multiple Keys**: You can generate multiple API keys for different environments or applications

## User Approval

All users must be approved before they can use the API. Your approval status is visible in the Filament dashboard. If your account is not approved, you will not be able to access protected API endpoints.

## Security Best Practices

1. **Store API keys securely**: Never store API keys in client-side code or expose them to users
2. **Use HTTPS**: Always use HTTPS for all API communications
3. **Use separate keys**: Use different API keys for development and production environments
4. **Rotate keys regularly**: Generate new API keys periodically and revoke old ones for enhanced security
5. **Implement least privilege**: Only use API keys with the minimum permissions required for your application

## Error Responses

### Invalid API Key

```json
{
  "success": false,
  "message": "Invalid API key",
  "code": 401
}
```

### Missing API Key

```json
{
  "success": false,
  "message": "API key is missing",
  "code": 401
}
```

### User Not Approved

```json
{
  "success": false,
  "message": "User account is not approved",
  "code": 403
}
```

## Next Steps

Once authenticated, you can:

1. [Explore available services](services.md)
2. [Process payments](api-reference.md#payments)
3. [Manage your wallet](wallet.md)
