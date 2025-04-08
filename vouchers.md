# Voucher System

This document provides detailed information about the voucher system in the XashPay API platform.

## Overview

The XashPay voucher system allows users to validate and redeem vouchers from multiple providers. The system also supports voucher actions, which enable users to immediately use voucher funds for specific services like Taura top-ups.

## Voucher Providers

XashPay supports multiple voucher providers, each with their own validation requirements and fee structures.

### Supported Providers

Currently supported voucher providers include:

| Provider | Code | Fee Structure | Required Fields |
|----------|------|---------------|----------------|
| OTT Vouchers | OTT | 5.0% | voucher_number, voucher_pin |
| Blu Voucher | BLU | 3.5% | voucher_number |

To get the most up-to-date list of supported providers, use the [List Voucher Providers](api-reference.md#list-voucher-providers) endpoint.

## Validation

Before redeeming a voucher, you can validate it to check if it's valid and hasn't been used before.

### Validation Process

1. Submit a validation request with the required provider code and voucher details
2. XashPay checks with the voucher provider to verify the voucher's validity
3. If valid, the response includes the voucher amount and status

### Validation Statuses

| Status | Description |
|--------|-------------|
| VALID | The voucher is valid and can be redeemed |
| INVALID | The voucher is invalid (incorrect number or PIN) |
| REDEEMED | The voucher has already been redeemed |
| EXPIRED | The voucher has expired |

## Redemption

Redeeming a voucher adds its value to your XashPay wallet balance, minus any applicable fees.

### Redemption Process

1. Submit a redemption request with the required provider code and voucher details
2. XashPay verifies the voucher with the provider
3. If valid, the voucher value (minus fees) is added to your wallet balance
4. A redemption reference is returned for tracking

### Simple Redemption

Simple redemption adds the voucher value to your wallet balance without performing any additional actions.

### Redemption with Actions

You can optionally specify an action to perform with the voucher funds immediately after redemption. This allows for seamless integration between voucher redemption and service utilization.

## Voucher Actions

Voucher actions enable users to immediately use voucher funds for specific services without requiring separate API calls.

### Supported Actions

Any service available through the XashPay API can be used as a voucher action. Common actions include:

- Taura top-ups
- DSTV payments
- OTT payouts

### Action Processing

1. The voucher is redeemed and the value is added to your wallet
2. The specified action is queued for processing
3. The action is processed asynchronously
4. You can check the action status using the provided action ID

### Action Statuses

| Status | Description |
|--------|-------------|
| pending | The action has been queued but not yet processed |
| processing | The action is currently being processed |
| completed | The action has been successfully completed |
| failed | The action failed to process |

### Retrying Failed Actions

If an action fails, you can retry it using the [Retry Failed Action](api-reference.md#retry-failed-action) endpoint.

## Implementation Example

### Validating a Voucher (PHP cURL)

```php
// Using PHP cURL
function validateVoucher($token, $providerCode, $voucherNumber, $voucherPin) {
    $url = 'https://api.xashpay.com/v1/vouchers/validate';
    $data = [
        'provider_code' => $providerCode,
        'voucher_number' => $voucherNumber,
        'voucher_pin' => $voucherPin
    ];
    
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Content-Type: application/json',
        'Accept: application/json',
        'Authorization: Bearer ' . $token
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    $responseData = json_decode($response, true);
    
    if ($responseData['success'] && $responseData['data']['is_valid']) {
        echo "Voucher is valid. Amount: " . $responseData['data']['amount'];
    } else {
        echo "Voucher is invalid: " . $responseData['message'];
    }
    
    return $responseData;
}

// Usage
$token = 'YOUR_API_TOKEN';
validateVoucher($token, 'OTT', '123456789', '1234');
```

### Validating a Voucher (JavaScript Fetch)

```javascript
// Using JavaScript Fetch API
async function validateVoucher(token, providerCode, voucherNumber, voucherPin) {
  try {
    const response = await fetch('https://api.xashpay.com/v1/vouchers/validate', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify({
        provider_code: providerCode,
        voucher_number: voucherNumber,
        voucher_pin: voucherPin
      })
    });
    
    const responseData = await response.json();
    
    if (responseData.success && responseData.data.is_valid) {
      console.log(`Voucher is valid. Amount: ${responseData.data.amount}`);
    } else {
      console.log(`Voucher is invalid: ${responseData.message}`);
    }
    
    return responseData;
  } catch (error) {
    console.error('Validation failed:', error.message);
    throw error;
  }
}

// Usage
const token = 'YOUR_API_TOKEN';
validateVoucher(token, 'OTT', '123456789', '1234');
```

### Redeeming a Voucher with Action (PHP cURL)

```php
// Using PHP cURL
function redeemVoucherWithAction($token, $providerCode, $voucherNumber, $voucherPin, $action) {
    $url = 'https://api.xashpay.com/v1/vouchers/redeem';
    $data = [
        'provider_code' => $providerCode,
        'voucher_number' => $voucherNumber,
        'voucher_pin' => $voucherPin,
        'voucher_action' => $action
    ];
    
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Content-Type: application/json',
        'Accept: application/json',
        'Authorization: Bearer ' . $token
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    $responseData = json_decode($response, true);
    
    if ($responseData['success']) {
        echo "Voucher redeemed successfully. Amount: " . $responseData['data']['amount'] . "\n";
        echo "Action ID: " . $responseData['data']['action_id'] . "\n";
        echo "Action Status: " . $responseData['data']['action_status'] . "\n";
    } else {
        echo "Redemption failed: " . $responseData['message'] . "\n";
    }
    
    return $responseData;
}

// Usage
$token = 'YOUR_API_TOKEN';
$action = [
    'service_code' => 'taura',
    'phone_number' => '0123456789'
];
redeemVoucherWithAction($token, 'OTT', '123456789', '1234', $action);
```

### Redeeming a Voucher with Action (JavaScript Axios)

```javascript
// Using JavaScript Axios
const axios = require('axios');

async function redeemVoucherWithAction(token, providerCode, voucherNumber, voucherPin, action) {
  try {
    const response = await axios.post('https://api.xashpay.com/v1/vouchers/redeem', {
      provider_code: providerCode,
      voucher_number: voucherNumber,
      voucher_pin: voucherPin,
      voucher_action: action
    }, {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    });
    
    const responseData = response.data;
    
    if (responseData.success) {
      console.log(`Voucher redeemed successfully. Amount: ${responseData.data.amount}`);
      console.log(`Action ID: ${responseData.data.action_id}`);
      console.log(`Action Status: ${responseData.data.action_status}`);
    } else {
      console.log(`Redemption failed: ${responseData.message}`);
    }
    
    return responseData;
  } catch (error) {
    console.error('Redemption failed:', error.response?.data?.message || error.message);
    throw error;
  }
}

// Usage
const token = 'YOUR_API_TOKEN';
const action = {
  service_code: 'taura',
  phone_number: '0123456789'
};
redeemVoucherWithAction(token, 'OTT', '123456789', '1234', action);
```

### Checking Action Status (PHP cURL)

```php
// Using PHP cURL
function checkActionStatus($token, $actionId) {
    $url = 'https://api.xashpay.com/v1/vouchers/actions/' . $actionId;
    
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Accept: application/json',
        'Authorization: Bearer ' . $token
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    $responseData = json_decode($response, true);
    
    if ($responseData['success']) {
        echo "Action status: " . $responseData['data']['status'] . "\n";
        
        // Display action logs
        echo "Action logs:\n";
        foreach ($responseData['data']['logs'] as $log) {
            echo $log['created_at'] . ": " . $log['status'] . " - " . $log['message'] . "\n";
        }
    } else {
        echo "Failed to get action status: " . $responseData['message'] . "\n";
    }
    
    return $responseData;
}

// Usage
$token = 'YOUR_API_TOKEN';
$actionId = 'ACTION_ID_FROM_REDEMPTION';
checkActionStatus($token, $actionId);
```

### Checking Action Status (JavaScript Fetch)

```javascript
// Using JavaScript Fetch API
async function checkActionStatus(token, actionId) {
  try {
    const response = await fetch(`https://api.xashpay.com/v1/vouchers/actions/${actionId}`, {
      method: 'GET',
      headers: {
        'Accept': 'application/json',
        'Authorization': `Bearer ${token}`
      }
    });
    
    const responseData = await response.json();
    
    if (responseData.success) {
      console.log(`Action status: ${responseData.data.status}`);
      
      // Display action logs
      console.log('Action logs:');
      responseData.data.logs.forEach(log => {
        console.log(`${log.created_at}: ${log.status} - ${log.message}`);
      });
    } else {
      console.log(`Failed to get action status: ${responseData.message}`);
    }
    
    return responseData;
  } catch (error) {
    console.error('Status check failed:', error.message);
    throw error;
  }
}

// Usage
const token = 'YOUR_API_TOKEN';
const actionId = 'ACTION_ID_FROM_REDEMPTION';
checkActionStatus(token, actionId);
```

## Best Practices

1. **Always validate vouchers** before attempting redemption
2. **Include a client reference** in redemption requests for your internal tracking
3. **Check action status** after redemption with action to confirm completion
4. **Implement proper error handling** for validation and redemption failures
5. **Set up webhooks** to receive notifications about action status changes

## Error Handling

Common voucher-related errors include:

- Invalid voucher number or PIN
- Voucher already redeemed
- Voucher expired
- Invalid action parameters
- Insufficient voucher value for the requested action

For more information on error handling, see the [Error Handling](error-handling.md) guide.
