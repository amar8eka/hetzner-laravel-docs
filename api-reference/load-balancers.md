---
sidebar_position: 4
---

# Load Balancers API Reference

The Load Balancers API allows you to manage Hetzner Cloud load balancers for distributing traffic across multiple servers.

## Available Methods

### `list(array $parameters = [])`

Get a list of all load balancers.

**Parameters:**
- `name` (string, optional): Filter by load balancer name
- `label_selector` (string, optional): Filter by label selector
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all load balancers
$loadBalancers = HetznerLaravel::loadBalancers()->list();

// Get load balancers with filters
$loadBalancers = HetznerLaravel::loadBalancers()->list([
    'name' => 'web-lb',
    'per_page' => 50,
]);
```

### `create(array $parameters)`

Create a new load balancer.

**Required Parameters:**
- `name` (string): Load balancer name
- `load_balancer_type` (string): Load balancer type (e.g., 'lb11', 'lb21')
- `location` (string): Location name or ID

**Optional Parameters:**
- `targets` (array): Array of target configurations
- `services` (array): Array of service configurations
- `labels` (array): Key-value pairs for labeling
- `public_interface` (object): Public interface configuration
- `network` (int): Private network ID
- `network_zone` (string): Network zone

**Example:**
```php
$loadBalancer = HetznerLaravel::loadBalancers()->create([
    'name' => 'my-load-balancer',
    'load_balancer_type' => 'lb11',
    'location' => 'nbg1',
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
            'proxyprotocol' => false,
            'http' => [
                'cookie_name' => 'HCLBSTICKY',
                'cookie_lifetime' => 300,
                'redirect_http' => true,
                'sticky_sessions' => true,
            ],
        ]
    ],
    'labels' => [
        'environment' => 'production',
        'purpose' => 'web-traffic'
    ],
]);
```

### `retrieve(string $loadBalancerId)`

Get details of a specific load balancer.

**Parameters:**
- `loadBalancerId` (string): Load balancer ID

**Example:**
```php
$loadBalancer = HetznerLaravel::loadBalancers()->retrieve('12345');
echo $loadBalancer->name; // Load balancer name
echo $loadBalancer->public_net->ipv4->ip; // Public IP
```

### `update(string $loadBalancerId, array $parameters)`

Update load balancer properties.

**Parameters:**
- `loadBalancerId` (string): Load balancer ID
- `name` (string, optional): New load balancer name
- `labels` (array, optional): New labels

**Example:**
```php
$loadBalancer = HetznerLaravel::loadBalancers()->update('12345', [
    'name' => 'updated-load-balancer',
    'labels' => [
        'environment' => 'staging',
        'updated' => 'true'
    ]
]);
```

### `delete(string $loadBalancerId)`

Delete a load balancer.

**Parameters:**
- `loadBalancerId` (string): Load balancer ID

**Example:**
```php
HetznerLaravel::loadBalancers()->delete('12345');
```

## Load Balancer Actions

### `actions()`

Access load balancer actions for performing operations like adding targets, updating services, etc.

**Example:**
```php
// Add target to load balancer
$action = HetznerLaravel::loadBalancers()->actions()->addTarget('12345', [
    'type' => 'server',
    'server' => [
        'id' => 67890,
    ],
]);

// Remove target from load balancer
$action = HetznerLaravel::loadBalancers()->actions()->removeTarget('12345', [
    'type' => 'server',
    'server' => [
        'id' => 67890,
    ],
]);

// Update service
$action = HetznerLaravel::loadBalancers()->actions()->updateService('12345', [
    'listen_port' => 80,
    'destination_port' => 80,
    'protocol' => 'http',
    'http' => [
        'sticky_sessions' => true,
        'cookie_name' => 'HCLBSTICKY',
        'cookie_lifetime' => 300,
    ],
]);

// Delete service
$action = HetznerLaravel::loadBalancers()->actions()->deleteService('12345', [
    'listen_port' => 80,
]);

// Change protection
$action = HetznerLaravel::loadBalancers()->actions()->changeProtection('12345', [
    'delete' => true
]);
```

## Response Objects

### Load Balancer Object
- `id` (int): Load balancer ID
- `name` (string): Load balancer name
- `created` (string): Creation timestamp
- `load_balancer_type` (object): Load balancer type details
- `location` (object): Location information
- `public_net` (object): Public network configuration
- `private_net` (array): Private network configuration
- `targets` (array): Target configurations
- `services` (array): Service configurations
- `algorithm` (object): Load balancing algorithm
- `included_traffic` (int): Included traffic
- `outgoing_traffic` (int): Outgoing traffic limit
- `ingoing_traffic` (int): Incoming traffic limit
- `protection` (object): Protection settings
- `labels` (object): Load balancer labels

### Target Object
- `type` (string): Target type ('server', 'label_selector', 'ip')
- `server` (object): Server details (if type is 'server')
- `label_selector` (object): Label selector (if type is 'label_selector')
- `ip` (object): IP details (if type is 'ip')
- `use_private_ip` (bool): Use private IP
- `health_status` (array): Health status for each target

### Service Object
- `protocol` (string): Service protocol ('http', 'https', 'tcp')
- `listen_port` (int): Listen port
- `destination_port` (int): Destination port
- `proxyprotocol` (bool): Enable PROXY protocol
- `http` (object): HTTP-specific configuration
- `https` (object): HTTPS-specific configuration
- `health_check` (object): Health check configuration

## Common Use Cases

### 1. Create a Web Load Balancer

```php
$loadBalancer = HetznerLaravel::loadBalancers()->create([
    'name' => 'web-load-balancer',
    'load_balancer_type' => 'lb11',
    'location' => 'nbg1',
    'targets' => [
        [
            'type' => 'server',
            'server' => [
                'id' => 12345, // Web server 1
            ],
        ],
        [
            'type' => 'server',
            'server' => [
                'id' => 67890, // Web server 2
            ],
        ]
    ],
    'services' => [
        [
            'protocol' => 'http',
            'listen_port' => 80,
            'destination_port' => 80,
            'http' => [
                'sticky_sessions' => true,
                'cookie_name' => 'HCLBSTICKY',
                'cookie_lifetime' => 300,
                'redirect_http' => true,
            ],
        ],
        [
            'protocol' => 'https',
            'listen_port' => 443,
            'destination_port' => 80,
            'https' => [
                'certificates' => [12345], // SSL certificate ID
            ],
        ]
    ],
    'labels' => [
        'role' => 'web-load-balancer',
        'environment' => 'production'
    ]
]);
```

### 2. Load Balancer with Label Selector Targets

```php
$loadBalancer = HetznerLaravel::loadBalancers()->create([
    'name' => 'auto-scaling-lb',
    'load_balancer_type' => 'lb21',
    'location' => 'nbg1',
    'targets' => [
        [
            'type' => 'label_selector',
            'label_selector' => [
                'selector' => 'role=web-server,environment=production',
            ],
        ]
    ],
    'services' => [
        [
            'protocol' => 'http',
            'listen_port' => 80,
            'destination_port' => 80,
            'http' => [
                'sticky_sessions' => false,
            ],
        ]
    ],
]);
```

### 3. Database Load Balancer

```php
$loadBalancer = HetznerLaravel::loadBalancers()->create([
    'name' => 'database-load-balancer',
    'load_balancer_type' => 'lb11',
    'location' => 'nbg1',
    'targets' => [
        [
            'type' => 'server',
            'server' => [
                'id' => 11111, // Database server 1
            ],
        ],
        [
            'type' => 'server',
            'server' => [
                'id' => 22222, // Database server 2
            ],
        ]
    ],
    'services' => [
        [
            'protocol' => 'tcp',
            'listen_port' => 3306,
            'destination_port' => 3306,
            'health_check' => [
                'protocol' => 'tcp',
                'port' => 3306,
                'interval' => 10,
                'timeout' => 5,
                'retries' => 3,
            ],
        ]
    ],
    'labels' => [
        'role' => 'database-load-balancer',
        'environment' => 'production'
    ]
]);
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $loadBalancer = HetznerLaravel::loadBalancers()->create($parameters);
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

- [Servers API](./servers) - Manage servers that can be load balanced
- [Networks API](./networks) - Configure private networks for load balancers
- [Certificates API](./certificates) - Manage SSL certificates for HTTPS services
- [Firewalls API](./firewalls) - Secure load balancer traffic
