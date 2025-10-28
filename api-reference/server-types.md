---
sidebar_position: 14
---

# Server Types API Reference

The Server Types API allows you to retrieve information about available Hetzner Cloud server types.

## Available Methods

### `list(array $parameters = [])`

Get a list of all server types.

**Parameters:**
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all server types
$serverTypes = HetznerLaravel::serverTypes()->list();

// Get server types with pagination
$serverTypes = HetznerLaravel::serverTypes()->list([
    'per_page' => 50,
]);
```

### `retrieve(string $serverTypeId)`

Get details of a specific server type.

**Parameters:**
- `serverTypeId` (string): Server type ID or name

**Example:**
```php
$serverType = HetznerLaravel::serverTypes()->retrieve('cpx11');
echo $serverType->name; // Server type name
echo $serverType->description; // Server type description
echo $serverType->cores; // Number of CPU cores
echo $serverType->memory; // Memory in GB
```

## Response Objects

### Server Type Object
- `id` (int): Server type ID
- `name` (string): Server type name
- `description` (string): Server type description
- `cores` (int): Number of CPU cores
- `memory` (float): Memory in GB
- `disk` (int): Disk size in GB
- `cpu_type` (string): CPU type
- `prices` (array): Pricing information
- `storage_type` (string): Storage type
- `network_zone` (string): Network zone

## Common Use Cases

### 1. List All Available Server Types

```php
$serverTypes = HetznerLaravel::serverTypes()->list();

echo "Available Server Types:\n";
echo "======================\n";

foreach ($serverTypes as $type) {
    echo sprintf("%-10s: %s (%d cores, %.1f GB RAM, %d GB disk)\n", 
        $type->name, 
        $type->description, 
        $type->cores, 
        $type->memory, 
        $type->disk);
}
```

### 2. Get Server Type Information

```php
$serverType = HetznerLaravel::serverTypes()->retrieve('cpx21');

echo "Server Type Details:\n";
echo "===================\n";
echo "Name: {$serverType->name}\n";
echo "Description: {$serverType->description}\n";
echo "CPU Cores: {$serverType->cores}\n";
echo "Memory: {$serverType->memory} GB\n";
echo "Disk: {$serverType->disk} GB\n";
echo "CPU Type: {$serverType->cpu_type}\n";
echo "Storage Type: {$serverType->storage_type}\n";
```

### 3. Find Server Types by CPU Cores

```php
$serverTypes = HetznerLaravel::serverTypes()->list();
$minCores = 2;

$multiCoreTypes = array_filter($serverTypes, function($type) use ($minCores) {
    return $type->cores >= $minCores;
});

echo "Server Types with {$minCores}+ CPU Cores:\n";
echo "=========================================\n";

foreach ($multiCoreTypes as $type) {
    echo "{$type->name}: {$type->cores} cores, {$type->memory} GB RAM\n";
}
```

### 4. Find Server Types by Memory

```php
$serverTypes = HetznerLaravel::serverTypes()->list();
$minMemory = 4; // GB

$highMemoryTypes = array_filter($serverTypes, function($type) use ($minMemory) {
    return $type->memory >= $minMemory;
});

echo "Server Types with {$minMemory}+ GB RAM:\n";
echo "======================================\n";

foreach ($highMemoryTypes as $type) {
    echo "{$type->name}: {$type->memory} GB RAM, {$type->cores} cores\n";
}
```

### 5. Compare Server Types

```php
$serverTypes = HetznerLaravel::serverTypes()->list();

echo "Server Type Comparison:\n";
echo "=======================\n";
echo sprintf("%-10s %8s %8s %8s %12s\n", "Name", "Cores", "RAM(GB)", "Disk(GB)", "CPU Type");
echo str_repeat("-", 50) . "\n";

foreach ($serverTypes as $type) {
    echo sprintf("%-10s %8d %8.1f %8d %12s\n", 
        $type->name, 
        $type->cores, 
        $type->memory, 
        $type->disk, 
        $type->cpu_type);
}
```

### 6. Select Server Type Based on Requirements

```php
function selectServerType(int $minCores, float $minMemory, int $minDisk): ?object
{
    $serverTypes = HetznerLaravel::serverTypes()->list();
    
    $suitableTypes = array_filter($serverTypes, function($type) use ($minCores, $minMemory, $minDisk) {
        return $type->cores >= $minCores && 
               $type->memory >= $minMemory && 
               $type->disk >= $minDisk;
    });
    
    if (empty($suitableTypes)) {
        return null;
    }
    
    // Return the smallest suitable type
    usort($suitableTypes, function($a, $b) {
        return ($a->cores + $a->memory + $a->disk) <=> ($b->cores + $b->memory + $b->disk);
    });
    
    return $suitableTypes[0];
}

// Example usage
$selectedType = selectServerType(2, 4.0, 20);

if ($selectedType) {
    echo "Selected server type: {$selectedType->name}\n";
    echo "Specifications: {$selectedType->cores} cores, {$selectedType->memory} GB RAM, {$selectedType->disk} GB disk\n";
} else {
    echo "No suitable server type found\n";
}
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $serverTypes = HetznerLaravel::serverTypes()->list();
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

- [Servers API](./servers) - Create servers with specific server types
- [Billing API](./billing) - Get pricing information for server types
- [Locations API](./locations) - Create servers in specific locations
- [Images API](./images) - Create servers with specific images
