---
sidebar_position: 6
---

# Floating IPs API Reference

The Floating IPs API allows you to manage static IP addresses that can be attached to servers for external access.

## Available Methods

### `list(array $parameters = [])`

Get a list of all floating IPs.

**Parameters:**
- `name` (string, optional): Filter by floating IP name
- `label_selector` (string, optional): Filter by label selector
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all floating IPs
$floatingIps = HetznerLaravel::floatingIps()->list();

// Get floating IPs with filters
$floatingIps = HetznerLaravel::floatingIps()->list([
    'name' => 'web-server-ip',
    'per_page' => 50,
]);
```

### `create(array $parameters)`

Create a new floating IP.

**Required Parameters:**
- `type` (string): Floating IP type ('ipv4' or 'ipv6')
- `location` (string): Location name or ID

**Optional Parameters:**
- `name` (string): Floating IP name
- `home_location` (string): Home location (for IPv6)
- `server` (int): Server ID to assign to
- `labels` (array): Key-value pairs for labeling

**Example:**
```php
$floatingIp = HetznerLaravel::floatingIps()->create([
    'type' => 'ipv4',
    'location' => 'nbg1',
    'name' => 'my-floating-ip',
    'labels' => [
        'environment' => 'production',
        'purpose' => 'web-server'
    ],
]);
```

### `retrieve(string $floatingIpId)`

Get details of a specific floating IP.

**Parameters:**
- `floatingIpId` (string): Floating IP ID

**Example:**
```php
$floatingIp = HetznerLaravel::floatingIps()->retrieve('12345');
echo $floatingIp->ip; // Floating IP address
echo $floatingIp->server->id; // Assigned server ID
```

### `update(string $floatingIpId, array $parameters)`

Update floating IP properties.

**Parameters:**
- `floatingIpId` (string): Floating IP ID
- `name` (string, optional): New floating IP name
- `labels` (array, optional): New labels

**Example:**
```php
$floatingIp = HetznerLaravel::floatingIps()->update('12345', [
    'name' => 'updated-floating-ip',
    'labels' => [
        'environment' => 'staging',
        'updated' => 'true'
    ]
]);
```

### `delete(string $floatingIpId)`

Delete a floating IP.

**Parameters:**
- `floatingIpId` (string): Floating IP ID

**Example:**
```php
HetznerLaravel::floatingIps()->delete('12345');
```

## Floating IP Actions

### `actions()`

Access floating IP actions for assigning, unassigning, and changing protection.

**Example:**
```php
// Assign floating IP to server
$action = HetznerLaravel::floatingIps()->actions()->assign('12345', [
    'assignee' => 67890, // Server ID
]);

// Unassign floating IP from server
$action = HetznerLaravel::floatingIps()->actions()->unassign('12345');

// Change protection
$action = HetznerLaravel::floatingIps()->actions()->changeProtection('12345', [
    'delete' => true
]);
```

## Response Objects

### Floating IP Object
- `id` (int): Floating IP ID
- `name` (string): Floating IP name
- `ip` (string): Floating IP address
- `type` (string): IP type ('ipv4' or 'ipv6')
- `created` (string): Creation timestamp
- `home_location` (object): Home location
- `blocked` (bool): Block status
- `dns_ptr` (array): DNS pointer records
- `server` (object): Assigned server
- `protection` (object): Protection settings
- `labels` (object): Floating IP labels

## Common Use Cases

### 1. Create and Assign Floating IP to Server

```php
// Create floating IP
$floatingIp = HetznerLaravel::floatingIps()->create([
    'type' => 'ipv4',
    'location' => 'nbg1',
    'name' => 'web-server-floating-ip',
    'labels' => [
        'role' => 'web-server',
        'environment' => 'production'
    ]
]);

// Assign to server
HetznerLaravel::floatingIps()->actions()->assign($floatingIp->id, [
    'assignee' => 12345, // Server ID
]);
```

### 2. Create IPv6 Floating IP

```php
$floatingIp = HetznerLaravel::floatingIps()->create([
    'type' => 'ipv6',
    'location' => 'nbg1',
    'name' => 'web-server-ipv6',
    'home_location' => 'nbg1',
    'labels' => [
        'role' => 'web-server',
        'environment' => 'production',
        'ip_version' => 'ipv6'
    ]
]);
```

### 3. Manage Multiple Floating IPs

```php
// Create multiple floating IPs for different servers
$servers = [12345, 67890, 11111];
$floatingIps = [];

foreach ($servers as $index => $serverId) {
    $floatingIp = HetznerLaravel::floatingIps()->create([
        'type' => 'ipv4',
        'location' => 'nbg1',
        'name' => "server-{$index}-floating-ip",
        'labels' => [
            'server_id' => $serverId,
            'environment' => 'production'
        ]
    ]);
    
    // Assign to server
    HetznerLaravel::floatingIps()->actions()->assign($floatingIp->id, [
        'assignee' => $serverId,
    ]);
    
    $floatingIps[] = $floatingIp;
}
```

### 4. Floating IP with DNS Records

```php
// Create floating IP
$floatingIp = HetznerLaravel::floatingIps()->create([
    'type' => 'ipv4',
    'location' => 'nbg1',
    'name' => 'api-server-floating-ip',
]);

// Create DNS zone
$dnsZone = HetznerLaravel::dnsZones()->create([
    'name' => 'api.example.com',
]);

// Add A record pointing to floating IP
HetznerLaravel::dnsZones()->rrsets()->create($dnsZone->id, [
    'type' => 'A',
    'name' => '@',
    'records' => [
        [
            'value' => $floatingIp->ip
        ]
    ],
    'ttl' => 3600,
]);
```

### 5. Load Balancer with Floating IP

```php
// Create floating IP for load balancer
$floatingIp = HetznerLaravel::floatingIps()->create([
    'type' => 'ipv4',
    'location' => 'nbg1',
    'name' => 'load-balancer-floating-ip',
    'labels' => [
        'role' => 'load-balancer',
        'environment' => 'production'
    ]
]);

// Create load balancer
$loadBalancer = HetznerLaravel::loadBalancers()->create([
    'name' => 'web-load-balancer',
    'load_balancer_type' => 'lb11',
    'location' => 'nbg1',
    'public_interface' => [
        'ipv4' => [
            'id' => $floatingIp->id,
        ],
    ],
    'targets' => [
        [
            'type' => 'server',
            'server' => [
                'id' => 12345,
            ],
        ]
    ],
    'services' => [
        [
            'protocol' => 'http',
            'listen_port' => 80,
            'destination_port' => 80,
        ]
    ],
]);
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $floatingIp = HetznerLaravel::floatingIps()->create($parameters);
} catch (ErrorException $e) {
    // Handle API errors
    echo "API Error: " . $e->getMessage();
    echo "Error Code: " . $e->getCode();
} catch (TransporterException $e) {
    // Handle network/transport errors
    echo "Network Error: " . $e->getMessage();
}
```

## Related Resources

- [Servers API](./servers) - Assign floating IPs to servers
- [Load Balancers API](./load-balancers) - Use floating IPs with load balancers
- [DNS Zones API](./dns-zones) - Create DNS records for floating IPs
- [Networks API](./networks) - Configure private networks
