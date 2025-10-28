---
sidebar_position: 15
---

# Load Balancer Types API Reference

The Load Balancer Types API allows you to retrieve information about available Hetzner Cloud load balancer types.

## Available Methods

### `list(array $parameters = [])`

Get a list of all load balancer types.

**Parameters:**
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all load balancer types
$loadBalancerTypes = HetznerLaravel::loadBalancerTypes()->list();

// Get load balancer types with pagination
$loadBalancerTypes = HetznerLaravel::loadBalancerTypes()->list([
    'per_page' => 50,
]);
```

### `retrieve(string $loadBalancerTypeId)`

Get details of a specific load balancer type.

**Parameters:**
- `loadBalancerTypeId` (string): Load balancer type ID or name

**Example:**
```php
$loadBalancerType = HetznerLaravel::loadBalancerTypes()->retrieve('lb11');
echo $loadBalancerType->name; // Load balancer type name
echo $loadBalancerType->description; // Load balancer type description
echo $loadBalancerType->max_connections; // Maximum connections
echo $loadBalancerType->max_services; // Maximum services
```

## Response Objects

### Load Balancer Type Object
- `id` (int): Load balancer type ID
- `name` (string): Load balancer type name
- `description` (string): Load balancer type description
- `max_connections` (int): Maximum connections
- `max_services` (int): Maximum services
- `max_targets` (int): Maximum targets
- `max_assigned_certificates` (int): Maximum assigned certificates
- `prices` (array): Pricing information

## Common Use Cases

### 1. List All Available Load Balancer Types

```php
$loadBalancerTypes = HetznerLaravel::loadBalancerTypes()->list();

echo "Available Load Balancer Types:\n";
echo "==============================\n";

foreach ($loadBalancerTypes as $type) {
    echo sprintf("%-10s: %s (Max: %d connections, %d services)\n", 
        $type->name, 
        $type->description, 
        $type->max_connections, 
        $type->max_services);
}
```

### 2. Get Load Balancer Type Information

```php
$loadBalancerType = HetznerLaravel::loadBalancerTypes()->retrieve('lb21');

echo "Load Balancer Type Details:\n";
echo "===========================\n";
echo "Name: {$loadBalancerType->name}\n";
echo "Description: {$loadBalancerType->description}\n";
echo "Max Connections: {$loadBalancerType->max_connections}\n";
echo "Max Services: {$loadBalancerType->max_services}\n";
echo "Max Targets: {$loadBalancerType->max_targets}\n";
echo "Max Certificates: {$loadBalancerType->max_assigned_certificates}\n";
```

### 3. Find Load Balancer Types by Capacity

```php
$loadBalancerTypes = HetznerLaravel::loadBalancerTypes()->list();
$minConnections = 1000;

$highCapacityTypes = array_filter($loadBalancerTypes, function($type) use ($minConnections) {
    return $type->max_connections >= $minConnections;
});

echo "Load Balancer Types with {$minConnections}+ Connections:\n";
echo "=======================================================\n";

foreach ($highCapacityTypes as $type) {
    echo "{$type->name}: {$type->max_connections} connections, {$type->max_services} services\n";
}
```

### 4. Compare Load Balancer Types

```php
$loadBalancerTypes = HetznerLaravel::loadBalancerTypes()->list();

echo "Load Balancer Type Comparison:\n";
echo "==============================\n";
echo sprintf("%-10s %12s %12s %12s %12s\n", "Name", "Connections", "Services", "Targets", "Certificates");
echo str_repeat("-", 60) . "\n";

foreach ($loadBalancerTypes as $type) {
    echo sprintf("%-10s %12d %12d %12d %12d\n", 
        $type->name, 
        $type->max_connections, 
        $type->max_services, 
        $type->max_targets, 
        $type->max_assigned_certificates);
}
```

### 5. Select Load Balancer Type Based on Requirements

```php
function selectLoadBalancerType(int $minConnections, int $minServices, int $minTargets): ?object
{
    $loadBalancerTypes = HetznerLaravel::loadBalancerTypes()->list();
    
    $suitableTypes = array_filter($loadBalancerTypes, function($type) use ($minConnections, $minServices, $minTargets) {
        return $type->max_connections >= $minConnections && 
               $type->max_services >= $minServices && 
               $type->max_targets >= $minTargets;
    });
    
    if (empty($suitableTypes)) {
        return null;
    }
    
    // Return the smallest suitable type
    usort($suitableTypes, function($a, $b) {
        return ($a->max_connections + $a->max_services + $a->max_targets) <=> 
               ($b->max_connections + $b->max_services + $b->max_targets);
    });
    
    return $suitableTypes[0];
}

// Example usage
$selectedType = selectLoadBalancerType(1000, 5, 10);

if ($selectedType) {
    echo "Selected load balancer type: {$selectedType->name}\n";
    echo "Specifications: {$selectedType->max_connections} connections, {$selectedType->max_services} services, {$selectedType->max_targets} targets\n";
} else {
    echo "No suitable load balancer type found\n";
}
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $loadBalancerTypes = HetznerLaravel::loadBalancerTypes()->list();
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

- [Load Balancers API](./load-balancers) - Create load balancers with specific types
- [Billing API](./billing) - Get pricing information for load balancer types
- [Locations API](./locations) - Create load balancers in specific locations
- [Servers API](./servers) - Create servers for load balancer targets
