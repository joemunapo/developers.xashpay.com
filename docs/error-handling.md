# Error Handling

This document provides comprehensive guidance on handling errors in the XashPay API system.

## Error Response Format

All API errors follow a consistent format:

```json
{
  "success": false,
  "message": "Descriptive error message",
  "code": 422,
  "errors": {
    "field_name": [
      "Validation error message"
    ]
  }
}
```

The `errors` object is only included for validation errors (HTTP 422).

## HTTP Status Codes

XashPay API uses standard HTTP status codes to indicate the success or failure of an API request:

| Status Code | Description |
|-------------|-------------|
| 200 | OK - The request was successful |
| 400 | Bad Request - The request was malformed or contains invalid parameters |
| 401 | Unauthorized - Authentication failed or token is invalid |
| 403 | Forbidden - The authenticated user doesn't have permission for the requested operation |
| 404 | Not Found - The requested resource doesn't exist |
| 409 | Conflict - The request couldn't be completed due to a conflict (e.g., voucher already redeemed) |
| 422 | Unprocessable Entity - Validation failed or the request couldn't be processed |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error - Something went wrong on the server |

## Common Error Types

### Authentication Errors

```json
{
  "success": false,
  "message": "Invalid credentials",
  "code": 401
}
```

```json
{
  "success": false,
  "message": "Token has expired",
  "code": 401
}
```

### Permission Errors

```json
{
  "success": false,
  "message": "User account is not approved",
  "code": 403
}
```

```json
{
  "success": false,
  "message": "Service not enabled for this vendor",
  "code": 403
}
```

### Validation Errors

```json
{
  "success": false,
  "message": "The given data was invalid",
  "code": 422,
  "errors": {
    "phone_number": [
      "The phone number field is required",
      "The phone number must be 10 digits"
    ],
    "amount": [
      "The amount must be at least 5"
    ]
  }
}
```

### Resource Errors

```json
{
  "success": false,
  "message": "Service not found",
  "code": 404
}
```

```json
{
  "success": false,
  "message": "Voucher has already been redeemed",
  "code": 409
}
```

### Rate Limiting Errors

```json
{
  "success": false,
  "message": "Too many requests. Please try again in 60 seconds",
  "code": 429
}
```

## Service-Specific Errors

### Service Initialization Errors

```json
{
  "success": false,
  "message": "Service does not require initiation",
  "code": 400
}
```

```json
{
  "success": false,
  "message": "Account number not found",
  "code": 422
}
```

### Payment Processing Errors

```json
{
  "success": false,
  "message": "Insufficient funds",
  "code": 422
}
```

```json
{
  "success": false,
  "message": "Service provider failed to process request",
  "code": 422
}
```

### Voucher Errors

```json
{
  "success": false,
  "message": "Invalid voucher number or PIN",
  "code": 422
}
```

```json
{
  "success": false,
  "message": "Voucher has expired",
  "code": 422
}
```

## Error Handling Best Practices

### Client-Side Error Handling

#### PHP (Using cURL)

```php
// Using PHP cURL
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

try {
    $token = 'YOUR_API_TOKEN';
    $response = makeApiRequest($token, 'POST', 'pay', [
        'service_code' => 'taura',
        'phone_number' => '0123456789',
        'amount' => 50
    ]);
    
    // Process successful response
    if ($response['success']) {
        echo "Payment successful!";
    }
} catch (Exception $e) {
    // Handle different types of errors based on error code
    switch ($e->getCode()) {
        case 401:
            echo "Authentication error: " . $e->getMessage();
            // Handle authentication errors
            break;
        case 422:
            echo "Validation error: " . $e->getMessage();
            // Handle validation errors
            break;
        default:
            echo "Error: " . $e->getMessage();
            // Handle other errors
    }
}
```

#### PHP (Using Guzzle)

```php
// Using Guzzle HTTP client
require 'vendor/autoload.php';

use GuzzleHttp\Client;
use GuzzleHttp\Exception\ClientException;
use GuzzleHttp\Exception\ServerException;
use GuzzleHttp\Exception\ConnectException;

function makeApiRequest($token, $method, $endpoint, $data = null) {
    $client = new Client([
        'base_uri' => 'https://api.xashpay.com/v1/',
        'timeout' => 30,
        'headers' => [
            'Authorization' => 'Bearer ' . $token,
            'Accept' => 'application/json',
            'Content-Type' => 'application/json'
        ]
    ]);
    
    try {
        $options = [];
        if ($data) {
            $options['json'] = $data;
        }
        
        $response = $client->request($method, $endpoint, $options);
        return json_decode($response->getBody(), true);
    } catch (ClientException $e) {
        // 4xx errors
        $response = $e->getResponse();
        $responseData = json_decode($response->getBody(), true);
        $errorMessage = $responseData['message'] ?? 'Unknown error';
        
        switch ($response->getStatusCode()) {
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
                throw new Exception("API error (" . $response->getStatusCode() . "): " . $errorMessage, $response->getStatusCode());
        }
    } catch (ServerException $e) {
        // 5xx errors
        throw new Exception("Server error: " . $e->getMessage(), 500);
    } catch (ConnectException $e) {
        // Network errors
        throw new Exception("Network error: " . $e->getMessage(), 0);
    }
}

try {
    $token = 'YOUR_API_TOKEN';
    $response = makeApiRequest($token, 'POST', 'pay', [
        'service_code' => 'taura',
        'phone_number' => '0123456789',
        'amount' => 50
    ]);
    
    // Process successful response
    if ($response['success']) {
        echo "Payment successful!";
    }
} catch (Exception $e) {
    // Handle different types of errors based on error code
    switch ($e->getCode()) {
        case 401:
            echo "Authentication error: " . $e->getMessage();
            // Handle authentication errors
            break;
        case 422:
            echo "Validation error: " . $e->getMessage();
            // Handle validation errors
            break;
        default:
            echo "Error: " . $e->getMessage();
            // Handle other errors
    }
}
```

#### JavaScript (Using Axios)

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
          default:
            console.error(`API error (${status}):`, errorData.message);
            // Handle other API errors
        }
      } else if (error.request) {
        // The request was made but no response was received
        console.error('No response received from server');
        // Handle network errors
      } else {
        // Something happened in setting up the request
        console.error('Error setting up request:', error.message);
        // Handle unexpected errors
      }
      
      return Promise.reject(error);
    }
  );
  
  return client;
}

// Usage
const token = 'YOUR_API_TOKEN';
const client = createApiClient(token);

async function makePayment() {
  try {
    const response = await client.post('/pay', {
      service_code: 'taura',
      phone_number: '0123456789',
      amount: 50
    });
    
    // Process successful response
    console.log('Payment successful!');
  } catch (error) {
    // Error already handled by interceptor
    // Additional handling can be done here if needed
  }
}

makePayment();
```

### Retry Strategies

For transient errors (network issues, rate limiting), implement a retry strategy:

#### PHP (Using cURL)

```php
function makeRequestWithRetry($token, $method, $endpoint, $data = null, $maxRetries = 3, $initialDelay = 1000) {
    $retries = 0;
    $delay = $initialDelay;
    
    while (true) {
        try {
            // Make the API request
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
            
            // Check for rate limiting or server errors that should be retried
            if ($httpCode == 429 || ($httpCode >= 500 && $httpCode < 600)) {
                throw new Exception("Retryable error: " . ($responseData['message'] ?? 'Unknown error'), $httpCode);
            }
            
            // For other error codes, don't retry
            if ($httpCode >= 400) {
                $errorMessage = $responseData['message'] ?? 'Unknown error';
                throw new Exception("API error (" . $httpCode . "): " . $errorMessage, $httpCode);
            }
            
            return $responseData;
        } catch (Exception $e) {
            $retries++;
            
            // Don't retry if we've reached the maximum number of retries
            if ($retries >= $maxRetries) {
                throw $e;
            }
            
            // Only retry for rate limiting or server errors
            if ($e->getCode() != 429 && ($e->getCode() < 500 || $e->getCode() >= 600)) {
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

// Usage
try {
    $token = 'YOUR_API_TOKEN';
    $result = makeRequestWithRetry($token, 'POST', 'pay', [
        'service_code' => 'taura',
        'phone_number' => '0123456789',
        'amount' => 50
    ]);
    
    // Process successful response
    if ($result['success']) {
        echo "Payment successful!\n";
        echo "Reference: " . $result['data']['reference'] . "\n";
    }
} catch (Exception $e) {
    // Handle error after all retries have failed
    echo "Failed after multiple attempts: " . $e->getMessage();
}
```

#### JavaScript (Using Axios)

```javascript
async function makeRequestWithRetry(token, method, endpoint, data = null, maxRetries = 3, initialDelay = 1000) {
  let retries = 0;
  let delay = initialDelay;
  
  const client = axios.create({
    baseURL: 'https://api.xashpay.com/v1',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  });
  
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
      
      // Only retry for rate limiting or server errors
      const isRateLimit = error.response && error.response.status === 429;
      const isServerError = error.response && error.response.status >= 500 && error.response.status < 600;
      const isNetworkError = !error.response && error.request;
      
      if (!isRateLimit && !isServerError && !isNetworkError) {
        throw error;
      }
      
      // Exponential backoff with jitter
      const jitter = Math.random();
      const sleepMs = delay * (1 + jitter);
      
      console.log(`Retrying after ${sleepMs}ms (attempt ${retries} of ${maxRetries})...`);
      
      await new Promise(resolve => setTimeout(resolve, sleepMs));
      
      // Increase delay for next retry (exponential backoff)
      delay *= 2;
    }
  }
}

// Usage
async function makePayment() {
  try {
    const token = 'YOUR_API_TOKEN';
    const result = await makeRequestWithRetry(token, 'post', '/pay', {
      service_code: 'taura',
      phone_number: '0123456789',
      amount: 50
    });
    
    // Process successful response
    if (result.success) {
      console.log('Payment successful!');
      console.log('Reference:', result.data.reference);
    }
  } catch (error) {
    // Handle error after all retries have failed
    console.error('Failed after multiple attempts:', error.response?.data?.message || error.message);
  }
}

makePayment();
```

## Logging Errors

Always log errors for debugging and monitoring:

### PHP

```php
// Using PHP cURL with error logging
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
        
        // Log the error
        error_log("API Request Failed: cURL Error: " . $error);
        
        throw new Exception("cURL Error: " . $error);
    }
    
    curl_close($ch);
    
    $responseData = json_decode($response, true);
    
    if ($httpCode >= 400) {
        $errorMessage = isset($responseData['message']) ? $responseData['message'] : 'Unknown error';
        
        // Log the error with context
        error_log("API Request Failed: " . $method . " " . $url . " - Status: " . $httpCode . " - Message: " . $errorMessage);
        
        throw new Exception("API error (" . $httpCode . "): " . $errorMessage, $httpCode);
    }
    
    return $responseData;
}

try {
    $token = 'YOUR_API_TOKEN';
    $paymentData = [
        'service_code' => 'taura',
        'phone_number' => '0123456789',
        'amount' => 50
    ];
    
    $response = makeApiRequest($token, 'POST', 'pay', $paymentData);
    
    // Log successful response
    error_log("Payment successful - Reference: " . $response['data']['reference'] . " - Amount: " . $response['data']['amount']);
    
    // Process successful response
} catch (Exception $e) {
    // Log error with context
    error_log("Payment failed - Error: " . $e->getMessage() . " - Code: " . $e->getCode() . " - Data: " . json_encode($paymentData));
    
    // Handle the error
    echo "Error: " . $e->getMessage();
}
```

### JavaScript

```javascript
// Using JavaScript/Node.js with Axios and error logging
const axios = require('axios');

async function makeApiRequest(token, method, endpoint, data = null) {
  try {
    const client = axios.create({
      baseURL: 'https://api.xashpay.com/v1',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    });
    
    const config = {
      method,
      url: endpoint
    };
    
    if (data) {
      config.data = data;
    }
    
    const response = await client(config);
    
    // Log successful response
    console.log(`API Request Successful: ${method} ${endpoint}`, {
      reference: response.data.data?.reference,
      amount: response.data.data?.amount
    });
    
    return response.data;
  } catch (error) {
    // Log error with context
    console.error(`API Request Failed: ${method} ${endpoint}`, {
      error: error.message,
      status: error.response?.status,
      data: error.response?.data,
      requestData: data
    });
    
    throw error;
  }
}

// Usage
async function makePayment() {
  try {
    const token = 'YOUR_API_TOKEN';
    const paymentData = {
      service_code: 'taura',
      phone_number: '0123456789',
      amount: 50
    };
    
    const response = await makeApiRequest(token, 'post', '/pay', paymentData);
    
    // Process successful response
    console.log('Payment successful!');
    console.log('Reference:', response.data.reference);
  } catch (error) {
    // Handle the error
    console.error('Payment failed:', error.response?.data?.message || error.message);
  }
}

makePayment();
```

## User-Friendly Error Messages

Map API error messages to user-friendly messages:

### PHP

```php
function getUserFriendlyErrorMessage($errorMessage) {
    $errorMessages = [
        'Invalid credentials' => 'Your login details are incorrect. Please check and try again.',
        'Token has expired' => 'Your session has expired. Please log in again.',
        'Insufficient funds' => 'You don\'t have enough funds to complete this transaction. Please top up your wallet.',
        'Voucher has already been redeemed' => 'This voucher has already been used and cannot be redeemed again.',
        'Invalid voucher number or PIN' => 'The voucher details you entered are incorrect. Please check and try again.',
        'Service provider failed to process request' => 'We couldn\'t process your request at this time. Please try again later.'
    ];
    
    return $errorMessages[$errorMessage] ?? 'An error occurred. Please try again or contact support.';
}

try {
    $token = 'YOUR_API_TOKEN';
    $response = makeApiRequest($token, 'POST', 'pay', [
        'service_code' => 'taura',
        'phone_number' => '0123456789',
        'amount' => 50
    ]);
    
    // Process successful response
} catch (Exception $e) {
    // Get the original error message
    $errorMessage = $e->getMessage();
    
    // Extract just the message part if it's in the format "API error (code): message"
    if (preg_match('/API error \(\d+\): (.+)/', $errorMessage, $matches)) {
        $errorMessage = $matches[1];
    }
    
    $userMessage = getUserFriendlyErrorMessage($errorMessage);
    echo $userMessage;
    
    // Log the original error and the user-friendly message
    error_log("Payment failed - Error: " . $e->getMessage() . " - User Message: " . $userMessage);
}
```

### JavaScript

```javascript
function getUserFriendlyErrorMessage(error) {
  const errorMessages = {
    'Invalid credentials': 'Your login details are incorrect. Please check and try again.',
    'Token has expired': 'Your session has expired. Please log in again.',
    'Insufficient funds': 'You don\'t have enough funds to complete this transaction. Please top up your wallet.',
    'Voucher has already been redeemed': 'This voucher has already been used and cannot be redeemed again.',
    'Invalid voucher number or PIN': 'The voucher details you entered are incorrect. Please check and try again.',
    'Service provider failed to process request': 'We couldn\'t process your request at this time. Please try again later.'
  };
  
  const message = error.response?.data?.message || error.message;
  
  return errorMessages[message] || 'An error occurred. Please try again or contact support.';
}

async function makePayment() {
  try {
    const token = 'YOUR_API_TOKEN';
    const response = await axios.post('https://api.xashpay.com/v1/pay', {
      service_code: 'taura',
      phone_number: '0123456789',
      amount: 50
    }, {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    });
    
    // Process successful response
    console.log('Payment successful!');
  } catch (error) {
    const userMessage = getUserFriendlyErrorMessage(error);
    console.log('User message:', userMessage);
    
    // Display to user (e.g., in UI)
    showErrorToUser(userMessage);
    
    // Log the original error and the user-friendly message
    console.error('Payment failed', {
      error: error.message,
      response: error.response?.data,
      user_message: userMessage
    });
  }
}

// Helper function to display error to user (implementation depends on your UI framework)
function showErrorToUser(message) {
  // Example implementation
  alert(message);
}

makePayment();
```

## Monitoring and Alerting

Set up monitoring and alerting for API errors:

1. **Log Aggregation**: Use a log aggregation service to collect and analyze API errors
2. **Error Rate Monitoring**: Set up alerts for unusual error rates
3. **Critical Error Alerts**: Configure immediate notifications for critical errors
4. **Error Dashboards**: Create dashboards to visualize error trends

## Troubleshooting Common Errors

### Authentication Issues

- **Invalid credentials**: Check that you're using the correct email and password
- **Token has expired**: Refresh your token or log in again
- **User account is not approved**: Contact XashPay support to get your account approved

### Payment Processing Issues

- **Insufficient funds**: Top up your wallet balance
- **Service provider failed to process request**: Check the service status and try again later
- **Invalid service parameters**: Verify that all required parameters are provided and valid

### Voucher Issues

- **Invalid voucher number or PIN**: Double-check the voucher details
- **Voucher has already been redeemed**: Verify that the voucher hasn't been used before
- **Voucher has expired**: Check the voucher's expiration date

### Rate Limiting Issues

- **Too many requests**: Implement proper rate limiting in your application
- **Implement exponential backoff**: Add delays between retries that increase with each attempt

## Support

If you encounter persistent errors or need assistance, contact our support team:

- Email: api-support@xashpay.com
- Support Portal: https://support.xashpay.com
- API Status: https://status.xashpay.com
