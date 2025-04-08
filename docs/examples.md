# Examples

This document provides comprehensive code examples for using the XashPay API system using standard HTTP clients and curl.

## Authentication

### PHP (Using cURL)

```php
<?php
// Using PHP cURL
function authenticate($email, $password) {
    $url = 'https://api.xashpay.com/v1/login';
    $data = [
        'email' => $email,
        'password' => $password
    ];
    
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Content-Type: application/json',
        'Accept: application/json'
    ]);
    
    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    $responseData = json_decode($response, true);
    
    if ($httpCode == 200 && $responseData['success']) {
        $token = $responseData['data']['token'];
        $user = $responseData['data']['user'];
        
        echo "Authentication successful! User: " . $user['name'] . "\n";
        return $token;
    } else {
        $message = isset($responseData['message']) ? $responseData['message'] : 'Unknown error';
        echo "Authentication failed: " . $message . "\n";
        return null;
    }
}

// Usage
$token = authenticate('your-email@example.com', 'your-password');
if ($token) {
    // Store the token for subsequent requests
    echo "Token: " . $token . "\n";
}
```

### PHP (Using Guzzle)

```php
<?php
// Using Guzzle HTTP client
require 'vendor/autoload.php';

use GuzzleHttp\Client;
use GuzzleHttp\Exception\RequestException;

function authenticate($email, $password) {
    $client = new Client([
        'base_uri' => 'https://api.xashpay.com/v1/',
        'timeout' => 30,
        'headers' => [
            'Content-Type' => 'application/json',
            'Accept' => 'application/json'
        ]
    ]);
    
    try {
        $response = $client->post('login', [
            'json' => [
                'email' => $email,
                'password' => $password
            ]
        ]);
        
        $responseData = json_decode($response->getBody(), true);
        
        if ($responseData['success']) {
            $token = $responseData['data']['token'];
            $user = $responseData['data']['user'];
            
            echo "Authentication successful! User: " . $user['name'] . "\n";
            return $token;
        } else {
            echo "Authentication failed: " . $responseData['message'] . "\n";
            return null;
        }
    } catch (RequestException $e) {
        if ($e->hasResponse()) {
            $responseData = json_decode($e->getResponse()->getBody(), true);
            echo "Authentication failed: " . $responseData['message'] . "\n";
        } else {
            echo "Authentication failed: " . $e->getMessage() . "\n";
        }
        return null;
    }
}

// Usage
$token = authenticate('your-email@example.com', 'your-password');
if ($token) {
    // Store the token for subsequent requests
    echo "Token: " . $token . "\n";
}
```

### JavaScript (Using Fetch)

```javascript
// Using JavaScript Fetch API
async function authenticate(email, password) {
  try {
    const response = await fetch('https://api.xashpay.com/v1/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
      },
      body: JSON.stringify({
        email: email,
        password: password
      })
    });
    
    const responseData = await response.json();
    
    if (response.ok && responseData.success) {
      const { token, user } = responseData.data;
      console.log(`Authentication successful! User: ${user.name}`);
      return token;
    } else {
      console.error('Authentication failed:', responseData.message);
      return null;
    }
  } catch (error) {
    console.error('Authentication failed:', error.message);
    return null;
  }
}

// Using the token for subsequent requests
async function makeAuthenticatedRequest(token) {
  try {
    const response = await fetch('https://api.xashpay.com/v1/profile', {
      method: 'GET',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Accept': 'application/json'
      }
    });
    
    const responseData = await response.json();
    
    if (response.ok && responseData.success) {
      console.log('User profile:', responseData.data);
    } else {
      console.error('Request failed:', responseData.message);
    }
  } catch (error) {
    console.error('Request failed:', error.message);
  }
}

// Usage
authenticate('your-email@example.com', 'your-password')
  .then(token => {
    if (token) {
      makeAuthenticatedRequest(token);
    }
  })
  .catch(err => console.error('Error:', err));
```

### JavaScript (Using Axios)

```javascript
// Using JavaScript/Node.js with Axios
const axios = require('axios');

async function authenticate(email, password) {
  try {
    const response = await axios.post('https://api.xashpay.com/v1/login', {
      email: email,
      password: password
    });
    
    const { token, user } = response.data.data;
    console.log(`Authentication successful! User: ${user.name}`);
    
    return token;
  } catch (error) {
    console.error('Authentication failed:', error.response?.data?.message || error.message);
    return null;
  }
}

// Using the token for subsequent requests
async function makeAuthenticatedRequest(token) {
  try {
    const response = await axios.get('https://api.xashpay.com/v1/profile', {
      headers: {
        'Authorization': `Bearer ${token}`
      }
    });
    
    console.log('User profile:', response.data.data);
  } catch (error) {
    console.error('Request failed:', error.response?.data?.message || error.message);
  }
}

// Usage
authenticate('your-email@example.com', 'your-password')
  .then(token => {
    if (token) {
      makeAuthenticatedRequest(token);
    }
  })
  .catch(err => console.error('Error:', err));
```

### cURL Command

```bash
# Authentication using cURL
curl -X POST https://api.xashpay.com/v1/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "your-email@example.com",
    "password": "your-password"
  }'
```

## Service Payments

### Taura Top-up (PHP cURL)

```php
<?php
// Using PHP cURL
function processPayment($token, $serviceCode, $params) {
    $url = 'https://api.xashpay.com/v1/pay';
    $data = array_merge(['service_code' => $serviceCode], $params);
    
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
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    $responseData = json_decode($response, true);
    
    if ($httpCode == 200 && $responseData['success']) {
        echo "Payment successful!\n";
        echo "Reference: " . $responseData['data']['reference'] . "\n";
        echo "Amount: " . $responseData['data']['amount'] . "\n";
        if (isset($responseData['data']['units'])) {
            echo "Units: " . $responseData['data']['units'] . "\n";
        }
        echo "New Balance: " . $responseData['data']['balance'] . "\n";
        return $responseData['data'];
    } else {
        $message = isset($responseData['message']) ? $responseData['message'] : 'Unknown error';
        echo "Payment failed: " . $message . "\n";
        return null;
    }
}

// Usage for Taura top-up
$token = 'YOUR_API_TOKEN';
$result = processPayment($token, 'taura', [
    'phone_number' => '0123456789',
    'amount' => 50
]);
```

### DSTV Payment (JavaScript Axios)

```javascript
// Using JavaScript/Node.js with Axios
const axios = require('axios');

// Set up axios with authentication
const api = axios.create({
  baseURL: 'https://api.xashpay.com/v1',
  headers: {
    'Authorization': 'Bearer YOUR_API_TOKEN',
    'Content-Type': 'application/json'
  }
});

// Step 1: Initiate the service to validate the account
async function initiateDstvPayment(accountNumber) {
  try {
    const response = await api.post('/initiate', {
      service_code: 'dstv',
      account_number: accountNumber
    });
    
    console.log('Service initiated successfully!');
    console.log('Customer:', response.data.data.customer_name);
    console.log('Outstanding amount:', response.data.data.outstanding_amount);
    
    return response.data.data.init_id;
  } catch (error) {
    console.error('Initiation failed:', error.response?.data?.message || error.message);
    throw error;
  }
}

// Step 2: Process the payment
async function processDstvPayment(initId, amount) {
  try {
    const response = await api.post(`/pay/${initId}`, {
      service_code: 'dstv',
      amount: amount
    });
    
    console.log('Payment successful!');
    console.log('Reference:', response.data.data.reference);
    console.log('Amount:', response.data.data.amount);
    console.log('New Balance:', response.data.data.balance);
    
    return response.data.data.reference;
  } catch (error) {
    console.error('Payment failed:', error.response?.data?.message || error.message);
    throw error;
  }
}

// Usage
async function payDstvBill(accountNumber, amount) {
  try {
    const initId = await initiateDstvPayment(accountNumber);
    const reference = await processDstvPayment(initId, amount);
    console.log('DSTV payment completed with reference:', reference);
  } catch (error) {
    console.error('DSTV payment process failed:', error);
  }
}

payDstvBill('12345678', 150);
```

### cURL Commands

```bash
# Step 1: Initiate DSTV payment
curl -X POST https://api.xashpay.com/v1/initiate \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{
    "service_code": "dstv",
    "account_number": "12345678"
  }'

# Step 2: Process DSTV payment (replace INIT_ID with the init_id from step 1)
curl -X POST https://api.xashpay.com/v1/pay/INIT_ID \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{
    "service_code": "dstv",
    "amount": 150
  }'

# Direct payment (Taura top-up)
curl -X POST https://api.xashpay.com/v1/pay \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{
    "service_code": "taura",
    "phone_number": "0123456789",
    "amount": 50
  }'
```

## Voucher Redemption

### Simple Voucher Redemption (PHP cURL)

```php
<?php
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
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    return json_decode($response, true);
}

function redeemVoucher($token, $providerCode, $voucherNumber, $voucherPin, $clientId = null) {
    $url = 'https://api.xashpay.com/v1/vouchers/redeem';
    $data = [
        'provider_code' => $providerCode,
        'voucher_number' => $voucherNumber,
        'voucher_pin' => $voucherPin
    ];
    
    if ($clientId) {
        $data['client_id'] = $clientId;
    }
    
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
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    return json_decode($response, true);
}

// Usage
$token = 'YOUR_API_TOKEN';
$providerCode = 'OTT';
$voucherNumber = '123456789';
$voucherPin = '1234';

// First, validate the voucher
$validateResponse = validateVoucher($token, $providerCode, $voucherNumber, $voucherPin);

if (!$validateResponse['success'] || !$validateResponse['data']['is_valid']) {
    echo "Voucher validation failed: " . $validateResponse['message'] . "\n";
    exit;
}

echo "Voucher is valid. Amount: " . $validateResponse['data']['amount'] . "\n";

// Now redeem the voucher
$clientId = 'CLIENT_REF_' . time(); // Optional client reference
$redeemResponse = redeemVoucher($token, $providerCode, $voucherNumber, $voucherPin, $clientId);

if ($redeemResponse['success']) {
    echo "Voucher redeemed successfully!\n";
    echo "Amount: " . $redeemResponse['data']['amount'] . "\n";
    echo "Fee: " . $redeemResponse['data']['fee'] . "\n";
    echo "New Balance: " . $redeemResponse['data']['balance'] . "\n";
} else {
    echo "Redemption failed: " . $redeemResponse['message'] . "\n";
}
```

### Voucher Redemption with Action (JavaScript Fetch)

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
    
    return await response.json();
  } catch (error) {
    console.error('Validation failed:', error.message);
    throw error;
  }
}

async function redeemVoucherWithAction(token, providerCode, voucherNumber, voucherPin, action) {
  try {
    const clientId = `CLIENT_REF_${Date.now()}`;
    
    const response = await fetch('https://api.xashpay.com/v1/vouchers/redeem', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify({
        provider_code: providerCode,
        voucher_number: voucherNumber,
        voucher_pin: voucherPin,
        client_id: clientId,
        voucher_action: action
      })
    });
    
    return await response.json();
  } catch (error) {
    console.error('Redemption failed:', error.message);
    throw error;
  }
}

async function checkActionStatus(token, actionId) {
  try {
    const response = await fetch(`https://api.xashpay.com/v1/vouchers/actions/${actionId}`, {
      method: 'GET',
      headers: {
        'Accept': 'application/json',
        'Authorization': `Bearer ${token}`
      }
    });
    
    return await response.json();
  } catch (error) {
    console.error('Status check failed:', error.message);
    throw error;
  }
}

// Usage
async function redeemVoucherWithTauraTopup(token, voucherNumber, voucherPin, phoneNumber) {
  try {
    // First validate the voucher
    const validateResponse = await validateVoucher(token, 'OTT', voucherNumber, voucherPin);
    
    if (!validateResponse.success || !validateResponse.data.is_valid) {
      console.error('Voucher validation failed:', validateResponse.message);
      return;
    }
    
    console.log('Voucher is valid. Amount:', validateResponse.data.amount);
    
    // Redeem the voucher with Taura top-up action
    const redeemResponse = await redeemVoucherWithAction(
      token,
      'OTT',
      voucherNumber,
      voucherPin,
      {
        service_code: 'taura',
        phone_number: phoneNumber
      }
    );
    
    if (!redeemResponse.success) {
      console.error('Redemption failed:', redeemResponse.message);
      return;
    }
    
    console.log('Voucher redeemed with Taura top-up action!');
    console.log('Amount:', redeemResponse.data.amount);
    console.log('Action ID:', redeemResponse.data.action_id);
    console.log('Action Status:', redeemResponse.data.action_status);
    
    // Check action status after a short delay
    setTimeout(async () => {
      const statusResponse = await checkActionStatus(token, redeemResponse.data.action_id);
      
      console.log('Action Status:', statusResponse.data.status);
      console.log('Action Logs:');
      statusResponse.data.logs.forEach(log => {
        console.log(`- ${log.created_at}: ${log.status} - ${log.message}`);
      });
      
      // If action is still pending or processing, check again after a delay
      if (['pending', 'processing'].includes(statusResponse.data.status)) {
        console.log('Action still in progress. Checking again in 5 seconds...');
        setTimeout(() => checkActionStatus(token, redeemResponse.data.action_id), 5000);
      }
    }, 5000);
  } catch (error) {
    console.error('Error:', error.message);
  }
}

// Usage
const token = 'YOUR_API_TOKEN';
redeemVoucherWithTauraTopup(token, '123456789', '1234', '0123456789');
```

### cURL Commands

```bash
# Validate voucher
curl -X POST https://api.xashpay.com/v1/vouchers/validate \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{
    "provider_code": "OTT",
    "voucher_number": "123456789",
    "voucher_pin": "1234"
  }'

# Redeem voucher (simple)
curl -X POST https://api.xashpay.com/v1/vouchers/redeem \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{
    "provider_code": "OTT",
    "voucher_number": "123456789",
    "voucher_pin": "1234",
    "client_id": "CLIENT_REF_123456"
  }'

# Redeem voucher with action
curl -X POST https://api.xashpay.com/v1/vouchers/redeem \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{
    "provider_code": "OTT",
    "voucher_number": "123456789",
    "voucher_pin": "1234",
    "client_id": "CLIENT_REF_123456",
    "voucher_action": {
      "service_code": "taura",
      "phone_number": "0123456789"
    }
  }'

# Check action status
curl -X GET https://api.xashpay.com/v1/vouchers/actions/ACTION_ID \
  -H "Authorization: Bearer YOUR_API_TOKEN"
```

## Wallet Operations

### Checking Wallet Balance (PHP cURL)

```php
<?php
// Using PHP cURL
function getWalletBalance($token) {
    $url = 'https://api.xashpay.com/v1/wallet';
    
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Accept: application/json',
        'Authorization: Bearer ' . $token
    ]);
    
    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    $responseData = json_decode($response, true);
    
    if ($httpCode == 200 && $responseData['success']) {
        echo "Wallet Balance:\n";
        echo "Currency: " . $responseData['data']['currency'] . "\n";
        echo "Available Balance: " . $responseData['data']['balance'] . "\n";
        echo "Commission on Hold: " . $responseData['data']['profit_on_hold'] . "\n";
        echo "Next Commission Release: " . $responseData['commission_release_date'] . "\n";
        return $responseData['data'];
    } else {
        $message = isset($responseData['message']) ? $responseData['message'] : 'Unknown error';
        echo "Failed to retrieve wallet: " . $message . "\n";
        return null;
    }
}

// Usage
$token = 'YOUR_API_TOKEN';
$walletData = getWalletBalance($token);
```

### Checking Wallet Balance (JavaScript Axios)

```javascript
// Using JavaScript/Node.js with Axios
const axios = require('axios');

async function checkWalletBalance(token) {
  try {
    const response = await axios.get('https://api.xashpay.com/v1/wallet', {
      headers: {
        'Authorization': `Bearer ${token}`
      }
    });
    
    console.log('Wallet Balance:');
    console.log('Currency:', response.data.data.currency);
    console.log('Available Balance:', response.data.data.balance);
    console.log('Commission on Hold:', response.data.data.profit_on_hold);
    console.log('Next Commission Release:', response.data.commission_release_date);
    
    return response.data.data;
  } catch (error) {
    console.error('Error checking wallet:', error.response?.data?.message || error.message);
    throw error;
  }
}

// Usage
const token = 'YOUR_API_TOKEN';
checkWalletBalance(token)
  .then(walletData => {
    // Do something with the wallet data
    if (parseFloat(walletData.balance) < 100) {
      console.log('Warning: Low balance!');
    }
  })
  .catch(err => console.error('Failed to check wallet:', err));
```

### cURL Command

```bash
# Get wallet balance
curl -X GET https://api.xashpay.com/v1/wallet \
  -H "Authorization: Bearer YOUR_API_TOKEN"
```

## Error Handling

### Comprehensive Error Handling (PHP cURL)

```php
<?php
// Using PHP cURL with error handling
function makeApiRequest($token, $method, $endpoint, $data = null) {
    $url = 'https://api.xashpay.com/v1/' . $endpoint;
    
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);
    
    $headers = [
        'Accept: application/json',
        'Authorization: Bearer ' . $token
    ];
    
    if ($data) {
        $headers[] = 'Content-Type: application/json';
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
    }
    
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    
    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    
    if (curl_errno($ch)) {
        $error = curl_error($ch);
        curl_close($ch);
        throw new Exception("cURL Error: " . $error);
    }
    
    curl_close($ch);
    
    $responseData = json_decode($response, true);
    
    if ($httpCode >= 400) {
        $errorMessage = isset($responseData['message']) ? $responseData['message'] : 'Unknown error';
        
        switch ($httpCode) {
            case 401:
                throw new Exception("Authentication error: " . $errorMessage, 401);
            case 403:
                throw new Exception("Permission error: " . $errorMessage, 403);
            case 404:
                throw new Exception("Resource not found: " . $errorMessage, 404);
            case 422:
                $validationErrors = isset($responseData['errors']) ? json_encode($responseData['errors']) : '';
                throw new Exception("Validation error: " . $errorMessage . " " . $validationErrors, 422);
            case 429:
                throw new Exception("Rate limit exceeded: " . $errorMessage, 429);
            default:
                throw new Exception("API error (" . $httpCode . "): " . $errorMessage, $httpCode);
        }
    }
    
    return $responseData;
}

// Usage with retry for rate limiting
function makeRequestWithRetry($token, $method, $endpoint, $data = null, $maxRetries = 3) {
    $retries = 0;
    $delay = 1000; // Start with 1 second delay
    
    while (true) {
        try {
            return makeApiRequest($token, $method, $endpoint, $data);
        } catch (Exception $e) {
            $retries++;
            
            // Don't retry if we've reached the maximum number of retries
            if ($retries >= $maxRetries) {
                throw $e;
            }
            
            // Only retry for rate limiting errors
            if ($e->getCode() != 429) {
                throw $e;
            }
            
            // Exponential backoff with jitter
            $jitter = rand(0, 100) / 100;
            $sleepMs = $delay * (1 + $jitter);
            usleep($sleepMs * 1000);
            
            // Increase delay for next retry (exponential backoff)
            $delay *= 2;
        }
    }
}

// Example usage
try {
    $token = 'YOUR_API_TOKEN';
    $response = makeRequestWithRetry($token, 'POST', 'pay', [
        'service_code' => 'taura',
        'phone_number' => '0123456789',
        'amount' => 50
    ]);
    
    if ($response['success']) {
        echo "Payment successful!\n";
        echo "Reference: " . $response['data']['reference'] . "\n";
    } else {
        echo "Payment failed: " . $response['message'] . "\n";
    }
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . " (Code: " . $e->getCode() . ")\n";
}
```

### Comprehensive Error Handling (JavaScript Axios)

```javascript
// Using JavaScript/Node.js with Axios
const axios = require('axios');

// Create an API client with error handling
function createApiClient(token) {
  const client = axios.create({
    baseURL: 'https://api.xashpay.com/v1',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  });
  
  // Add response interceptor for error handling
  client.interceptors.response.use(
    response => response,
    error => {
      if (error.response) {
        // The request was made and the server responded with a status code
        // that falls out of the range of 2xx
        const status = error.response.status;
        const errorData = error.response.data;
        
        switch (status) {
          case 401:
            console.error('Authentication error:', errorData.message);
            // Handle authentication errors
            break;
          case 403:
            console.error('Permission error:', errorData.message);
            // Handle permission errors
            break;
          case 422:
            console.error('Validation error:', errorData.message);
            
            // Handle validation errors
            if (errorData.errors) {
              Object.entries(errorData.errors).forEach(([field, messages]) => {
                console.error(`${field}:`, messages.join(', '));
              });
            }
            break;
          case 429:
            console.error('Rate limit exceeded:', errorData.message);
            // Handle rate limiting
            break;
          default:
            console.error(`API error (${status}):`, errorData.message);
            // Handle other API errors
        }
      } else if (error.request) {
        // The request was made but no response was received
        console.error('No response received from server. Check your network connection.');
      } else {
        // Something happened in setting up the request
        console.error('Error setting up request:', error.message);
      }
      
      return Promise.reject(error);
    }
  );
  
  return client;
}

// Function to make API requests with retry for rate limiting
async function makeRequestWithRetry(client, method, endpoint, data = null, maxRetries = 3) {
  let retries = 0;
  let delay = 1000; // Start with 1 second delay
  
  while (true) {
    try {
      const config = {
        method,
        url: endpoint
      };
      
      if (data) {
        config.data = data;
      }
      
      const response = await client(config);
      return response.data;
    } catch (error) {
      retries++;
      
      // Don't retry if we've reached the maximum number of retries
      if (retries >= maxRetries) {
        throw error;
      }
      
      // Only retry for rate limiting errors
      if (!error.response || error.response.status !== 429) {
        throw error;
      }
      
      // Exponential backoff with jitter
      const jitter = Math.random();
      const sleepMs = delay * (1 + jitter);
      
      console.log(`Rate limited. Retrying after ${sleepMs}ms (attempt ${retries} of ${maxRetries})...`);
      
      await new Promise(resolve => setTimeout(resolve, sleepMs));
      
      // Increase delay for next retry (exponential backoff)
      delay *= 2;
    }
  }
}

// Example usage
async function makePayment(token) {
  const client = createApiClient(token);
  
  try {
    const response = await makeRequestWithRetry(client, 'post', '/pay', {
      service_code: 'taura',
      phone_number: '0123456789',
      amount: 50
    });
    
    if (response.success) {
      console.log('Payment successful!');
      console.log('Reference:', response.data.reference);
      return response.data;
    } else {
      console.log('Payment failed:', response.message);
      return null;
    }
  } catch (error) {
    console.error('Payment failed:', error.message);
    return null;
  }
}

// Usage
const token = 'YOUR_API_TOKEN';
makePayment(token)
  .then(result => {
    if (result) {
      // Process successful result
      console.log('Transaction completed!');
    }
  });
```

## Webhook Integration

### Processing Webhooks (PHP)

```php
<?php
// Webhook handler for XashPay notifications

// Get the raw POST data
$payload = file_get_contents('php://input');

// Verify webhook signature
$signature = $_SERVER['HTTP_X_XASHPAY_SIGNATURE'] ?? '';
$secret = 'YOUR_WEBHOOK_SECRET';

$computedSignature = hash_hmac('sha256', $payload, $secret);

if (!hash_equals($computedSignature, $signature)) {
    http_response_code(401);
    echo json_encode(['error' => 'Invalid signature']);
    exit;
}

// Parse the webhook payload
$data = json_decode($payload, true);

// Process different event types
switch ($data['event']) {
    case 'transaction.completed':
        // Handle completed transaction
        $reference = $data['data']['reference'];
        $amount = $data['data']['amount'];
        $status = $data['data']['status'];
        
        // Update your database or notify your system
        logTransaction($reference, $amount, $status);
        break;
        
    case 'voucher_action.status_changed':
        // Handle voucher action status change
        $actionId = $data['data']['action_id'];
        $status = $data['data']['status'];
        
        // Update your database or notify your system
        updateActionStatus($actionId, $status);
        break;
        
    case 'wallet.balance_updated':
        // Handle wallet balance update
        $newBalance = $data['data']['balance'];
        
        // Update your database or notify your system
        updateWalletBalance($newBalance);
        break;
        
    default:
        // Unknown event type
        http_response_code(400);
        echo json_encode(['error' => 'Unknown event type']);
        exit;
}

// Acknowledge receipt of the webhook
http_response_code(200);
echo json_encode(['success' => true]);

// Helper functions
function logTransaction($reference, $amount, $status) {
    // Implementation depends on your system
    // e.g., store in database, send notification, etc.
}

function updateActionStatus($actionId, $status) {
    // Implementation depends on your system
}

function updateWalletBalance($newBalance) {
    // Implementation depends on your system
}
```

### Processing Webhooks (Node.js)

```javascript
// Using Express.js for webhook handling
const express = require('express');
const crypto = require('crypto');
const bodyParser = require('body-parser');

const app = express();

// Use raw body parser for signature verification
app.use(bodyParser.json({
  verify: (req, res, buf) => {
    req.rawBody = buf;
  }
}));

// Webhook endpoint
app.post('/webhooks/xashpay', (req, res) => {
  // Verify webhook signature
  const signature = req.headers['x-xashpay-signature'];
  const secret = 'YOUR_WEBHOOK_SECRET';
  
  const computedSignature = crypto
    .createHmac('sha256', secret)
    .update(req.rawBody)
    .digest('hex');
  
  if (computedSignature !== signature) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Process different event types
  const { event, data } = req.body;
  
  switch (event) {
    case 'transaction.completed':
      // Handle completed transaction
      const { reference, amount, status } = data;
      
      // Update your database or notify your system
      logTransaction(reference, amount, status);
      break;
      
    case 'voucher_action.status_changed':
      // Handle voucher action status change
      const { action_id, status: actionStatus } = data;
      
      // Update your database or notify your system
      updateActionStatus(action_id, actionStatus);
      break;
      
    case 'wallet.balance_updated':
      // Handle wallet balance update
      const { balance } = data;
      
      // Update your database or notify your system
      updateWalletBalance(balance);
      break;
      
    default:
      // Unknown event type
      return res.status(400).json({ error: 'Unknown event type' });
  }
  
  // Acknowledge receipt of the webhook
  res.status(200).json({ success: true });
});

// Helper functions
function logTransaction(reference, amount, status) {
  // Implementation depends on your system
  console.log(`Transaction ${reference} for ${amount} is ${status}`);
}

function updateActionStatus(actionId, status) {
  // Implementation depends on your system
  console.log(`Action ${actionId} status changed to ${status}`);
}

function updateWalletBalance(newBalance) {
  // Implementation depends on your system
  console.log(`Wallet balance updated to ${newBalance}`);
}

// Start the server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Webhook server listening on port ${PORT}`);
});
```

These examples demonstrate how to integrate with the XashPay API system using standard HTTP clients and curl commands for various use cases.
