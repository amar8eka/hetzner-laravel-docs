---
sidebar_position: 3
---

# Networks API Reference

The Networks API allows you to manage private networks and subnets in Hetzner Cloud. This includes creating networks, managing subnets, and configuring network settings.

## Available Methods

### `list(array $parameters = [])`

Get a list of all networks.

**Parameters:**
- `name` (string, optional): Filter by network name
- `label_selector` (string, optional): Filter by label selector
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all networks
$networks = HetznerLaravel::networks()->list();

// Get networks with filters
$networks = HetznerLaravel::networks()->list([
    'name' => 'private-network',
    'per_page' => 50,
]);
```

### `create(array $parameters)`

Create a new private network.

**Required Parameters:**
- `name` (string): Network name
- `ip_range` (string): IP range for the network (e.g., '10.0.0.0/16')

**Optional Parameters:**
- `subnets` (array): Array of subnet configurations
- `labels` (array): Key-value pairs for labeling

**Example:**
```php
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
    'labels' => [
        'environment' => 'production',
        'purpose' => 'web-servers'
    ],
]);
```

### `retrieve(string $networkId)`

Get details of a specific network.

**Parameters:**
- `networkId` (string): Network ID

**Example:**
```php
$network = HetznerLaravel::networks()->retrieve('12345');
echo $network->name; // Network name
echo $network->ip_range; // IP range
```

### `update(string $networkId, array $parameters)`

Update network properties.

**Parameters:**
- `networkId` (string): Network ID
- `name` (string, optional): New network name
- `labels` (array, optional): New labels

**Example:**
```php
$network = HetznerLaravel::networks()->update('12345', [
    'name' => 'updated-network-name',
    'labels' => [
        'environment' => 'staging',
        'updated' => 'true'
    ]
]);
```

### `delete(string $networkId)`

Delete a network.

**Parameters:**
- `networkId` (string): Network ID

**Example:**
```php
HetznerLaravel::networks()->delete('12345');
```

## Network Actions

### `actions()`

Access network actions for performing operations like changing protection.

**Example:**
```php
// Change network protection
$action = HetznerLaravel::networks()->actions()->changeProtection('12345', [
    'delete' => true
]);
```

## Response Objects

### Network Object
- `id` (int): Network ID
- `name` (string): Network name
- `ip_range` (string): IP range
- `created` (string): Creation timestamp
- `subnets` (array): Subnet configurations
- `routes` (array): Network routes
- `servers` (array): Attached servers
- `load_balancers` (array): Associated load balancers
- `protection` (object): Protection settings
- `labels` (object): Network labels

### Subnet Object
- `type` (string): Subnet type ('cloud', 'vswitch')
- `ip_range` (string): Subnet IP range
- `network_zone` (string): Network zone
- `gateway` (string): Gateway IP
- `vswitch_id` (int): VSwitch ID (for vswitch type)

## Common Use Cases

### 1. Create a Multi-Zone Network

```php
$network = HetznerLaravel::networks()->create([
    'name' => 'multi-zone-network',
    'ip_range' => '10.0.0.0/16',
    'subnets' => [
        [
            'type' => 'cloud',
            'ip_range' => '10.0.1.0/24',
            'network_zone' => 'eu-central',
        ],
        [
            'type' => 'cloud',
            'ip_range' => '10.0.2.0/24',
            'network_zone' => 'eu-central',
        ]
    ],
    'labels' => [
        'environment' => 'production',
        'multi-zone' => 'true'
    ]
]);
```

### 2. Create Servers in Private Network

```php
// First create the network
$network = HetznerLaravel::networks()->create([
    'name' => 'web-servers-network',
    'ip_range' => '10.0.0.0/16',
    'subnets' => [
        [
            'type' => 'cloud',
            'ip_range' => '10.0.1.0/24',
            'network_zone' => 'eu-central',
        ]
    ],
]);

// Then create servers in the network
$server = HetznerLaravel::servers()->create([
    'name' => 'web-server-1',
    'server_type' => 'cpx21',
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
    'networks' => [$network->id],
    'ssh_keys' => ['my-ssh-key'],
]);
```

### 3. Network Segmentation

```php
// Create separate networks for different tiers
$networks = [
    'web' => HetznerLaravel::networks()->create([
        'name' => 'web-tier',
        'ip_range' => '10.1.0.0/16',
        'subnets' => [
            [
                'type' => 'cloud',
                'ip_range' => '10.1.1.0/24',
                'network_zone' => 'eu-central',
            ]
        ],
    ]),
    'app' => HetznerLaravel::networks()->create([
        'name' => 'app-tier',
        'ip_range' => '10.2.0.0/16',
        'subnets' => [
            [
                'type' => 'cloud',
                'ip_range' => '10.2.1.0/24',
                'network_zone' => 'eu-central',
            ]
        ],
    ]),
    'db' => HetznerLaravel::networks()->create([
        'name' => 'db-tier',
        'ip_range' => '10.3.0.0/16',
        'subnets' => [
            [
                'type' => 'cloud',
                'ip_range' => '10.3.1.0/24',
                'network_zone' => 'eu-central',
            ]
        ],
    ]),
];
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $network = HetznerLaravel::networks()->create($parameters);
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

- [Servers API](./servers) - Create servers in networks
- [Load Balancers API](./load-balancers) - Create load balancers in networks
- [Firewalls API](./firewalls) - Secure network traffic
- [Floating IPs API](./floating-ips) - Manage external IP addresses
