# Troubleshooting

This document provides solutions for common issues you might encounter when using the XashPay API system.

## Connection Issues

### API Unreachable

**Symptoms:**
- Timeout errors
- Connection refused errors
- Network-related exceptions

**Solutions:**
1. Check your internet connection
2. Verify the API endpoint URL is correct
3. Check if the XashPay API status page reports any outages
4. Try accessing the API from a different network to rule out local network issues

### SSL/TLS Errors

**Symptoms:**
- SSL certificate errors
- Handshake failures
- "Unable to verify certificate" errors

**Solutions:**
1. Ensure your system has up-to-date CA certificates
2. Check if your client is using an outdated SSL/TLS version
3. Verify that your system time is accurate (certificate validation depends on this)

```php
// PHP cURL: Force TLS 1.2 or higher
$ch = curl_init('https://api.xashpay.com/v1/endpoint');
curl_setopt($ch, CURLOPT_SSLVERSION, CURL_SSLVERSION_TLSv1_2);
// Other curl options...
curl_exec($ch);
```

```php
// PHP Guzzle: Force TLS 1.2 or higher
$client = new GuzzleHttp\Client([
    'base_uri' => 'https://api.xashpay.com/v1/',
    'curl' => [
        CURLOPT_SSLVERSION => CURL_SSLVERSION_TLSv1_2
    ]
]);
```

```javascript
// Node.js: Force TLS 1.2 or higher
const https = require('https');
const axios = require('axios');

const agent = new https.Agent({
  secureProtocol: 'TLSv1_2_method'
});

const client = axios.create({
  baseURL: 'https://api.xashpay.com/v1',
  httpsAgent: agent,
  headers: {
    'Authorization': 'Bearer YOUR_API_TOKEN'
  }
});
```

## Authentication Issues

### Invalid Credentials

**Symptoms:**
- "Invalid credentials" error message
- HTTP 401 Unauthorized responses

**Solutions:**
1. Double-check your email and password
2. Ensure there are no typos or extra spaces
3. Reset your password if necessary
4. Contact support if you continue to have issues

### Token Expired

**Symptoms:**
- "Token has expired" error message
- HTTP 401 Unauthorized responses after previously successful authentication

**Solutions:**
1. Implement token refresh logic in your application
2. Re-authenticate to obtain a new token
3. Check if your system clock is synchronized (JWT validation is time-sensitive)

```php
// PHP cURL: Token refresh example
function refreshToken($refreshToken) {
    $ch = curl_init('https://api.xashpay.com/v1/refresh');
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode(['refresh_token' => $refreshToken]));
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Content-Type: application/json',
        'Accept: application/json'
    ]);
    
    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    if ($httpCode == 200) {
        $data = json_decode($response, true);
        if ($data['success']) {
            return $data['data']['token'];
        }
    }
    
    return null;
}

function makePaymentWithTokenRefresh($token, $refreshToken, $paymentData) {
    $ch = curl_init('https://api.xashpay.com/v1/pay');
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($paymentData));
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Content-Type: application/json',
        'Accept: application/json',
        'Authorization: Bearer ' . $token
    ]);
    
    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    if ($httpCode == 401) {
        $responseData = json_decode($response, true);
        if (isset($responseData['message']) && $responseData['message'] === 'Token has expired') {
            // Refresh token
            $newToken = refreshToken($refreshToken);
            if ($newToken) {
                // Retry with new token
                return makePaymentWithTokenRefresh($newToken, $refreshToken, $paymentData);
            }
        }
    }
    
    return json_decode($response, true);
}

// Usage
$token = 'YOUR_API_TOKEN';
$refreshToken = 'YOUR_REFRESH_TOKEN';
$paymentData = [
    'service_code' => 'taura',
    'phone_number' => '0123456789',
    'amount' => 50
];

$result = makePaymentWithTokenRefresh($token, $refreshToken, $paymentData);
```

```javascript
// JavaScript Axios: Token refresh example
async function makeAuthenticatedRequest(token, refreshToken) {
  const api = axios.create({
    baseURL: 'https://api.xashpay.com/v1',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  });
  
  try {
    const response = await api.post('/pay', {
      service_code: 'taura',
      phone_number: '0123456789',
      amount: 50
    });
    return response.data;
  } catch (error) {
    if (error.response?.status === 401 && 
        error.response?.data?.message === 'Token has expired') {
      // Refresh token
      try {
        const refreshResponse = await axios.post('https://api.xashpay.com/v1/refresh', {
          refresh_token: refreshToken
        });
        
        if (refreshResponse.data.success) {
          const newToken = refreshResponse.data.data.token;
          
          // Retry the original request with new token
          const retryResponse = await axios.post('https://api.xashpay.com/v1/pay', {
            service_code: 'taura',
            phone_number: '0123456789',
            amount: 50
          }, {
            headers: {
              'Authorization': `Bearer ${newToken}`,
              'Content-Type': 'application/json'
            }
          });
          
          return retryResponse.data;
        }
      } catch (refreshError) {
        console.error('Token refresh failed:', refreshError.message);
        throw refreshError;
      }
    }
    throw error;
  }
}
```

### User Not Approved

**Symptoms:**
- "User account is not approved" error message
- HTTP 403 Forbidden responses

**Solutions:**
1. Contact XashPay support to get your account approved
2. Check your email for any verification or approval instructions
3. Complete any required verification steps in your XashPay dashboard

## Service-Related Issues

### Service Not Found

**Symptoms:**
- "Service not found" error message
- HTTP 404 Not Found responses when trying to use a service

**Solutions:**
1. Verify the service code is correct
2. Check if the service is available in your account using the List Services endpoint
3. Contact XashPay support to request access to the service

### Service Provider Unavailable

**Symptoms:**
- "Service provider failed to process request" error message
- HTTP 422 Unprocessable Entity responses

**Solutions:**
1. Try again later as the service provider might be experiencing temporary issues
2. Check the XashPay status page for any reported service outages
3. Implement retry logic with exponential backoff for transient errors

```php
// PHP cURL: Retry with exponential backoff
function retryWithBackoff($callback, $maxRetries = 3) {
    $retries = 0;
    
    while (true) {
        try {
            $result = $callback();
            $responseData = json_decode($result, true);
            
            if (!$responseData['success'] && 
                isset($responseData['message']) && 
                $responseData['message'] === 'Service provider failed to process request') {
                throw new Exception('Service provider failed to process request');
            }
            
            return $result;
        } catch (Exception $e) {
            if ($e->getMessage() !== 'Service provider failed to process request' ||
                $retries >= $maxRetries) {
                throw $e;
            }
            
            $retries++;
            $delay = pow(2, $retries) * 1000; // Exponential backoff: 2s, 4s, 8s, etc.
            usleep($delay * 1000);
        }
    }
}

// Usage
function makePaymentRequest($token) {
    $ch = curl_init('https://api.xashpay.com/v1/pay');
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode([
        'service_code' => 'taura',
        'phone_number' => '0123456789',
        'amount' => 50
    ]));
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Content-Type: application/json',
        'Accept: application/json',
        'Authorization: Bearer ' . $token
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    return $response;
}

$token = 'YOUR_API_TOKEN';
$result = retryWithBackoff(function() use ($token) {
    return makePaymentRequest($token);
});

$responseData = json_decode($result, true);
if ($responseData['success']) {
    echo "Payment successful!";
}
```

```javascript
// JavaScript Axios: Retry with exponential backoff
async function retryWithBackoff(callback, maxRetries = 3) {
  let retries = 0;
  
  while (true) {
    try {
      const result = await callback();
      
      if (!result.data.success && 
          result.data.message === 'Service provider failed to process request') {
        throw new Error('Service provider failed to process request');
      }
      
      return result;
    } catch (error) {
      const isProviderError = 
        error.message === 'Service provider failed to process request' ||
        error.response?.data?.message === 'Service provider failed to process request';
        
      if (!isProviderError || retries >= maxRetries) {
        throw error;
      }
      
      retries++;
      const delay = Math.pow(2, retries) * 1000; // Exponential backoff: 2s, 4s, 8s, etc.
      console.log(`Retrying after ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Usage
async function makePayment(token) {
  const result = await retryWithBackoff(async () => {
    return await axios.post('https://api.xashpay.com/v1/pay', {
      service_code: 'taura',
      phone_number: '0123456789',
      amount: 50
    }, {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    });
  });
  
  console.log('Payment successful!');
  return result.data;
}

const token = 'YOUR_API_TOKEN';
makePayment(token).catch(error => console.error('Payment failed:', error.message));
```

### Insufficient Funds

**Symptoms:**
- "Insufficient funds" error message
- HTTP 422 Unprocessable Entity responses

**Solutions:**
1. Check your wallet balance using the Get Wallet Balance endpoint
2. Add funds to your wallet
3. Verify the transaction amount is correct
4. Implement balance checking before attempting transactions

## Voucher-Related Issues

### Invalid Voucher

**Symptoms:**
- "Invalid voucher number or PIN" error message
- HTTP 422 Unprocessable Entity responses

**Solutions:**
1. Double-check the voucher number and PIN for typos
2. Verify the voucher is from a supported provider
3. Always validate vouchers before attempting redemption

### Voucher Already Redeemed

**Symptoms:**
- "Voucher has already been redeemed" error message
- HTTP 409 Conflict responses

**Solutions:**
1. Verify the voucher hasn't been used before
2. Implement voucher validation before redemption
3. Keep records of previously redeemed vouchers to avoid duplicate attempts

### Voucher Action Failed

**Symptoms:**
- Voucher redemption succeeds but the action fails
- Action status shows as "failed" when checked

**Solutions:**
1. Check the action logs for specific error details
2. Verify the action parameters are correct
3. Use the Retry Failed Action endpoint to attempt the action again
4. Ensure the voucher value is sufficient for the requested action

```php
// PHP cURL: Check action status and retry if failed
function redeemVoucherWithAction($token, $voucherData) {
    $ch = curl_init('https://api.xashpay.com/v1/vouchers/redeem');
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($voucherData));
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Content-Type: application/json',
        'Accept: application/json',
        'Authorization: Bearer ' . $token
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    return json_decode($response, true);
}

function checkActionStatus($token, $actionId) {
    $ch = curl_init('https://api.xashpay.com/v1/vouchers/actions/' . $actionId);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Accept: application/json',
        'Authorization: Bearer ' . $token
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    return json_decode($response, true);
}

function retryAction($token, $actionId) {
    $ch = curl_init('https://api.xashpay.com/v1/vouchers/actions/' . $actionId . '/retry');
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Accept: application/json',
        'Authorization: Bearer ' . $token
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    return json_decode($response, true);
}

// Usage
$token = 'YOUR_API_TOKEN';
$voucherData = [
    'provider_code' => 'OTT',
    'voucher_number' => '123456789',
    'voucher_pin' => '1234',
    'voucher_action' => [
        'service_code' => 'taura',
        'phone_number' => '0123456789'
    ]
];

$redeemResponse = redeemVoucherWithAction($token, $voucherData);

if ($redeemResponse['success']) {
    $actionId = $redeemResponse['data']['action_id'];
    
    // Wait a moment for processing
    sleep(5);
    
    // Check action status
    $statusResponse = checkActionStatus($token, $actionId);
    
    if ($statusResponse['success'] && $statusResponse['data']['status'] === 'failed') {
        // Retry the action
        $retryResponse = retryAction($token, $actionId);
        
        if ($retryResponse['success']) {
            echo "Action retry initiated successfully";
        }
    }
}
```

```javascript
// JavaScript Fetch API: Check action status and retry if failed
async function redeemVoucherWithAction(token, voucherData) {
  const response = await fetch('https://api.xashpay.com/v1/vouchers/redeem', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
      'Authorization': `Bearer ${token}`
    },
    body: JSON.stringify(voucherData)
  });
  
  return await response.json();
}

async function checkActionStatus(token, actionId) {
  const response = await fetch(`https://api.xashpay.com/v1/vouchers/actions/${actionId}`, {
    method: 'GET',
    headers: {
      'Accept': 'application/json',
      'Authorization': `Bearer ${token}`
    }
  });
  
  return await response.json();
}

async function retryAction(token, actionId) {
  const response = await fetch(`https://api.xashpay.com/v1/vouchers/actions/${actionId}/retry`, {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Authorization': `Bearer ${token}`
    }
  });
  
  return await response.json();
}

// Usage
async function redeemAndMonitorAction(token) {
  const voucherData = {
    provider_code: 'OTT',
    voucher_number: '123456789',
    voucher_pin: '1234',
    voucher_action: {
      service_code: 'taura',
      phone_number: '0123456789'
    }
  };
  
  const redeemResponse = await redeemVoucherWithAction(token, voucherData);
  
  if (redeemResponse.success) {
    const actionId = redeemResponse.data.action_id;
    
    // Wait a moment for processing
    await new Promise(resolve => setTimeout(resolve, 5000));
    
    // Check action status
    const statusResponse = await checkActionStatus(token, actionId);
    
    if (statusResponse.success && statusResponse.data.status === 'failed') {
      // Retry the action
      const retryResponse = await retryAction(token, actionId);
      
      if (retryResponse.success) {
        console.log('Action retry initiated successfully');
      }
    }
  }
}

const token = 'YOUR_API_TOKEN';
redeemAndMonitorAction(token).catch(error => console.error('Error:', error));
```

## Rate Limiting Issues

### Too Many Requests

**Symptoms:**
- "Too many requests" error message
- HTTP 429 Too Many Requests responses

**Solutions:**
1. Implement rate limiting in your application
2. Add exponential backoff for retries
3. Distribute requests over time
4. Consider batching multiple operations when possible

```php
// PHP cURL: Handle rate limiting
function makeApiRequestWithRateLimitHandling($token, $url, $method = 'GET', $data = null) {
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);
    
    $headers = [
        'Accept: application/json',
        'Authorization: Bearer ' . $token
    ];
    
    if ($data && ($method === 'POST' || $method === 'PUT')) {
        $headers[] = 'Content-Type: application/json';
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
    }
    
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    curl_setopt($ch, CURLOPT_HEADER, true);
    
    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    $headerSize = curl_getinfo($ch, CURLINFO_HEADER_SIZE);
    
    $responseHeaders = substr($response, 0, $headerSize);
    $responseBody = substr($response, $headerSize);
    
    curl_close($ch);
    
    if ($httpCode === 429) {
        // Extract Retry-After header if available
        preg_match('/Retry-After: (\d+)/i', $responseHeaders, $matches);
        $retryAfter = isset($matches[1]) ? (int)$matches[1] : 60;
        
        echo "Rate limited. Waiting for {$retryAfter} seconds before retrying...\n";
        sleep($retryAfter);
        
        // Retry the request
        return makeApiRequestWithRateLimitHandling($token, $url, $method, $data);
    }
    
    return json_decode($responseBody, true);
}

// Usage
$token = 'YOUR_API_TOKEN';
$result = makeApiRequestWithRateLimitHandling(
    $token,
    'https://api.xashpay.com/v1/pay',
    'POST',
    [
        'service_code' => 'taura',
        'phone_number' => '0123456789',
        'amount' => 50
    ]
);

if ($result['success']) {
    echo "Payment successful!";
}
```

```javascript
// JavaScript Axios: Handle rate limiting
async function makeApiRequestWithRateLimitHandling(token, method, endpoint, data = null) {
  try {
    const config = {
      method,
      url: `https://api.xashpay.com/v1/${endpoint}`,
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    };
    
    if (data) {
      config.data = data;
    }
    
    const response = await axios(config);
    return response.data;
  } catch (error) {
    if (error.response && error.response.status === 429) {
      // Get retry-after header if available
      const retryAfter = parseInt(error.response.headers['retry-after']) || 60;
      
      console.log(`Rate limited. Retrying after ${retryAfter} seconds...`);
      
      // Wait for the specified time
      await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
      
      // Retry the request
      return makeApiRequestWithRateLimitHandling(token, method, endpoint, data);
    }
    
    throw error;
  }
}

// Usage
async function makePayment(token) {
  try {
    const result = await makeApiRequestWithRateLimitHandling(
      token,
      'post',
      'pay',
      {
        service_code: 'taura',
        phone_number: '0123456789',
        amount: 50
      }
    );
    
    if (result.success) {
      console.log('Payment successful!');
    }
    
    return result;
  } catch (error) {
    console.error('Payment failed:', error.message);
    throw error;
  }
}

const token = 'YOUR_API_TOKEN';
makePayment(token);
```

## Webhook Issues

### Webhook Not Received

**Symptoms:**
- Your webhook endpoint is not receiving notifications
- Events are not being processed

**Solutions:**
1. Verify your webhook URL is correctly registered in the XashPay dashboard
2. Ensure your webhook endpoint is publicly accessible
3. Check your server logs for any errors in processing webhook requests
4. Verify your server is not blocking requests from XashPay's IP addresses

### Invalid Webhook Signature

**Symptoms:**
- Webhook signature verification fails
- Security-related errors when processing webhooks

**Solutions:**
1. Ensure you're using the correct webhook secret
2. Verify your signature calculation matches XashPay's method
3. Check for any modifications to the payload during transmission

```php
// PHP: Correct webhook signature verification
$payload = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_XASHPAY_SIGNATURE'] ?? '';
$secret = 'YOUR_WEBHOOK_SECRET';

$computedSignature = hash_hmac('sha256', $payload, $secret);

if (!hash_equals($computedSignature, $signature)) {
    http_response_code(401);
    echo json_encode(['error' => 'Invalid signature']);
    exit;
}
```

```javascript
// Node.js: Correct webhook signature verification
const crypto = require('crypto');

// In your Express.js route handler
app.post('/webhooks/xashpay', (req, res) => {
  const signature = req.headers['x-xashpay-signature'];
  const secret = 'YOUR_WEBHOOK_SECRET';
  
  const computedSignature = crypto
    .createHmac('sha256', req.rawBody)
    .update(secret)
    .digest('hex');
  
  if (computedSignature !== signature) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Process the webhook
  // ...
});
```

## Memory Issues

**Symptoms:**
- Out of memory errors
- Performance degradation
- Application crashes

**Solutions:**
1. Optimize your code to release resources after use
2. Increase memory limits if necessary
3. Process large datasets in smaller chunks

```php
// PHP: Increase memory limit
ini_set('memory_limit', '256M');
```

## Getting Help

If you've tried the solutions above and are still experiencing issues, contact XashPay support:

- Email: api-support@xashpay.com
- Support Portal: https://support.xashpay.com
- API Status: https://status.xashpay.com

When contacting support, please provide:

1. Your account information
2. Detailed description of the issue
3. Steps to reproduce the problem
4. Error messages and codes
5. Request and response data (with sensitive information redacted)
6. Timestamps of when the issue occurred
