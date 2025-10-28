---
sidebar_position: 13
---

# Locations API Reference

The Locations API allows you to retrieve information about Hetzner Cloud datacenter locations.

## Available Methods

### `list(array $parameters = [])`

Get a list of all locations.

**Parameters:**
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all locations
$locations = HetznerLaravel::locations()->list();

// Get locations with pagination
$locations = HetznerLaravel::locations()->list([
    'per_page' => 50,
]);
```

### `retrieve(string $locationId)`

Get details of a specific location.

**Parameters:**
- `locationId` (string): Location ID or name

**Example:**
```php
$location = HetznerLaravel::locations()->retrieve('nbg1');
echo $location->name; // Location name
echo $location->description; // Location description
echo $location->country; // Country
echo $location->city; // City
```

## Response Objects

### Location Object
- `id` (int): Location ID
- `name` (string): Location name
- `description` (string): Location description
- `country` (string): Country code
- `city` (string): City name
- `latitude` (float): Latitude coordinate
- `longitude` (float): Longitude coordinate
- `network_zone` (string): Network zone

## Common Use Cases

### 1. List All Available Locations

```php
$locations = HetznerLaravel::locations()->list();

echo "Available Hetzner Cloud Locations:\n";
echo "==================================\n";

foreach ($locations as $location) {
    echo sprintf("%-10s: %s (%s, %s)\n", 
        $location->name, 
        $location->description, 
        $location->city, 
        $location->country);
}
```

### 2. Get Location Information

```php
$location = HetznerLaravel::locations()->retrieve('nbg1');

echo "Location Details:\n";
echo "================\n";
echo "Name: {$location->name}\n";
echo "Description: {$location->description}\n";
echo "City: {$location->city}\n";
echo "Country: {$location->country}\n";
echo "Network Zone: {$location->network_zone}\n";
echo "Coordinates: {$location->latitude}, {$location->longitude}\n";
```

### 3. Find Locations by Country

```php
$locations = HetznerLaravel::locations()->list();
$country = 'DE'; // Germany

$germanLocations = array_filter($locations, function($location) use ($country) {
    return $location->country === $country;
});

echo "Locations in {$country}:\n";
echo "=======================\n";

foreach ($germanLocations as $location) {
    echo "{$location->name}: {$location->city}\n";
}
```

### 4. Create Servers in Different Locations

```php
$locations = HetznerLaravel::locations()->list();
$serverType = 'cpx11';
$image = 'ubuntu-24.04';

foreach ($locations as $location) {
    $server = HetznerLaravel::servers()->create([
        'name' => "server-{$location->name}",
        'server_type' => $serverType,
        'image' => $image,
        'location' => $location->name,
        'ssh_keys' => ['my-ssh-key'],
        'labels' => [
            'location' => $location->name,
            'city' => $location->city,
            'country' => $location->country,
        ]
    ]);
    
    echo "Created server in {$location->name} ({$location->city})\n";
}
```

### 5. Location-Based Load Balancing

```php
$locations = HetznerLaravel::locations()->list();
$loadBalancerType = 'lb11';

// Create load balancers in different locations
foreach ($locations as $location) {
    $loadBalancer = HetznerLaravel::loadBalancers()->create([
        'name' => "lb-{$location->name}",
        'load_balancer_type' => $loadBalancerType,
        'location' => $location->name,
        'targets' => [
            [
                'type' => 'label_selector',
                'label_selector' => [
                    'selector' => "location={$location->name}",
                ],
            ]
        ],
        'services' => [
            [
                'protocol' => 'http',
                'listen_port' => 80,
                'destination_port' => 80,
            ]
        ],
        'labels' => [
            'location' => $location->name,
            'city' => $location->city,
            'country' => $location->country,
        ]
    ]);
    
    echo "Created load balancer in {$location->name} ({$location->city})\n";
}
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $locations = HetznerLaravel::locations()->list();
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

- [Servers API](./servers) - Create servers in specific locations
- [Load Balancers API](./load-balancers) - Create load balancers in specific locations
- [Volumes API](./volumes) - Create volumes in specific locations
- [Networks API](./networks) - Configure networks for specific locations
