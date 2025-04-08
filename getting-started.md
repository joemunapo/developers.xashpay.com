# Getting Started

This guide will help you get started with the XashPay API Voucher Action feature.

## Prerequisites

Before you begin, ensure you have:

- An active XashPay API account
- API credentials (API key or OAuth tokens)
- Basic understanding of RESTful APIs
- A development environment with PHP 8.0+ (for PHP examples) or any language with HTTP client capabilities

## Making API Requests

The XashPay API is a RESTful API that accepts and returns JSON data. You can use any HTTP client in your preferred programming language to interact with our API.

## Authentication

All API requests require authentication using an API key in the Authorization header:

```
Authorization: Bearer YOUR_API_KEY
```

## Base URL

All API endpoints are relative to the base URL:

```
https://api.xashpay.com/v1
```

## Basic Usage

### Redeeming a Voucher with Action

Here's a basic example of redeeming a voucher with a Taura top-up action:

#### Using cURL

```bash
curl -X POST https://api.xashpay.com/v1/vouchers/redeem \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "provider_code": "OTT",
    "voucher_number": "123456789",
    "voucher_pin": "1234",
    "voucher_action": {
      "service_code": "taura",
      "phone_number": "0123456789"
    }
  }'
```

#### Using PHP with cURL

```php
<?php
// Using PHP cURL
$url = 'https://api.xashpay.com/v1/vouchers/redeem';
$data = [
    'provider_code' => 'OTT',
    'voucher_number' => '123456789',
    'voucher_pin' => '1234',
    'voucher_action' => [
        'service_code' => 'taura',
        'phone_number' => '0123456789'
    ]
];

$ch = curl_init($url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Content-Type: application/json',
    'Accept: application/json',
    'Authorization: Bearer YOUR_API_KEY'
]);

$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

$responseData = json_decode($response, true);

// Check if redemption was successful
if ($responseData['success']) {
    $actionId = $responseData['data']['action_id'];
    echo "Voucher redeemed successfully. Action ID: " . $actionId;
} else {
    echo "Error: " . $responseData['message'];
}
```

#### Using JavaScript Fetch API

```javascript
// Using JavaScript Fetch API
const redeemVoucher = async () => {
  try {
    const response = await fetch('https://api.xashpay.com/v1/vouchers/redeem', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'Authorization': 'Bearer YOUR_API_KEY'
      },
      body: JSON.stringify({
        provider_code: 'OTT',
        voucher_number: '123456789',
        voucher_pin: '1234',
        voucher_action: {
          service_code: 'taura',
          phone_number: '0123456789'
        }
      })
    });
    
    const responseData = await response.json();
    
    // Check if redemption was successful
    if (responseData.success) {
      const actionId = responseData.data.action_id;
      console.log(`Voucher redeemed successfully. Action ID: ${actionId}`);
    } else {
      console.error(`Error: ${responseData.message}`);
    }
    
    return responseData;
  } catch (error) {
    console.error('API request failed:', error);
    throw error;
  }
};

redeemVoucher();
```

### Checking Action Status

#### Using cURL

```bash
curl -X GET https://api.xashpay.com/v1/vouchers/actions/ACTION_ID_FROM_REDEMPTION \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

#### Using PHP with cURL

```php
<?php
// Using PHP cURL
$actionId = 'ACTION_ID_FROM_REDEMPTION';
$url = 'https://api.xashpay.com/v1/vouchers/actions/' . $actionId;

$ch = curl_init($url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Accept: application/json',
    'Authorization: Bearer YOUR_API_KEY'
]);

$response = curl_exec($ch);
curl_close($ch);

$responseData = json_decode($response, true);

// Display action status
if ($responseData['success']) {
    echo "Action status: " . $responseData['data']['status'];
    
    // Display action logs
    echo "\nAction logs:\n";
    foreach ($responseData['data']['logs'] as $log) {
        echo $log['created_at'] . ": " . $log['status'] . " - " . $log['message'] . "\n";
    }
} else {
    echo "Error: " . $responseData['message'];
}
```

#### Using JavaScript Fetch API

```javascript
// Using JavaScript Fetch API
const checkActionStatus = async (actionId) => {
  try {
    const response = await fetch(`https://api.xashpay.com/v1/vouchers/actions/${actionId}`, {
      method: 'GET',
      headers: {
        'Accept': 'application/json',
        'Authorization': 'Bearer YOUR_API_KEY'
      }
    });
    
    const responseData = await response.json();
    
    // Display action status
    if (responseData.success) {
      console.log(`Action status: ${responseData.data.status}`);
      
      // Display action logs
      console.log('Action logs:');
      responseData.data.logs.forEach(log => {
        console.log(`${log.created_at}: ${log.status} - ${log.message}`);
      });
    } else {
      console.error(`Error: ${responseData.message}`);
    }
    
    return responseData;
  } catch (error) {
    console.error('API request failed:', error);
    throw error;
  }
};

const actionId = 'ACTION_ID_FROM_REDEMPTION';
checkActionStatus(actionId);
```

## Next Steps

Now that you understand the basics, you can:

1. Explore the [API Reference](api-reference.md) for detailed endpoint documentation
2. Check out the [Examples](examples.md) for more complex scenarios
3. Learn about error handling in the [Troubleshooting](troubleshooting.md) section
