---
sidebar_position: 1
---

# Servers API Reference

The Servers API allows you to manage Hetzner Cloud servers. This includes creating, updating, deleting, and monitoring servers.

## Available Methods

### `list(array $parameters = [])`

Get a list of all servers.

**Parameters:**
- `name` (string, optional): Filter by server name
- `label_selector` (string, optional): Filter by label selector
- `status` (string, optional): Filter by server status
- `sort` (string, optional): Sort by field (id, name, created)
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all servers
$servers = HetznerLaravel::servers()->list();

// Get servers with filters
$servers = HetznerLaravel::servers()->list([
    'name' => 'web-server',
    'status' => 'running',
    'per_page' => 50,
]);
```

### `create(array $parameters)`

Create a new server.

**Required Parameters:**
- `name` (string): Server name
- `server_type` (string): Server type (e.g., 'cpx11', 'cpx21')
- `image` (string): Image name or ID
- `location` (string): Location name or ID

**Optional Parameters:**
- `ssh_keys` (array): Array of SSH key names or IDs
- `user_data` (string): Cloud-init user data
- `networks` (array): Network IDs to attach
- `volumes` (array): Volume IDs to attach
- `firewalls` (array): Firewall IDs to apply
- `labels` (array): Key-value pairs for labeling
- `placement_group` (int): Placement group ID
- `public_net` (array): Public network configuration
- `automount` (bool): Automatically mount volumes

**Example:**
```php
$server = HetznerLaravel::servers()->create([
    'name' => 'my-web-server',
    'server_type' => 'cpx11',
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
    'ssh_keys' => ['my-ssh-key'],
    'user_data' => '#cloud-config
runcmd:
  - apt-get update
  - apt-get install -y nginx',
    'labels' => [
        'environment' => 'production',
        'role' => 'web-server'
    ],
]);
```

### `retrieve(string $serverId)`

Get details of a specific server.

**Parameters:**
- `serverId` (string): Server ID

**Example:**
```php
$server = HetznerLaravel::servers()->retrieve('12345');
echo $server->name; // Server name
echo $server->status; // Server status
echo $server->public_net->ipv4->ip; // Public IP
```

### `update(string $serverId, array $parameters)`

Update server properties.

**Parameters:**
- `serverId` (string): Server ID
- `name` (string, optional): New server name
- `labels` (array, optional): New labels

**Example:**
```php
$server = HetznerLaravel::servers()->update('12345', [
    'name' => 'updated-server-name',
    'labels' => [
        'environment' => 'staging',
        'updated' => 'true'
    ]
]);
```

### `delete(string $serverId)`

Delete a server.

**Parameters:**
- `serverId` (string): Server ID

**Example:**
```php
HetznerLaravel::servers()->delete('12345');
```

### `metrics(string $serverId, array $parameters)`

Get server metrics for monitoring and analysis.

**Parameters:**
- `serverId` (string): Server ID
- `type` (string, required): Metric type (`cpu`, `disk`, `network`)
- `start` (string, required): Start time in ISO 8601 format (e.g., `2024-01-01T00:00:00Z`)
- `end` (string, required): End time in ISO 8601 format (e.g., `2024-01-01T01:00:00Z`)

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get CPU metrics for the last hour
$metrics = HetznerLaravel::servers()->metrics('12345', [
    'type' => 'cpu',
    'start' => now()->subHour()->toIso8601ZuluString(),
    'end' => now()->toIso8601ZuluString(),
]);

// Return metrics as JSON response
return response()->json($metrics->toArray());
```

**Example in Controller:**
```php
public function getServerMetrics($serverId)
{
    $metrics = HetznerLaravel::servers()->metrics($serverId, [
        'type' => 'cpu',
        'start' => now()->subHour()->toIso8601ZuluString(),
        'end' => now()->toIso8601ZuluString(),
    ]);

    return response()->json($metrics->toArray());
}
```

**Available Metric Types:**
- `cpu`: CPU usage metrics
- `disk`: Disk I/O metrics
- `network`: Network traffic metrics

## Server Actions

The servers resource also provides access to server actions:

### `actions()`

Access server actions for performing operations like power on/off, reboot, etc.

**Example:**
```php
// Power on server
$action = HetznerLaravel::servers()->actions()->powerOn('12345');

// Reboot server
$action = HetznerLaravel::servers()->actions()->reboot('12345');

// Shutdown server
$action = HetznerLaravel::servers()->actions()->shutdown('12345');

// Reset server
$action = HetznerLaravel::servers()->actions()->reset('12345');
```

## Response Objects

All server methods return response objects with the following properties:

### Server Object
- `id` (int): Server ID
- `name` (string): Server name
- `status` (string): Server status (running, initializing, starting, stopping, off, deleting, migrating, unknown)
- `created` (string): Creation timestamp
- `public_net` (object): Public network configuration
- `private_net` (array): Private network configuration
- `server_type` (object): Server type details
- `datacenter` (object): Datacenter information
- `image` (object): Image details
- `iso` (object): ISO details (if attached)
- `rescue_enabled` (bool): Rescue mode status
- `locked` (bool): Server lock status
- `backup_window` (string): Backup window
- `outgoing_traffic` (int): Outgoing traffic limit
- `ingoing_traffic` (int): Incoming traffic limit
- `included_traffic` (int): Included traffic
- `protection` (object): Protection settings
- `labels` (object): Server labels
- `volumes` (array): Attached volumes
- `load_balancers` (array): Associated load balancers
- `placement_group` (object): Placement group details

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $server = HetznerLaravel::servers()->create($parameters);
} catch (ErrorException $e) {
    // Handle API errors (400, 401, 403, 404, etc.)
    echo "API Error: " . $e->getMessage();
    echo "Error Code: " . $e->getCode();
} catch (TransporterException $e) {
    // Handle network/transport errors
    echo "Network Error: " . $e->getMessage();
}
```

## Common Use Cases

### 1. Create a Web Server

```php
$server = HetznerLaravel::servers()->create([
    'name' => 'web-server-' . time(),
    'server_type' => 'cpx21', // 2 vCPU, 4 GB RAM
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
    'ssh_keys' => ['my-ssh-key'],
    'user_data' => '#cloud-config
packages:
  - nginx
  - php8.1-fpm
  - mysql-server
runcmd:
  - systemctl enable nginx
  - systemctl start nginx',
    'labels' => [
        'role' => 'web-server',
        'environment' => 'production'
    ]
]);
```

### 2. Create a Database Server

```php
$server = HetznerLaravel::servers()->create([
    'name' => 'db-server-' . time(),
    'server_type' => 'cpx31', // 4 vCPU, 8 GB RAM
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
    'ssh_keys' => ['my-ssh-key'],
    'volumes' => [12345], // Attach a volume for data storage
    'labels' => [
        'role' => 'database',
        'environment' => 'production'
    ]
]);
```

### 3. List Servers by Environment

```php
$productionServers = HetznerLaravel::servers()->list([
    'label_selector' => 'environment=production'
]);

$stagingServers = HetznerLaravel::servers()->list([
    'label_selector' => 'environment=staging'
]);
```

## Related Resources

- [Server Types API](./server-types) - Get available server types
- [Images API](./images) - Manage server images
- [SSH Keys API](./ssh-keys) - Manage SSH keys
- [Volumes API](./volumes) - Manage block storage
- [Networks API](./networks) - Manage private networks
- [Firewalls API](./firewalls) - Manage security rules
