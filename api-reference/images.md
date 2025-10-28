---
sidebar_position: 8
---

# Images API Reference

The Images API allows you to manage server images, including system images, snapshots, and custom images.

## Available Methods

### `list(array $parameters = [])`

Get a list of all images.

**Parameters:**
- `name` (string, optional): Filter by image name
- `label_selector` (string, optional): Filter by label selector
- `type` (string, optional): Filter by image type ('system', 'snapshot', 'backup')
- `status` (string, optional): Filter by image status
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all images
$images = HetznerLaravel::images()->list();

// Get system images only
$systemImages = HetznerLaravel::images()->list([
    'type' => 'system',
    'per_page' => 50,
]);

// Get images with filters
$images = HetznerLaravel::images()->list([
    'name' => 'ubuntu',
    'status' => 'available',
]);
```

### `retrieve(string $imageId)`

Get details of a specific image.

**Parameters:**
- `imageId` (string): Image ID

**Example:**
```php
$image = HetznerLaravel::images()->retrieve('12345');
echo $image->name; // Image name
echo $image->type; // Image type
echo $image->status; // Image status
```

### `update(string $imageId, array $parameters)`

Update image properties.

**Parameters:**
- `imageId` (string): Image ID
- `description` (string, optional): New image description
- `type` (string, optional): New image type
- `labels` (array, optional): New labels

**Example:**
```php
$image = HetznerLaravel::images()->update('12345', [
    'description' => 'Updated image description',
    'labels' => [
        'environment' => 'staging',
        'updated' => 'true'
    ]
]);
```

### `delete(string $imageId)`

Delete an image.

**Parameters:**
- `imageId` (string): Image ID

**Example:**
```php
HetznerLaravel::images()->delete('12345');
```

## Image Actions

### `actions()`

Access image actions for creating images from servers and other operations.

**Example:**
```php
// Create image from server
$action = HetznerLaravel::images()->actions()->createFromServer('12345', [
    'description' => 'Web server snapshot',
    'type' => 'snapshot',
    'labels' => [
        'environment' => 'production',
        'backup' => 'daily'
    ]
]);

// Change protection
$action = HetznerLaravel::images()->actions()->changeProtection('12345', [
    'delete' => true
]);
```

## Response Objects

### Image Object
- `id` (int): Image ID
- `name` (string): Image name
- `description` (string): Image description
- `type` (string): Image type ('system', 'snapshot', 'backup')
- `status` (string): Image status ('available', 'creating', 'unavailable')
- `created` (string): Creation timestamp
- `created_from` (object): Source server or image
- `bound_to` (int): Bound server ID
- `os_flavor` (string): Operating system flavor
- `os_version` (string): Operating system version
- `architecture` (string): Architecture ('x86', 'arm')
- `image_size` (float): Image size in GB
- `disk_size` (float): Disk size in GB
- `protection` (object): Protection settings
- `labels` (object): Image labels
- `deprecated` (string): Deprecation timestamp

## Common Use Cases

### 1. List Available System Images

```php
$systemImages = HetznerLaravel::images()->list([
    'type' => 'system',
    'status' => 'available',
]);

foreach ($systemImages as $image) {
    echo "{$image->name} - {$image->description} ({$image->os_flavor} {$image->os_version})\n";
}
```

### 2. Create Server Snapshot

```php
// Create snapshot from running server
$action = HetznerLaravel::images()->actions()->createFromServer('12345', [
    'description' => 'Production web server snapshot',
    'type' => 'snapshot',
    'labels' => [
        'environment' => 'production',
        'server_id' => '12345',
        'backup_date' => date('Y-m-d'),
    ]
]);

// Wait for action to complete
$image = HetznerLaravel::images()->retrieve($action->image->id);
echo "Snapshot created: {$image->name}";
```

### 3. Create Custom Image from Server

```php
// Create custom image from server
$action = HetznerLaravel::images()->actions()->createFromServer('12345', [
    'description' => 'Custom web server image',
    'type' => 'snapshot',
    'labels' => [
        'custom' => 'true',
        'environment' => 'production',
        'version' => '1.0',
    ]
]);

// Use the custom image to create new servers
$newServer = HetznerLaravel::servers()->create([
    'name' => 'new-web-server',
    'server_type' => 'cpx21',
    'image' => $action->image->id,
    'location' => 'nbg1',
    'ssh_keys' => ['my-ssh-key'],
]);
```

### 4. Image Backup Strategy

```php
// Create daily backup of production server
$productionServerId = 12345;
$backupImage = HetznerLaravel::images()->actions()->createFromServer($productionServerId, [
    'description' => 'Daily backup - ' . date('Y-m-d'),
    'type' => 'snapshot',
    'labels' => [
        'backup_type' => 'daily',
        'server_id' => $productionServerId,
        'backup_date' => date('Y-m-d'),
        'environment' => 'production',
    ]
]);

// Create weekly backup
$weeklyBackup = HetznerLaravel::images()->actions()->createFromServer($productionServerId, [
    'description' => 'Weekly backup - ' . date('Y-m-d'),
    'type' => 'snapshot',
    'labels' => [
        'backup_type' => 'weekly',
        'server_id' => $productionServerId,
        'backup_date' => date('Y-m-d'),
        'environment' => 'production',
    ]
]);
```

### 5. Restore Server from Image

```php
// Get the latest backup image
$backupImages = HetznerLaravel::images()->list([
    'type' => 'snapshot',
    'label_selector' => 'backup_type=daily',
    'per_page' => 1,
]);

if (!empty($backupImages)) {
    $latestBackup = $backupImages[0];
    
    // Create new server from backup
    $restoredServer = HetznerLaravel::servers()->create([
        'name' => 'restored-server-' . time(),
        'server_type' => 'cpx21',
        'image' => $latestBackup->id,
        'location' => 'nbg1',
        'ssh_keys' => ['my-ssh-key'],
        'labels' => [
            'restored_from' => $latestBackup->id,
            'restore_date' => date('Y-m-d'),
        ]
    ]);
}
```

### 6. Manage Image Lifecycle

```php
// List old backup images
$oldBackups = HetznerLaravel::images()->list([
    'type' => 'snapshot',
    'label_selector' => 'backup_type=daily',
    'per_page' => 100,
]);

// Keep only last 7 days of backups
$cutoffDate = date('Y-m-d', strtotime('-7 days'));

foreach ($oldBackups as $backup) {
    $backupDate = $backup->labels['backup_date'] ?? '';
    
    if ($backupDate && $backupDate < $cutoffDate) {
        // Delete old backup
        HetznerLaravel::images()->delete($backup->id);
        echo "Deleted old backup: {$backup->name}\n";
    }
}
```

### 7. Create Image from Volume

```php
// Create image from volume (if supported)
$volume = HetznerLaravel::volumes()->retrieve('12345');

// Note: This would typically be done through server actions
// First attach volume to server, then create image from server
$server = HetznerLaravel::servers()->create([
    'name' => 'image-creation-server',
    'server_type' => 'cpx11',
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
    'volumes' => [$volume->id],
    'ssh_keys' => ['my-ssh-key'],
]);

// Then create image from the server
$image = HetznerLaravel::images()->actions()->createFromServer($server->id, [
    'description' => 'Image created from volume',
    'type' => 'snapshot',
]);
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $image = HetznerLaravel::images()->create($parameters);
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

- [Servers API](./servers) - Create servers from images
- [Volumes API](./volumes) - Manage storage volumes
- [Actions API](./actions) - Monitor image creation progress
- [Firewalls API](./firewalls) - Secure image access
