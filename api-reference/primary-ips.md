---
sidebar_position: 18
---

# Primary IPs API Reference

The Primary IPs API allows you to manage static IP addresses that can be assigned to servers for external access.

## Available Methods

### `list(array $parameters = [])`

Get a list of all primary IPs.

**Parameters:**
- `name` (string, optional): Filter by primary IP name
- `label_selector` (string, optional): Filter by label selector
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all primary IPs
$primaryIps = HetznerLaravel::primaryIps()->list();

// Get primary IPs with filters
$primaryIps = HetznerLaravel::primaryIps()->list([
    'name' => 'web-server-ip',
    'per_page' => 50,
]);
```

### `create(array $parameters)`

Create a new primary IP.

**Required Parameters:**
- `type` (string): Primary IP type ('ipv4' or 'ipv6')
- `location` (string): Location name or ID

**Optional Parameters:**
- `name` (string): Primary IP name
- `assignee_id` (int): Server ID to assign to
- `labels` (array): Key-value pairs for labeling

**Example:**
```php
$primaryIp = HetznerLaravel::primaryIps()->create([
    'type' => 'ipv4',
    'location' => 'nbg1',
    'name' => 'my-primary-ip',
    'labels' => [
        'environment' => 'production',
        'purpose' => 'web-server'
    ],
]);
```

### `retrieve(string $primaryIpId)`

Get details of a specific primary IP.

**Parameters:**
- `primaryIpId` (string): Primary IP ID

**Example:**
```php
$primaryIp = HetznerLaravel::primaryIps()->retrieve('12345');
echo $primaryIp->ip; // Primary IP address
echo $primaryIp->assignee->id; // Assigned server ID
```

### `update(string $primaryIpId, array $parameters)`

Update primary IP properties.

**Parameters:**
- `primaryIpId` (string): Primary IP ID
- `name` (string, optional): New primary IP name
- `labels` (array, optional): New labels

**Example:**
```php
$primaryIp = HetznerLaravel::primaryIps()->update('12345', [
    'name' => 'updated-primary-ip',
    'labels' => [
        'environment' => 'staging',
        'updated' => 'true'
    ]
]);
```

### `delete(string $primaryIpId)`

Delete a primary IP.

**Parameters:**
- `primaryIpId` (string): Primary IP ID

**Example:**
```php
HetznerLaravel::primaryIps()->delete('12345');
```

## Primary IP Actions

### `actions()`

Access primary IP actions for assigning, unassigning, and changing protection.

**Example:**
```php
// Assign primary IP to server
$action = HetznerLaravel::primaryIps()->actions()->assign('12345', [
    'assignee_id' => 67890, // Server ID
]);

// Unassign primary IP from server
$action = HetznerLaravel::primaryIps()->actions()->unassign('12345');

// Change protection
$action = HetznerLaravel::primaryIps()->actions()->changeProtection('12345', [
    'delete' => true
]);
```

## Response Objects

### Primary IP Object
- `id` (int): Primary IP ID
- `name` (string): Primary IP name
- `ip` (string): Primary IP address
- `type` (string): IP type ('ipv4' or 'ipv6')
- `created` (string): Creation timestamp
- `location` (object): Location information
- `assignee` (object): Assigned server
- `dns_ptr` (array): DNS pointer records
- `protection` (object): Protection settings
- `labels` (object): Primary IP labels

## Common Use Cases

### 1. Create and Assign Primary IP to Server

```php
// Create primary IP
$primaryIp = HetznerLaravel::primaryIps()->create([
    'type' => 'ipv4',
    'location' => 'nbg1',
    'name' => 'web-server-primary-ip',
    'labels' => [
        'role' => 'web-server',
        'environment' => 'production'
    ]
]);

// Assign to server
HetznerLaravel::primaryIps()->actions()->assign($primaryIp->id, [
    'assignee_id' => 12345, // Server ID
]);
```

### 2. Create IPv6 Primary IP

```php
$primaryIp = HetznerLaravel::primaryIps()->create([
    'type' => 'ipv6',
    'location' => 'nbg1',
    'name' => 'web-server-ipv6',
    'labels' => [
        'role' => 'web-server',
        'environment' => 'production',
        'ip_version' => 'ipv6'
    ]
]);
```

### 3. Manage Multiple Primary IPs

```php
// Create multiple primary IPs for different servers
$servers = [12345, 67890, 11111];
$primaryIps = [];

foreach ($servers as $index => $serverId) {
    $primaryIp = HetznerLaravel::primaryIps()->create([
        'type' => 'ipv4',
        'location' => 'nbg1',
        'name' => "server-{$index}-primary-ip",
        'labels' => [
            'server_id' => $serverId,
            'environment' => 'production'
        ]
    ]);
    
    // Assign to server
    HetznerLaravel::primaryIps()->actions()->assign($primaryIp->id, [
        'assignee_id' => $serverId,
    ]);
    
    $primaryIps[] = $primaryIp;
}
```

### 4. Primary IP with DNS Records

```php
// Create primary IP
$primaryIp = HetznerLaravel::primaryIps()->create([
    'type' => 'ipv4',
    'location' => 'nbg1',
    'name' => 'api-server-primary-ip',
]);

// Create DNS zone
$dnsZone = HetznerLaravel::dnsZones()->create([
    'name' => 'api.example.com',
]);

// Add A record pointing to primary IP
HetznerLaravel::dnsZones()->rrsets()->create($dnsZone->id, [
    'type' => 'A',
    'name' => '@',
    'records' => [
        [
            'value' => $primaryIp->ip
        ]
    ],
    'ttl' => 3600,
]);
```

### 5. Load Balancer with Primary IP

```php
// Create primary IP for load balancer
$primaryIp = HetznerLaravel::primaryIps()->create([
    'type' => 'ipv4',
    'location' => 'nbg1',
    'name' => 'load-balancer-primary-ip',
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
            'id' => $primaryIp->id,
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

### 6. Primary IP Management Service

```php
<?php

namespace App\Services;

use Boci\HetznerLaravel\Facades\HetznerLaravel;

class PrimaryIpManagementService
{
    public function createPrimaryIp(string $type, string $location, string $name, array $labels = []): array
    {
        try {
            $primaryIp = HetznerLaravel::primaryIps()->create([
                'type' => $type,
                'location' => $location,
                'name' => $name,
                'labels' => $labels,
            ]);
            
            return [
                'success' => true,
                'primary_ip' => $primaryIp->toArray(),
                'message' => 'Primary IP created successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function getPrimaryIp(string $primaryIpId): array
    {
        try {
            $primaryIp = HetznerLaravel::primaryIps()->retrieve($primaryIpId);
            
            return [
                'success' => true,
                'primary_ip' => $primaryIp->toArray()
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function listPrimaryIps(array $filters = []): array
    {
        try {
            $primaryIps = HetznerLaravel::primaryIps()->list($filters);
            
            return [
                'success' => true,
                'primary_ips' => $primaryIps->toArray(),
                'count' => count($primaryIps)
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function updatePrimaryIp(string $primaryIpId, array $data): array
    {
        try {
            $primaryIp = HetznerLaravel::primaryIps()->update($primaryIpId, $data);
            
            return [
                'success' => true,
                'primary_ip' => $primaryIp->toArray(),
                'message' => 'Primary IP updated successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function deletePrimaryIp(string $primaryIpId): array
    {
        try {
            HetznerLaravel::primaryIps()->delete($primaryIpId);
            
            return [
                'success' => true,
                'message' => 'Primary IP deleted successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function assignPrimaryIp(string $primaryIpId, int $serverId): array
    {
        try {
            $action = HetznerLaravel::primaryIps()->actions()->assign($primaryIpId, [
                'assignee_id' => $serverId,
            ]);
            
            return [
                'success' => true,
                'action' => $action->toArray(),
                'message' => 'Primary IP assigned successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function unassignPrimaryIp(string $primaryIpId): array
    {
        try {
            $action = HetznerLaravel::primaryIps()->actions()->unassign($primaryIpId);
            
            return [
                'success' => true,
                'action' => $action->toArray(),
                'message' => 'Primary IP unassigned successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
}
```

### 7. High Availability with Primary IPs

```php
// Create primary IPs for high availability setup
$primaryIps = [];

for ($i = 1; $i <= 3; $i++) {
    $primaryIp = HetznerLaravel::primaryIps()->create([
        'type' => 'ipv4',
        'location' => 'nbg1',
        'name' => "ha-server-{$i}-primary-ip",
        'labels' => [
            'role' => 'ha-server',
            'environment' => 'production',
            'server_index' => $i
        ]
    ]);
    
    $primaryIps[] = $primaryIp;
}

// Create servers and assign primary IPs
$servers = [];
foreach ($primaryIps as $index => $primaryIp) {
    $server = HetznerLaravel::servers()->create([
        'name' => "ha-server-" . ($index + 1),
        'server_type' => 'cpx21',
        'image' => 'ubuntu-24.04',
        'location' => 'nbg1',
        'ssh_keys' => ['my-ssh-key'],
        'labels' => [
            'role' => 'ha-server',
            'environment' => 'production',
            'primary_ip' => $primaryIp->ip
        ]
    ]);
    
    // Assign primary IP to server
    HetznerLaravel::primaryIps()->actions()->assign($primaryIp->id, [
        'assignee_id' => $server->id,
    ]);
    
    $servers[] = $server;
}

// Create load balancer to distribute traffic
$loadBalancer = HetznerLaravel::loadBalancers()->create([
    'name' => 'ha-load-balancer',
    'load_balancer_type' => 'lb11',
    'location' => 'nbg1',
    'targets' => array_map(function($server) {
        return [
            'type' => 'server',
            'server' => [
                'id' => $server->id,
            ],
        ];
    }, $servers),
    'services' => [
        [
            'protocol' => 'http',
            'listen_port' => 80,
            'destination_port' => 80,
        ]
    ],
    'labels' => [
        'role' => 'ha-load-balancer',
        'environment' => 'production'
    ]
]);
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $primaryIp = HetznerLaravel::primaryIps()->create($parameters);
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

- [Servers API](./servers) - Assign primary IPs to servers
- [Load Balancers API](./load-balancers) - Use primary IPs with load balancers
- [DNS Zones API](./dns-zones) - Create DNS records for primary IPs
- [Networks API](./networks) - Configure private networks
- [Floating IPs API](./floating-ips) - Compare with floating IPs
