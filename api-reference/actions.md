---
sidebar_position: 11
---

# Actions API Reference

The Actions API allows you to monitor and manage actions performed on Hetzner Cloud resources.

## Available Methods

### `list(array $parameters = [])`

Get a list of all actions.

**Parameters:**
- `status` (string, optional): Filter by action status
- `sort` (string, optional): Sort by field (id, command, status, progress, started, finished)
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all actions
$actions = HetznerLaravel::actions()->list();

// Get actions with filters
$actions = HetznerLaravel::actions()->list([
    'status' => 'running',
    'per_page' => 50,
]);
```

### `retrieve(string $actionId)`

Get details of a specific action.

**Parameters:**
- `actionId` (string): Action ID

**Example:**
```php
$action = HetznerLaravel::actions()->retrieve('12345');
echo $action->command; // Action command
echo $action->status; // Action status
echo $action->progress; // Action progress
```

## Response Objects

### Action Object
- `id` (int): Action ID
- `command` (string): Action command
- `status` (string): Action status ('running', 'success', 'error')
- `progress` (int): Action progress (0-100)
- `started` (string): Start timestamp
- `finished` (string): Finish timestamp
- `error` (object): Error details (if failed)
- `resources` (array): Related resources

## Common Use Cases

### 1. Monitor Server Creation

```php
// Create server
$server = HetznerLaravel::servers()->create([
    'name' => 'my-server',
    'server_type' => 'cpx11',
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
]);

// Monitor creation action
$action = HetznerLaravel::actions()->retrieve($server->action->id);

while ($action->status === 'running') {
    echo "Server creation progress: {$action->progress}%\n";
    sleep(5);
    $action = HetznerLaravel::actions()->retrieve($action->id);
}

if ($action->status === 'success') {
    echo "Server created successfully!\n";
} else {
    echo "Server creation failed: {$action->error->message}\n";
}
```

### 2. Monitor Volume Attachment

```php
// Attach volume to server
$action = HetznerLaravel::volumes()->actions()->attach('12345', [
    'server' => 67890,
    'automount' => true,
]);

// Monitor attachment progress
while ($action->status === 'running') {
    echo "Volume attachment progress: {$action->progress}%\n";
    sleep(2);
    $action = HetznerLaravel::actions()->retrieve($action->id);
}

if ($action->status === 'success') {
    echo "Volume attached successfully!\n";
} else {
    echo "Volume attachment failed: {$action->error->message}\n";
}
```

### 3. Monitor Image Creation

```php
// Create image from server
$action = HetznerLaravel::images()->actions()->createFromServer('12345', [
    'description' => 'Server snapshot',
    'type' => 'snapshot',
]);

// Monitor image creation progress
while ($action->status === 'running') {
    echo "Image creation progress: {$action->progress}%\n";
    sleep(10);
    $action = HetznerLaravel::actions()->retrieve($action->id);
}

if ($action->status === 'success') {
    echo "Image created successfully!\n";
} else {
    echo "Image creation failed: {$action->error->message}\n";
}
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $action = HetznerLaravel::actions()->retrieve($actionId);
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

- [Servers API](./servers) - Monitor server actions
- [Volumes API](./volumes) - Monitor volume actions
- [Images API](./images) - Monitor image actions
- [Load Balancers API](./load-balancers) - Monitor load balancer actions
