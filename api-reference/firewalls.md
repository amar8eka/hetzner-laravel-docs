---
sidebar_position: 5
---

# Firewalls API Reference

The Firewalls API allows you to manage Hetzner Cloud firewalls for securing your servers and networks.

## Available Methods

### `list(array $parameters = [])`

Get a list of all firewalls.

**Parameters:**
- `name` (string, optional): Filter by firewall name
- `label_selector` (string, optional): Filter by label selector
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all firewalls
$firewalls = HetznerLaravel::firewalls()->list();

// Get firewalls with filters
$firewalls = HetznerLaravel::firewalls()->list([
    'name' => 'web-firewall',
    'per_page' => 50,
]);
```

### `create(array $parameters)`

Create a new firewall.

**Required Parameters:**
- `name` (string): Firewall name

**Optional Parameters:**
- `rules` (array): Array of firewall rules
- `labels` (array): Key-value pairs for labeling
- `apply_to` (array): Array of resources to apply firewall to

**Example:**
```php
$firewall = HetznerLaravel::firewalls()->create([
    'name' => 'my-firewall',
    'rules' => [
        [
            'direction' => 'in',
            'source_ips' => ['0.0.0.0/0'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '80',
            'description' => 'Allow HTTP traffic',
        ],
        [
            'direction' => 'in',
            'source_ips' => ['0.0.0.0/0'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '443',
            'description' => 'Allow HTTPS traffic',
        ],
        [
            'direction' => 'in',
            'source_ips' => ['0.0.0.0/0'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '22',
            'description' => 'Allow SSH traffic',
        ],
    ],
    'labels' => [
        'environment' => 'production',
        'purpose' => 'web-security'
    ],
]);
```

### `retrieve(string $firewallId)`

Get details of a specific firewall.

**Parameters:**
- `firewallId` (string): Firewall ID

**Example:**
```php
$firewall = HetznerLaravel::firewalls()->retrieve('12345');
echo $firewall->name; // Firewall name
echo count($firewall->rules); // Number of rules
```

### `update(string $firewallId, array $parameters)`

Update firewall properties.

**Parameters:**
- `firewallId` (string): Firewall ID
- `name` (string, optional): New firewall name
- `labels` (array, optional): New labels

**Example:**
```php
$firewall = HetznerLaravel::firewalls()->update('12345', [
    'name' => 'updated-firewall',
    'labels' => [
        'environment' => 'staging',
        'updated' => 'true'
    ]
]);
```

### `delete(string $firewallId)`

Delete a firewall.

**Parameters:**
- `firewallId` (string): Firewall ID

**Example:**
```php
HetznerLaravel::firewalls()->delete('12345');
```

## Firewall Actions

### `actions()`

Access firewall actions for managing firewall rules and applying to resources.

**Example:**
```php
// Set firewall rules
$action = HetznerLaravel::firewalls()->actions()->setRules('12345', [
    'rules' => [
        [
            'direction' => 'in',
            'source_ips' => ['0.0.0.0/0'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '80',
            'description' => 'Allow HTTP',
        ]
    ]
]);

// Apply firewall to server
$action = HetznerLaravel::firewalls()->actions()->applyToResources('12345', [
    'apply_to' => [
        [
            'type' => 'server',
            'server' => [
                'id' => 67890,
            ],
        ]
    ]
]);

// Remove firewall from server
$action = HetznerLaravel::firewalls()->actions()->removeFromResources('12345', [
    'remove_from' => [
        [
            'type' => 'server',
            'server' => [
                'id' => 67890,
            ],
        ]
    ]
]);
```

## Response Objects

### Firewall Object
- `id` (int): Firewall ID
- `name` (string): Firewall name
- `created` (string): Creation timestamp
- `rules` (array): Firewall rules
- `applied_to` (array): Applied resources
- `labels` (object): Firewall labels

### Firewall Rule Object
- `direction` (string): Rule direction ('in', 'out')
- `source_ips` (array): Source IP addresses
- `destination_ips` (array): Destination IP addresses
- `protocol` (string): Protocol ('tcp', 'udp', 'icmp', 'esp', 'gre')
- `port` (string): Port or port range
- `description` (string): Rule description

## Common Use Cases

### 1. Web Server Firewall

```php
$firewall = HetznerLaravel::firewalls()->create([
    'name' => 'web-server-firewall',
    'rules' => [
        [
            'direction' => 'in',
            'source_ips' => ['0.0.0.0/0'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '80',
            'description' => 'Allow HTTP traffic',
        ],
        [
            'direction' => 'in',
            'source_ips' => ['0.0.0.0/0'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '443',
            'description' => 'Allow HTTPS traffic',
        ],
        [
            'direction' => 'in',
            'source_ips' => ['10.0.0.0/8'], // Only from private networks
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '22',
            'description' => 'Allow SSH from private networks',
        ],
        [
            'direction' => 'in',
            'source_ips' => ['0.0.0.0/0'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '3306',
            'description' => 'Allow MySQL from anywhere (consider restricting)',
        ],
    ],
    'labels' => [
        'role' => 'web-server',
        'environment' => 'production'
    ]
]);
```

### 2. Database Server Firewall

```php
$firewall = HetznerLaravel::firewalls()->create([
    'name' => 'database-server-firewall',
    'rules' => [
        [
            'direction' => 'in',
            'source_ips' => ['10.0.0.0/8'], // Only from private networks
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '22',
            'description' => 'Allow SSH from private networks',
        ],
        [
            'direction' => 'in',
            'source_ips' => ['10.0.0.0/8'], // Only from private networks
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '3306',
            'description' => 'Allow MySQL from private networks only',
        ],
        [
            'direction' => 'in',
            'source_ips' => ['10.0.0.0/8'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '5432',
            'description' => 'Allow PostgreSQL from private networks only',
        ],
    ],
    'labels' => [
        'role' => 'database-server',
        'environment' => 'production'
    ]
]);
```

### 3. Load Balancer Firewall

```php
$firewall = HetznerLaravel::firewalls()->create([
    'name' => 'load-balancer-firewall',
    'rules' => [
        [
            'direction' => 'in',
            'source_ips' => ['0.0.0.0/0'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '80',
            'description' => 'Allow HTTP traffic',
        ],
        [
            'direction' => 'in',
            'source_ips' => ['0.0.0.0/0'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '443',
            'description' => 'Allow HTTPS traffic',
        ],
        [
            'direction' => 'in',
            'source_ips' => ['10.0.0.0/8'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '22',
            'description' => 'Allow SSH from private networks',
        ],
    ],
    'labels' => [
        'role' => 'load-balancer',
        'environment' => 'production'
    ]
]);
```

### 4. Apply Firewall to Multiple Servers

```php
// Create firewall
$firewall = HetznerLaravel::firewalls()->create([
    'name' => 'web-tier-firewall',
    'rules' => [
        [
            'direction' => 'in',
            'source_ips' => ['0.0.0.0/0'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '80',
            'description' => 'Allow HTTP',
        ],
        [
            'direction' => 'in',
            'source_ips' => ['0.0.0.0/0'],
            'destination_ips' => [],
            'protocol' => 'tcp',
            'port' => '443',
            'description' => 'Allow HTTPS',
        ],
    ],
]);

// Apply to multiple servers
$serverIds = [12345, 67890, 11111];
foreach ($serverIds as $serverId) {
    HetznerLaravel::firewalls()->actions()->applyToResources($firewall->id, [
        'apply_to' => [
            [
                'type' => 'server',
                'server' => [
                    'id' => $serverId,
                ],
            ]
        ]
    ]);
}
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $firewall = HetznerLaravel::firewalls()->create($parameters);
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

- [Servers API](./servers) - Apply firewalls to servers
- [Networks API](./networks) - Configure private networks
- [Load Balancers API](./load-balancers) - Secure load balancer traffic
- [Floating IPs API](./floating-ips) - Manage external IP addresses
