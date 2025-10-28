---
sidebar_position: 7
---

# Volumes API Reference

The Volumes API allows you to manage block storage volumes that can be attached to servers for persistent data storage.

## Available Methods

### `list(array $parameters = [])`

Get a list of all volumes.

**Parameters:**
- `name` (string, optional): Filter by volume name
- `label_selector` (string, optional): Filter by label selector
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all volumes
$volumes = HetznerLaravel::volumes()->list();

// Get volumes with filters
$volumes = HetznerLaravel::volumes()->list([
    'name' => 'database-volume',
    'per_page' => 50,
]);
```

### `create(array $parameters)`

Create a new volume.

**Required Parameters:**
- `name` (string): Volume name
- `size` (int): Volume size in GB (10-10240)
- `location` (string): Location name or ID

**Optional Parameters:**
- `server` (int): Server ID to attach to
- `automount` (bool): Automatically mount the volume
- `format` (string): Filesystem format ('xfs' or 'ext4')
- `labels` (array): Key-value pairs for labeling

**Example:**
```php
$volume = HetznerLaravel::volumes()->create([
    'name' => 'my-volume',
    'size' => 20,
    'location' => 'nbg1',
    'automount' => true,
    'format' => 'ext4',
    'labels' => [
        'environment' => 'production',
        'purpose' => 'data-storage'
    ],
]);
```

### `retrieve(string $volumeId)`

Get details of a specific volume.

**Parameters:**
- `volumeId` (string): Volume ID

**Example:**
```php
$volume = HetznerLaravel::volumes()->retrieve('12345');
echo $volume->name; // Volume name
echo $volume->size; // Volume size in GB
echo $volume->server->id; // Attached server ID
```

### `update(string $volumeId, array $parameters)`

Update volume properties.

**Parameters:**
- `volumeId` (string): Volume ID
- `name` (string, optional): New volume name
- `labels` (array, optional): New labels

**Example:**
```php
$volume = HetznerLaravel::volumes()->update('12345', [
    'name' => 'updated-volume',
    'labels' => [
        'environment' => 'staging',
        'updated' => 'true'
    ]
]);
```

### `delete(string $volumeId)`

Delete a volume.

**Parameters:**
- `volumeId` (string): Volume ID

**Example:**
```php
HetznerLaravel::volumes()->delete('12345');
```

## Volume Actions

### `actions()`

Access volume actions for attaching, detaching, and resizing volumes.

**Example:**
```php
// Attach volume to server
$action = HetznerLaravel::volumes()->actions()->attach('12345', [
    'server' => 67890, // Server ID
    'automount' => true,
]);

// Detach volume from server
$action = HetznerLaravel::volumes()->actions()->detach('12345');

// Resize volume
$action = HetznerLaravel::volumes()->actions()->resize('12345', [
    'size' => 50, // New size in GB
]);

// Change protection
$action = HetznerLaravel::volumes()->actions()->changeProtection('12345', [
    'delete' => true
]);
```

## Response Objects

### Volume Object
- `id` (int): Volume ID
- `name` (string): Volume name
- `size` (int): Volume size in GB
- `created` (string): Creation timestamp
- `location` (object): Volume location
- `server` (object): Attached server
- `linux_device` (string): Linux device path
- `protection` (object): Protection settings
- `labels` (object): Volume labels

## Common Use Cases

### 1. Create Database Volume

```php
$volume = HetznerLaravel::volumes()->create([
    'name' => 'database-volume',
    'size' => 100,
    'location' => 'nbg1',
    'automount' => true,
    'format' => 'ext4',
    'labels' => [
        'role' => 'database-storage',
        'environment' => 'production',
        'backup' => 'daily'
    ]
]);
```

### 2. Attach Volume to Server

```php
// Create volume
$volume = HetznerLaravel::volumes()->create([
    'name' => 'web-data-volume',
    'size' => 50,
    'location' => 'nbg1',
    'automount' => false,
]);

// Attach to server
HetznerLaravel::volumes()->actions()->attach($volume->id, [
    'server' => 12345, // Server ID
    'automount' => true,
]);
```

### 3. Create Server with Volume

```php
// First create the volume
$volume = HetznerLaravel::volumes()->create([
    'name' => 'app-server-volume',
    'size' => 30,
    'location' => 'nbg1',
    'automount' => true,
    'format' => 'ext4',
]);

// Then create server with the volume
$server = HetznerLaravel::servers()->create([
    'name' => 'app-server',
    'server_type' => 'cpx21',
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
    'volumes' => [$volume->id],
    'ssh_keys' => ['my-ssh-key'],
    'user_data' => '#cloud-config
runcmd:
  - mkdir -p /mnt/data
  - mount /dev/disk/by-id/scsi-0HC_Volume_' . $volume->id . ' /mnt/data
  - echo "/dev/disk/by-id/scsi-0HC_Volume_' . $volume->id . ' /mnt/data ext4 defaults 0 0" >> /etc/fstab',
]);
```

### 4. Resize Volume

```php
// Resize volume from 20GB to 50GB
$action = HetznerLaravel::volumes()->actions()->resize('12345', [
    'size' => 50,
]);

// Wait for action to complete, then resize filesystem
// (This would typically be done on the server)
```

### 5. Multiple Volumes for Different Purposes

```php
$volumes = [
    'database' => HetznerLaravel::volumes()->create([
        'name' => 'mysql-data-volume',
        'size' => 100,
        'location' => 'nbg1',
        'automount' => true,
        'format' => 'ext4',
        'labels' => [
            'purpose' => 'mysql-data',
            'environment' => 'production'
        ]
    ]),
    'logs' => HetznerLaravel::volumes()->create([
        'name' => 'application-logs-volume',
        'size' => 20,
        'location' => 'nbg1',
        'automount' => true,
        'format' => 'ext4',
        'labels' => [
            'purpose' => 'application-logs',
            'environment' => 'production'
        ]
    ]),
    'backups' => HetznerLaravel::volumes()->create([
        'name' => 'backup-storage-volume',
        'size' => 200,
        'location' => 'nbg1',
        'automount' => true,
        'format' => 'ext4',
        'labels' => [
            'purpose' => 'backup-storage',
            'environment' => 'production'
        ]
    ]),
];

// Attach all volumes to server
$server = HetznerLaravel::servers()->create([
    'name' => 'database-server',
    'server_type' => 'cpx31',
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
    'volumes' => array_column($volumes, 'id'),
    'ssh_keys' => ['my-ssh-key'],
]);
```

### 6. Volume Backup Strategy

```php
// Create production volume
$productionVolume = HetznerLaravel::volumes()->create([
    'name' => 'production-data-volume',
    'size' => 100,
    'location' => 'nbg1',
    'automount' => true,
    'format' => 'ext4',
    'labels' => [
        'environment' => 'production',
        'backup-strategy' => 'daily'
    ]
]);

// Create backup volume in different location
$backupVolume = HetznerLaravel::volumes()->create([
    'name' => 'backup-data-volume',
    'size' => 100,
    'location' => 'fsn1', // Different location for disaster recovery
    'automount' => false,
    'format' => 'ext4',
    'labels' => [
        'environment' => 'backup',
        'source-volume' => $productionVolume->id
    ]
]);
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $volume = HetznerLaravel::volumes()->create($parameters);
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

- [Servers API](./servers) - Attach volumes to servers
- [Images API](./images) - Create images from volumes
- [Networks API](./networks) - Configure private networks
- [Firewalls API](./firewalls) - Secure volume access
