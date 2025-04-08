# Deployment

This document provides comprehensive instructions for deploying and integrating the XashPay API system into your application.

## System Requirements

To properly implement the XashPay API system, your server should meet the following requirements:

### Server Requirements

- PHP 8.0 or higher (for PHP implementations)
- Node.js 14.0 or higher (for JavaScript implementations)
- Reliable internet connection to communicate with the XashPay API
- SSL/TLS encryption for secure API communication
- Sufficient memory and processing power to handle asynchronous requests

### Security Requirements

- Firewall protection for your servers
- HTTPS for all client-server communications
- Secure storage for API credentials
- Regular security updates for all system components

## Integration Options

### API Client Libraries

We provide official client libraries for easy integration:

#### PHP

```bash
composer require xashpay/api-client
```

#### JavaScript/Node.js

```bash
npm install xashpay-api-client
# or
yarn add xashpay-api-client
```

### Direct API Integration

If you prefer to integrate directly with our API:

1. Use a reliable HTTP client library
2. Implement proper error handling and retries
3. Follow our [API Reference](api-reference.md) documentation

## Environment Setup

### Development Environment

For testing and development, use our sandbox environment:

```php
// PHP
$client = new XashPay\ApiClient\Client([
    'api_key' => 'YOUR_SANDBOX_API_KEY',
    'environment' => 'sandbox'
]);
```

```javascript
// JavaScript
const xashpay = require('xashpay-api-client');
const client = new xashpay.Client({
  apiKey: 'YOUR_SANDBOX_API_KEY',
  environment: 'sandbox'
});
```

### Production Environment

When you're ready to go live, switch to our production environment:

```php
// PHP
$client = new XashPay\ApiClient\Client([
    'api_key' => 'YOUR_PRODUCTION_API_KEY',
    'environment' => 'production'
]);
```

```javascript
// JavaScript
const xashpay = require('xashpay-api-client');
const client = new xashpay.Client({
  apiKey: 'YOUR_PRODUCTION_API_KEY',
  environment: 'production'
});
```

## Integration Architecture

### Recommended Architecture

For optimal performance and reliability, we recommend the following architecture:

1. **API Client Layer**: Handles communication with XashPay API
2. **Service Layer**: Business logic for your application
3. **Queue System**: For handling asynchronous operations
4. **Webhook Handler**: For receiving real-time updates
5. **Database**: For storing transaction records and user data

### Queue Worker Setup

For optimal performance with asynchronous operations like voucher actions, we recommend setting up a queue worker.

#### Laravel Queue Worker

If you're using Laravel, configure your queue worker:

1. Set up the database queue driver in your `.env` file:
   ```
   QUEUE_CONNECTION=database
   ```

2. Run the queue worker:
   ```bash
   php artisan queue:work --tries=3 --timeout=90
   ```

3. For production, set up a supervisor configuration:
   ```
   [program:laravel-queue]
   process_name=%(program_name)s_%(process_num)02d
   command=php /path/to/your/project/artisan queue:work --tries=3 --timeout=90
   autostart=true
   autorestart=true
   user=www-data
   numprocs=2
   redirect_stderr=true
   stdout_logfile=/path/to/your/project/storage/logs/queue.log
   ```

#### Node.js Queue Worker

For Node.js applications, you can use libraries like Bull or Agenda:

```javascript
// Using Bull
const Queue = require('bull');
const actionQueue = new Queue('xashpay-actions');

// Process jobs
actionQueue.process(async (job) => {
  const { actionId } = job.data;
  // Process the action
  await processAction(actionId);
});

// Add a job to the queue
actionQueue.add({ actionId: 'action-123' });
```

## Webhook Integration

XashPay can send real-time updates to your system via webhooks. This is especially useful for asynchronous operations like voucher actions.

### Setting Up Webhooks

1. Create a webhook endpoint in your application
2. Register the webhook URL in your XashPay dashboard
3. Implement signature verification to ensure webhook authenticity
4. Process webhook events and update your system accordingly

### Webhook Verification

Always verify webhook signatures to ensure they come from XashPay:

```php
// PHP
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
// Node.js
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

### Webhook Events

XashPay sends the following webhook events:

| Event | Description |
|-------|-------------|
| `transaction.completed` | A transaction has been completed |
| `transaction.failed` | A transaction has failed |
| `voucher_action.status_changed` | A voucher action's status has changed |
| `wallet.balance_updated` | Your wallet balance has been updated |
| `commission.released` | Commission has been released to your available balance |

## Security Considerations

### API Key Security

- Store API keys securely using environment variables or a secrets manager
- Never hardcode API keys in your source code
- Rotate API keys periodically
- Use different API keys for different environments

### HTTPS

Always use HTTPS for all API communications. Never send API requests over unencrypted HTTP.

### Input Validation

Validate all user inputs before sending them to the API:

```php
// PHP
function validatePhoneNumber($phoneNumber) {
    if (!preg_match('/^\d{10}$/', $phoneNumber)) {
        throw new \InvalidArgumentException('Phone number must be 10 digits');
    }
    return $phoneNumber;
}
```

```javascript
// JavaScript
function validatePhoneNumber(phoneNumber) {
  if (!/^\d{10}$/.test(phoneNumber)) {
    throw new Error('Phone number must be 10 digits');
  }
  return phoneNumber;
}
```

### Rate Limiting

Implement rate limiting in your application to prevent abuse:

```php
// PHP example using Redis
function checkRateLimit($userId, $limit = 100, $period = 60) {
    $redis = new \Redis();
    $redis->connect('127.0.0.1', 6379);
    
    $key = "rate_limit:{$userId}";
    $current = $redis->get($key);
    
    if (!$current) {
        $redis->setex($key, $period, 1);
        return true;
    }
    
    if ($current >= $limit) {
        return false;
    }
    
    $redis->incr($key);
    return true;
}
```

```javascript
// Node.js example using Redis
const Redis = require('ioredis');
const redis = new Redis();

async function checkRateLimit(userId, limit = 100, period = 60) {
  const key = `rate_limit:${userId}`;
  const current = await redis.get(key);
  
  if (!current) {
    await redis.setex(key, period, 1);
    return true;
  }
  
  if (parseInt(current) >= limit) {
    return false;
  }
  
  await redis.incr(key);
  return true;
}
```

## Monitoring and Logging

### Logging

Implement comprehensive logging for all API interactions:

```php
// PHP
try {
    $response = $client->services->pay([
        'service_code' => 'taura',
        'phone_number' => '0123456789',
        'amount' => 50
    ]);
    
    // Log successful response
    logger()->info('Payment successful', [
        'reference' => $response->data->reference,
        'amount' => $response->data->amount,
        'service' => 'taura'
    ]);
} catch (\Exception $e) {
    // Log error
    logger()->error('Payment failed', [
        'error' => $e->getMessage(),
        'code' => $e->getCode(),
        'service' => 'taura'
    ]);
}
```

```javascript
// JavaScript
try {
  const response = await client.services.pay({
    service_code: 'taura',
    phone_number: '0123456789',
    amount: 50
  });
  
  // Log successful response
  console.log('Payment successful', {
    reference: response.data.reference,
    amount: response.data.amount,
    service: 'taura'
  });
} catch (error) {
  // Log error
  console.error('Payment failed', {
    error: error.message,
    code: error.code,
    service: 'taura'
  });
}
```

### Monitoring

Monitor your API usage and system performance:

1. Track API response times
2. Monitor error rates
3. Set up alerts for unusual activity
4. Track transaction success/failure rates
5. Monitor wallet balance to prevent insufficient funds errors

## Deployment Strategies

### Phased Rollout

For a smooth integration, we recommend a phased rollout:

1. **Development Phase**: Integrate with sandbox environment
2. **Internal Testing**: Test with real data in a controlled environment
3. **Beta Testing**: Limited rollout to select users
4. **Full Production**: Complete rollout to all users

### Continuous Integration/Continuous Deployment (CI/CD)

Implement CI/CD for your XashPay integration:

1. Automated testing for API interactions
2. Deployment pipelines for different environments
3. Automated monitoring and alerting
4. Regular security scans

## Going Live Checklist

Before deploying to production, ensure you have:

- [x] Switched to production API keys
- [x] Implemented proper error handling
- [x] Set up queue workers for asynchronous operations
- [x] Implemented comprehensive logging
- [x] Tested all service scenarios
- [x] Implemented proper security measures
- [x] Set up monitoring and alerting
- [x] Registered webhook endpoints
- [x] Implemented webhook signature verification
- [x] Documented internal processes for your team

## Support

If you encounter any issues during deployment, contact our support team:

- Email: api-support@xashpay.com
- Support Portal: https://support.xashpay.com
- API Status: https://status.xashpay.com

## Additional Resources

- [API Reference](api-reference.md)
- [Examples](examples.md)
- [Troubleshooting](troubleshooting.md)
- [Service Documentation](services.md)
- [Voucher System](vouchers.md)
- [Wallet Management](wallet.md)
