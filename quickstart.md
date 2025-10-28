---
sidebar_position: 3
---

# Quick Start

Get up and running with Hetzner Laravel in minutes! This guide will walk you through creating your first server and performing basic operations.

## Prerequisites

Before starting, make sure you have:

- ✅ Hetzner Laravel package installed (see [Installation Guide](./installation))
- ✅ Valid Hetzner Cloud API token
- ✅ Basic Laravel knowledge

## Your First Server

Let's create your first Hetzner Cloud server using Laravel.

### Step 1: Create a Simple Controller

Create a controller to handle server operations:

```bash
php artisan make:controller ServerController
```

### Step 2: Add Server Creation Method

```php
<?php

namespace App\Http\Controllers;

use Boci\HetznerLaravel\Facades\HetznerLaravel;
use Illuminate\Http\Request;

class ServerController extends Controller
{
    public function createServer(Request $request)
    {
        try {
            $server = HetznerLaravel::servers()->create([
                'name' => 'my-first-server',
                'server_type' => 'cpx11', // 1 vCPU, 2 GB RAM
                'image' => 'ubuntu-24.04',
                'location' => 'nbg1', // Nuremberg
                'ssh_keys' => ['my-ssh-key'], // Optional: your SSH key name
            ]);

            return response()->json([
                'success' => true,
                'server' => $server->toArray(),
                'message' => 'Server created successfully!'
            ]);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'error' => $e->getMessage()
            ], 500);
        }
    }
}
```

### Step 3: Add Route

Add a route in your `routes/web.php` or `routes/api.php`:

```php
use App\Http\Controllers\ServerController;

Route::post('/servers', [ServerController::class, 'createServer']);
```

### Step 4: Test the Server Creation

You can test this using:

**cURL:**
```bash
curl -X POST http://your-app.test/servers \
  -H "Content-Type: application/json"
```

**Or create a simple test:**
```php
// tests/Feature/ServerTest.php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use Boci\HetznerLaravel\Facades\HetznerLaravel;

class ServerTest extends TestCase
{
    public function test_can_create_server()
    {
        $server = HetznerLaravel::servers()->create([
            'name' => 'test-server-' . time(),
            'server_type' => 'cpx11',
            'image' => 'ubuntu-24.04',
            'location' => 'nbg1',
        ]);

        $this->assertNotNull($server);
        $this->assertEquals('test-server-' . time(), $server->name);
    }
}
```

## Common Operations

### List All Servers

```php
public function listServers()
{
    $servers = HetznerLaravel::servers()->list();
    
    return response()->json([
        'servers' => $servers->toArray()
    ]);
}
```

### Get Server Details

```php
public function getServer($serverId)
{
    $server = HetznerLaravel::servers()->retrieve($serverId);
    
    return response()->json(
        $server->toArray()
    );
}
```

### Delete a Server

```php
public function deleteServer($serverId)
{
    try {
        HetznerLaravel::servers()->delete($serverId);
        
        return response()->json([
            'success' => true,
            'message' => 'Server deleted successfully'
        ]);
    } catch (\Exception $e) {
        return response()->json([
            'success' => false,
            'error' => $e->getMessage()
        ], 500);
    }
}
```

## Working with DNS Zones

### Create a DNS Zone

```php
public function createDnsZone(Request $request)
{
    $dnsZone = HetznerLaravel::dnsZones()->create([
        'name' => 'example.com',
        'mode' => 'primary',
        'ttl' => 3600,
    ]);

    return response()->json([
        'success' => true,
        'dns_zone' => $dnsZone->toArray()
    ]);
}
```

### Add DNS Records

```php
public function addDnsRecord(Request $request, $zoneId)
{
    $record = HetznerLaravel::dnsZones()->rrsets()->create($zoneId, [
            'name' => 'www',
            'type' => 'A',
            'ttl' => 3600,
            'records' => [
                [
                    'value' => '198.51.100.1',
                    'comment' => 'My web server at Hetzner Cloud.'
                ]
            ],
            'labels' => [
                'environment' => 'prod',
                'example.com/my' => 'label',
                'just-a-key' => ''
            ]
    ]);

    return response()->json([
        'success' => true,
        'record' => $record->toArray()
    ]);
}
```

## Working with Networks

### Create a Private Network

```php
public function createNetwork(Request $request)
{
    $network = HetznerLaravel::networks()->create([
        'name' => 'my-private-network',
        'ip_range' => '10.0.0.0/16',
        'subnets' => [
            [
                'type' => 'cloud',
                'ip_range' => '10.0.1.0/24',
                'network_zone' => 'eu-central',
            ]
        ],
    ]);

    return response()->json([
        'success' => true,
        'network' => $network->toArray()
    ]);
}
```

## Working with Load Balancers

### Create a Load Balancer

```php
public function createLoadBalancer(Request $request)
{
    $loadBalancer = HetznerLaravel::loadBalancers()->create([
        'name' => 'my-load-balancer',
        'load_balancer_type' => 'lb11',
        'location' => 'nbg1',
        'targets' => [
            [
                'type' => 'server',
                'server' => [
                    'id' => 12345, // Your server ID
                ],
            ]
        ],
    ]);

    return response()->json([
        'success' => true,
        'load_balancer' => $loadBalancer->toArray()
    ]);
}
```

## Error Handling

Always wrap your API calls in try-catch blocks:

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $server = HetznerLaravel::servers()->create($parameters);
} catch (ErrorException $e) {
    // Handle API errors (400, 401, 403, 404, etc.)
    return response()->json([
        'error' => 'API Error: ' . $e->getMessage(),
        'code' => $e->getCode()
    ], 400);
} catch (TransporterException $e) {
    // Handle network/transport errors
    return response()->json([
        'error' => 'Network Error: ' . $e->getMessage()
    ], 500);
} catch (\Exception $e) {
    // Handle any other errors
    return response()->json([
        'error' => 'Unexpected Error: ' . $e->getMessage()
    ], 500);
}
```

## Best Practices

### 1. Use Dependency Injection

```php
use Boci\HetznerLaravel\Client;

class ServerService
{
    public function __construct(
        private Client $hetznerClient
    ) {}

    public function createServer(array $data)
    {
        return $this->hetznerClient->servers()->create($data);
    }
}
```

### 2. Validate Input Data

```php
public function createServer(Request $request)
{
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'server_type' => 'required|string',
        'image' => 'required|string',
        'location' => 'required|string',
    ]);

    $server = HetznerLaravel::servers()->create($validated);
    
    return response()->json($server->toArray());
}
```

### 3. Use Queues for Long Operations

```php
// Create a job for server creation
php artisan make:job CreateServerJob

// In your job
class CreateServerJob implements ShouldQueue
{
    public function handle()
    {
        $server = HetznerLaravel::servers()->create($this->serverData);
        
        // Notify user when server is ready
        Mail::to($this->user)->send(new ServerCreatedMail($server));
    }
}
```

## Next Steps

Now that you've created your first server, explore more features:

1. **[API Reference](./api-reference/servers)** - Complete method documentation
2. **[Examples](./examples/server-management)** - Real-world usage patterns
3. **[Error Handling](./error-handling)** - Advanced error management
4. **[Troubleshooting](./troubleshooting)** - Common issues and solutions

## Need Help?

- 📚 Check the [API Reference](./api-reference/servers) for detailed method documentation
- 🌐 Visit [docs.hetzner.cloud](https://docs.hetzner.cloud/) for official Hetzner Cloud API documentation
- 🐛 Report issues on [GitHub](https://github.com/amar8eka/hetzner-laravel/issues)
- 💬 Join community discussions
- 📧 Contact support at amar8eka@gmail.com
