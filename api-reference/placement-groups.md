---
sidebar_position: 17
---

# Placement Groups API Reference

The Placement Groups API allows you to manage server placement groups for controlling server distribution across physical hosts.

## Available Methods

### `list(array $parameters = [])`

Get a list of all placement groups.

**Parameters:**
- `name` (string, optional): Filter by placement group name
- `label_selector` (string, optional): Filter by label selector
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all placement groups
$placementGroups = HetznerLaravel::placementGroups()->list();

// Get placement groups with filters
$placementGroups = HetznerLaravel::placementGroups()->list([
    'name' => 'web-servers-group',
    'per_page' => 50,
]);
```

### `create(array $parameters)`

Create a new placement group.

**Required Parameters:**
- `name` (string): Placement group name
- `type` (string): Placement group type ('spread' or 'anti_affinity')

**Optional Parameters:**
- `labels` (array): Key-value pairs for labeling

**Example:**
```php
$placementGroup = HetznerLaravel::placementGroups()->create([
    'name' => 'my-placement-group',
    'type' => 'spread',
    'labels' => [
        'environment' => 'production',
        'purpose' => 'web-servers'
    ],
]);
```

### `retrieve(string $placementGroupId)`

Get details of a specific placement group.

**Parameters:**
- `placementGroupId` (string): Placement group ID

**Example:**
```php
$placementGroup = HetznerLaravel::placementGroups()->retrieve('12345');
echo $placementGroup->name; // Placement group name
echo $placementGroup->type; // Placement group type
echo count($placementGroup->servers); // Number of servers
```

### `update(string $placementGroupId, array $parameters)`

Update placement group properties.

**Parameters:**
- `placementGroupId` (string): Placement group ID
- `name` (string, optional): New placement group name
- `labels` (array, optional): New labels

**Example:**
```php
$placementGroup = HetznerLaravel::placementGroups()->update('12345', [
    'name' => 'updated-placement-group',
    'labels' => [
        'environment' => 'staging',
        'updated' => 'true'
    ]
]);
```

### `delete(string $placementGroupId)`

Delete a placement group.

**Parameters:**
- `placementGroupId` (string): Placement group ID

**Example:**
```php
HetznerLaravel::placementGroups()->delete('12345');
```

## Response Objects

### Placement Group Object
- `id` (int): Placement group ID
- `name` (string): Placement group name
- `type` (string): Placement group type ('spread' or 'anti_affinity')
- `created` (string): Creation timestamp
- `servers` (array): Servers in the placement group
- `labels` (object): Placement group labels

## Common Use Cases

### 1. Create Spread Placement Group

```php
$placementGroup = HetznerLaravel::placementGroups()->create([
    'name' => 'web-servers-spread',
    'type' => 'spread',
    'labels' => [
        'role' => 'web-servers',
        'environment' => 'production',
        'strategy' => 'spread'
    ]
]);

// Create servers in the placement group
$servers = [];
for ($i = 1; $i <= 3; $i++) {
    $server = HetznerLaravel::servers()->create([
        'name' => "web-server-{$i}",
        'server_type' => 'cpx21',
        'image' => 'ubuntu-24.04',
        'location' => 'nbg1',
        'placement_group' => $placementGroup->id,
        'ssh_keys' => ['my-ssh-key'],
        'labels' => [
            'role' => 'web-server',
            'placement_group' => $placementGroup->name
        ]
    ]);
    $servers[] = $server;
}
```

### 2. Create Anti-Affinity Placement Group

```php
$placementGroup = HetznerLaravel::placementGroups()->create([
    'name' => 'database-servers-anti-affinity',
    'type' => 'anti_affinity',
    'labels' => [
        'role' => 'database-servers',
        'environment' => 'production',
        'strategy' => 'anti-affinity'
    ]
]);

// Create database servers that won't be on the same physical host
$dbServers = [];
for ($i = 1; $i <= 2; $i++) {
    $server = HetznerLaravel::servers()->create([
        'name' => "db-server-{$i}",
        'server_type' => 'cpx31',
        'image' => 'ubuntu-24.04',
        'location' => 'nbg1',
        'placement_group' => $placementGroup->id,
        'ssh_keys' => ['my-ssh-key'],
        'labels' => [
            'role' => 'database-server',
            'placement_group' => $placementGroup->name
        ]
    ]);
    $dbServers[] = $server;
}
```

### 3. Multi-Tier Architecture with Placement Groups

```php
// Create placement groups for different tiers
$placementGroups = [
    'web' => HetznerLaravel::placementGroups()->create([
        'name' => 'web-tier-spread',
        'type' => 'spread',
        'labels' => [
            'tier' => 'web',
            'environment' => 'production'
        ]
    ]),
    'app' => HetznerLaravel::placementGroups()->create([
        'name' => 'app-tier-spread',
        'type' => 'spread',
        'labels' => [
            'tier' => 'app',
            'environment' => 'production'
        ]
    ]),
    'db' => HetznerLaravel::placementGroups()->create([
        'name' => 'db-tier-anti-affinity',
        'type' => 'anti_affinity',
        'labels' => [
            'tier' => 'database',
            'environment' => 'production'
        ]
    ])
];

// Create servers for each tier
$servers = [
    'web' => [],
    'app' => [],
    'db' => []
];

// Web tier servers (spread across hosts)
for ($i = 1; $i <= 3; $i++) {
    $servers['web'][] = HetznerLaravel::servers()->create([
        'name' => "web-server-{$i}",
        'server_type' => 'cpx21',
        'image' => 'ubuntu-24.04',
        'location' => 'nbg1',
        'placement_group' => $placementGroups['web']->id,
        'ssh_keys' => ['my-ssh-key'],
        'labels' => [
            'tier' => 'web',
            'environment' => 'production'
        ]
    ]);
}

// App tier servers (spread across hosts)
for ($i = 1; $i <= 2; $i++) {
    $servers['app'][] = HetznerLaravel::servers()->create([
        'name' => "app-server-{$i}",
        'server_type' => 'cpx31',
        'image' => 'ubuntu-24.04',
        'location' => 'nbg1',
        'placement_group' => $placementGroups['app']->id,
        'ssh_keys' => ['my-ssh-key'],
        'labels' => [
            'tier' => 'app',
            'environment' => 'production'
        ]
    ]);
}

// Database tier servers (anti-affinity)
for ($i = 1; $i <= 2; $i++) {
    $servers['db'][] = HetznerLaravel::servers()->create([
        'name' => "db-server-{$i}",
        'server_type' => 'cpx41',
        'image' => 'ubuntu-24.04',
        'location' => 'nbg1',
        'placement_group' => $placementGroups['db']->id,
        'ssh_keys' => ['my-ssh-key'],
        'labels' => [
            'tier' => 'database',
            'environment' => 'production'
        ]
    ]);
}
```

### 4. Placement Group Management Service

```php
<?php

namespace App\Services;

use Boci\HetznerLaravel\Facades\HetznerLaravel;

class PlacementGroupManagementService
{
    public function createPlacementGroup(string $name, string $type, array $labels = []): array
    {
        try {
            $placementGroup = HetznerLaravel::placementGroups()->create([
                'name' => $name,
                'type' => $type,
                'labels' => $labels,
            ]);
            
            return [
                'success' => true,
                'placement_group' => $placementGroup->toArray(),
                'message' => 'Placement group created successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function getPlacementGroup(string $placementGroupId): array
    {
        try {
            $placementGroup = HetznerLaravel::placementGroups()->retrieve($placementGroupId);
            
            return [
                'success' => true,
                'placement_group' => $placementGroup->toArray()
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function listPlacementGroups(array $filters = []): array
    {
        try {
            $placementGroups = HetznerLaravel::placementGroups()->list($filters);
            
            return [
                'success' => true,
                'placement_groups' => $placementGroups->toArray(),
                'count' => count($placementGroups)
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function updatePlacementGroup(string $placementGroupId, array $data): array
    {
        try {
            $placementGroup = HetznerLaravel::placementGroups()->update($placementGroupId, $data);
            
            return [
                'success' => true,
                'placement_group' => $placementGroup->toArray(),
                'message' => 'Placement group updated successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function deletePlacementGroup(string $placementGroupId): array
    {
        try {
            HetznerLaravel::placementGroups()->delete($placementGroupId);
            
            return [
                'success' => true,
                'message' => 'Placement group deleted successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function createServersInPlacementGroup(string $placementGroupId, int $count, array $serverConfig): array
    {
        try {
            $servers = [];
            
            for ($i = 1; $i <= $count; $i++) {
                $server = HetznerLaravel::servers()->create(array_merge($serverConfig, [
                    'name' => $serverConfig['name'] . "-{$i}",
                    'placement_group' => $placementGroupId,
                ]));
                $servers[] = $server;
            }
            
            return [
                'success' => true,
                'servers' => $servers,
                'count' => count($servers),
                'message' => "Created {$count} servers in placement group"
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

### 5. High Availability Setup

```php
// Create anti-affinity placement group for high availability
$haPlacementGroup = HetznerLaravel::placementGroups()->create([
    'name' => 'ha-servers-anti-affinity',
    'type' => 'anti_affinity',
    'labels' => [
        'purpose' => 'high-availability',
        'environment' => 'production',
        'strategy' => 'anti-affinity'
    ]
]);

// Create load balancer
$loadBalancer = HetznerLaravel::loadBalancers()->create([
    'name' => 'ha-load-balancer',
    'load_balancer_type' => 'lb11',
    'location' => 'nbg1',
    'services' => [
        [
            'protocol' => 'http',
            'listen_port' => 80,
            'destination_port' => 80,
        ]
    ],
    'labels' => [
        'purpose' => 'high-availability',
        'environment' => 'production'
    ]
]);

// Create servers in anti-affinity placement group
$haServers = [];
for ($i = 1; $i <= 3; $i++) {
    $server = HetznerLaravel::servers()->create([
        'name' => "ha-server-{$i}",
        'server_type' => 'cpx21',
        'image' => 'ubuntu-24.04',
        'location' => 'nbg1',
        'placement_group' => $haPlacementGroup->id,
        'ssh_keys' => ['my-ssh-key'],
        'labels' => [
            'purpose' => 'high-availability',
            'environment' => 'production'
        ]
    ]);
    
    // Add server to load balancer
    HetznerLaravel::loadBalancers()->actions()->addTarget($loadBalancer->id, [
        'type' => 'server',
        'server' => [
            'id' => $server->id,
        ],
    ]);
    
    $haServers[] = $server;
}
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $placementGroup = HetznerLaravel::placementGroups()->create($parameters);
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

- [Servers API](./servers) - Create servers in placement groups
- [Load Balancers API](./load-balancers) - Distribute traffic across servers in placement groups
- [Networks API](./networks) - Configure networks for servers in placement groups
- [Firewalls API](./firewalls) - Secure servers in placement groups
