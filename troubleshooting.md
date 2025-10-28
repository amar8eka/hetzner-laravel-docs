---
sidebar_position: 5
---

# Troubleshooting

This guide helps you resolve common issues when using the Hetzner Laravel package.

## Common Issues and Solutions

### 1. Authentication Issues

#### Problem: "API token not provided" Error

**Symptoms:**
- Error message: "API token not provided"
- 401 Unauthorized responses

**Solutions:**

1. **Check Environment Variables**
   ```bash
   # Verify your .env file contains the token
   cat .env | grep HETZNER_TOKEN
   ```

2. **Clear Configuration Cache**
   ```bash
   php artisan config:clear
   php artisan cache:clear
   ```

3. **Verify Token Format**
   ```php
   // Your token should look like this
   HETZNER_TOKEN=hcloud_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   ```

4. **Test Token Manually**
   ```php
   // Test in tinker
   php artisan tinker
   
   // Then run:
   echo env('HETZNER_TOKEN');
   ```

#### Problem: "Invalid API token" Error

**Symptoms:**
- Error message: "Invalid API token"
- 401 Unauthorized responses

**Solutions:**

1. **Regenerate Token**
   - Go to [Hetzner Cloud Console](https://console.hetzner.cloud/)
   - Navigate to Security → API Tokens
   - Create a new token
   - Update your `.env` file

2. **Check Token Permissions**
   - Ensure the token has the required permissions
   - For server management, you need "Read & Write" permissions

### 2. Server Creation Issues

#### Problem: "Server type not available" Error

**Symptoms:**
- Error message: "Server type not available"
- 422 Validation error

**Solutions:**

1. **Check Available Server Types**
   ```php
   $serverTypes = HetznerLaravel::serverTypes()->list();
   foreach ($serverTypes as $type) {
       echo $type->name . " - " . $type->description . "\n";
   }
   ```

2. **Use Correct Server Type Names**
   ```php
   // Correct format
   'server_type' => 'cpx11'  // Not 'CPX11' or 'cpx-11'
   ```

3. **Check Location Availability**
   ```php
   $locations = HetznerLaravel::locations()->list();
   foreach ($locations as $location) {
       echo $location->name . " - " . $location->description . "\n";
   }
   ```

#### Problem: "Image not available" Error

**Symptoms:**
- Error message: "Image not available"
- 422 Validation error

**Solutions:**

1. **List Available Images**
   ```php
   $images = HetznerLaravel::images()->list();
   foreach ($images as $image) {
       echo $image->name . " - " . $image->description . "\n";
   }
   ```

2. **Use Correct Image Names**
   ```php
   // Correct format
   'image' => 'ubuntu-24.04'  // Not 'Ubuntu 24.04' or 'ubuntu_24.04'
   ```

### 3. Network and Connection Issues

#### Problem: "Connection timeout" Error

**Symptoms:**
- Error message: "Connection timeout"
- TransporterException thrown

**Solutions:**

1. **Increase Timeout**
   ```php
   // In config/hetzner-laravel.php
   return [
       'timeout' => 60, // Increase from default 30 seconds
   ];
   ```

2. **Check Network Connectivity**
   ```bash
   # Test connectivity to Hetzner API
   curl -I https://api.hetzner.cloud/v1/servers
   ```

3. **Use Custom HTTP Client**
   ```php
   use GuzzleHttp\Client as GuzzleClient;
   
   $client = Client::make('your-api-token')
       ->withHttpClient(new GuzzleClient([
           'timeout' => 60,
           'connect_timeout' => 10,
       ]));
   ```

#### Problem: "SSL certificate problem" Error

**Symptoms:**
- Error message: "SSL certificate problem"
- TransporterException thrown

**Solutions:**

1. **For Development Only - Disable SSL Verification**
   ```php
   // In config/hetzner-laravel.php
   return [
       'verify_ssl' => false, // Only for development!
   ];
   ```

2. **Update CA Certificates**
   ```bash
   # Ubuntu/Debian
   sudo apt-get update
   sudo apt-get install ca-certificates
   
   # CentOS/RHEL
   sudo yum update ca-certificates
   ```

### 4. DNS Zone Issues

#### Problem: "Zone already exists" Error

**Symptoms:**
- Error message: "Zone already exists"
- 422 Validation error

**Solutions:**

1. **Check Existing Zones**
   ```php
   $zones = HetznerLaravel::dnsZones()->list();
   foreach ($zones as $zone) {
       echo $zone->name . "\n";
   }
   ```

2. **Use Different Zone Name**
   ```php
   // Use a subdomain or different domain
   'name' => 'subdomain.example.com'
   ```

#### Problem: "Invalid DNS record" Error

**Symptoms:**
- Error message: "Invalid DNS record"
- 422 Validation error

**Solutions:**

1. **Validate Record Format**
   ```php
   // Correct A record format
   [
       'type' => 'A',
       'name' => 'www',
       'records' => [
           [
               'value' => '1.2.3.4'
           ]
       ],
       'ttl' => 3600,
   ]
   
   // Correct MX record format
   [
       'type' => 'MX',
       'name' => '@',
       'records' => [
           [
               'value' => '10 mail.example.com'
           ]
       ],
       'ttl' => 3600,
   ]
   ```

2. **Check TTL Values**
   ```php
   // TTL must be between 60 and 86400 seconds
   'ttl' => 300  // Valid
   'ttl' => 30   // Invalid - too low
   ```

### 5. Volume and Storage Issues

#### Problem: "Volume not found" Error

**Symptoms:**
- Error message: "Volume not found"
- 404 Not Found error

**Solutions:**

1. **List Available Volumes**
   ```php
   $volumes = HetznerLaravel::volumes()->list();
   foreach ($volumes as $volume) {
       echo $volume->id . " - " . $volume->name . "\n";
   }
   ```

2. **Create Volume First**
   ```php
   $volume = HetznerLaravel::volumes()->create([
       'name' => 'my-volume',
       'size' => 20,
       'location' => 'nbg1',
   ]);
   
   // Then use the volume ID
   $server = HetznerLaravel::servers()->create([
       'name' => 'my-server',
       'server_type' => 'cpx11',
       'image' => 'ubuntu-24.04',
       'location' => 'nbg1',
       'volumes' => [$volume->id],
   ]);
   ```

### 6. Load Balancer Issues

#### Problem: "Load balancer type not available" Error

**Symptoms:**
- Error message: "Load balancer type not available"
- 422 Validation error

**Solutions:**

1. **Check Available Load Balancer Types**
   ```php
   $types = HetznerLaravel::loadBalancerTypes()->list();
   foreach ($types as $type) {
       echo $type->name . " - " . $type->description . "\n";
   }
   ```

2. **Use Correct Type Names**
   ```php
   // Correct format
   'load_balancer_type' => 'lb11'  // Not 'LB11' or 'lb-11'
   ```

### 7. Rate Limiting Issues

#### Problem: "Rate limit exceeded" Error

**Symptoms:**
- Error message: "Rate limit exceeded"
- 429 Too Many Requests error

**Solutions:**

1. **Implement Retry Logic**
   ```php
   use Illuminate\Support\Facades\Cache;
   
   public function createServerWithRetry(array $data): array
   {
       $maxRetries = 3;
       $retryDelay = 1000; // milliseconds
       
       for ($attempt = 0; $attempt < $maxRetries; $attempt++) {
           try {
               return HetznerLaravel::servers()->create($data);
           } catch (ErrorException $e) {
               if ($e->getCode() === 429 && $attempt < $maxRetries - 1) {
                   usleep($retryDelay * 1000);
                   continue;
               }
               throw $e;
           }
       }
   }
   ```

2. **Implement Request Queuing**
   ```php
   // Use Laravel queues for non-urgent operations
   dispatch(new CreateServerJob($data));
   ```

### 8. Configuration Issues

#### Problem: Package Not Working After Installation

**Symptoms:**
- Class not found errors
- Service provider not registered

**Solutions:**

1. **Check Service Provider Registration**
   ```php
   // In config/app.php, ensure the service provider is registered
   'providers' => [
       // ... other providers
       Boci\HetznerLaravel\HetznerLaravelServiceProvider::class,
   ],
   ```

2. **Clear All Caches**
   ```bash
   php artisan config:clear
   php artisan cache:clear
   php artisan route:clear
   php artisan view:clear
   composer dump-autoload
   ```

3. **Check Package Installation**
   ```bash
   composer show boci/hetzner-laravel
   ```

### 9. Debugging Tips

#### Enable Debug Mode

```php
// In config/hetzner-laravel.php
return [
    'debug' => true,
    'log_requests' => true,
];
```

#### Check API Response

```php
try {
    $server = HetznerLaravel::servers()->create($data);
} catch (ErrorException $e) {
    // Log the full response for debugging
    Log::error('Hetzner API Error', [
        'code' => $e->getCode(),
        'message' => $e->getMessage(),
        'response_body' => $e->getResponseBody(),
    ]);
}
```

#### Test API Connection

```php
// Create a simple test command
php artisan make:command TestHetznerConnection

// In the command
public function handle()
{
    try {
        $servers = HetznerLaravel::servers()->list();
        $this->info('Connection successful! Found ' . count($servers) . ' servers.');
    } catch (\Exception $e) {
        $this->error('Connection failed: ' . $e->getMessage());
    }
}
```

## Getting Help

### 1. Check Documentation
- Review the [API Reference](../api-reference/servers) for detailed method documentation
- Check [Examples](../examples/server-management) for usage patterns

### 2. Enable Logging
```php
// Add to config/logging.php
'channels' => [
    'hetzner' => [
        'driver' => 'daily',
        'path' => storage_path('logs/hetzner.log'),
        'level' => 'debug',
    ],
],
```

### 3. Community Support
- [GitHub Issues](https://github.com/amar8eka/hetzner-laravel/issues) - Report bugs and request features
- [GitHub Discussions](https://github.com/amar8eka/hetzner-laravel/discussions) - Ask questions and share solutions

### 4. Professional Support
- Email: amar8eka@gmail.com
- For security issues, please email directly instead of using the issue tracker

## Related Resources

- [Error Handling](./error-handling) - Comprehensive error management guide
- [Installation Guide](./installation) - Proper package setup
- [Quick Start Guide](./quickstart) - Basic usage examples
- [API Reference](../api-reference/servers) - Complete API documentation
