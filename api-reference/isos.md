---
sidebar_position: 16
---

# ISOs API Reference

The ISOs API allows you to retrieve information about available ISO images that can be attached to servers.

## Available Methods

### `list(array $parameters = [])`

Get a list of all available ISOs.

**Parameters:**
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all ISOs
$isos = HetznerLaravel::isos()->list();

// Get ISOs with pagination
$isos = HetznerLaravel::isos()->list([
    'per_page' => 50,
]);
```

### `retrieve(string $isoId)`

Get details of a specific ISO.

**Parameters:**
- `isoId` (string): ISO ID or name

**Example:**
```php
$iso = HetznerLaravel::isos()->retrieve('ubuntu-20.04-server-amd64');
echo $iso->name; // ISO name
echo $iso->description; // ISO description
echo $iso->type; // ISO type
```

## Response Objects

### ISO Object
- `id` (int): ISO ID
- `name` (string): ISO name
- `description` (string): ISO description
- `type` (string): ISO type ('public' or 'private')
- `architecture` (string): Architecture ('x86' or 'arm')
- `deprecated` (string): Deprecation timestamp (if deprecated)

## Common Use Cases

### 1. List All Available ISOs

```php
$isos = HetznerLaravel::isos()->list();

echo "Available ISOs:\n";
echo "================\n";

foreach ($isos as $iso) {
    echo sprintf("%-30s: %s (%s)\n", 
        $iso->name, 
        $iso->description, 
        $iso->architecture);
}
```

### 2. Get ISO Information

```php
$iso = HetznerLaravel::isos()->retrieve('ubuntu-20.04-server-amd64');

echo "ISO Details:\n";
echo "============\n";
echo "Name: {$iso->name}\n";
echo "Description: {$iso->description}\n";
echo "Type: {$iso->type}\n";
echo "Architecture: {$iso->architecture}\n";
```

### 3. Find ISOs by Architecture

```php
$isos = HetznerLaravel::isos()->list();
$architecture = 'x86';

$x86Isos = array_filter($isos, function($iso) use ($architecture) {
    return $iso->architecture === $architecture;
});

echo "x86 ISOs:\n";
echo "==========\n";

foreach ($x86Isos as $iso) {
    echo "{$iso->name}: {$iso->description}\n";
}
```

### 4. Find ISOs by Type

```php
$isos = HetznerLaravel::isos()->list();

$publicIsos = array_filter($isos, function($iso) {
    return $iso->type === 'public';
});

$privateIsos = array_filter($isos, function($iso) {
    return $iso->type === 'private';
});

echo "Public ISOs: " . count($publicIsos) . "\n";
echo "Private ISOs: " . count($privateIsos) . "\n";
```

### 5. Use ISO with Server Creation

```php
// Get available ISOs
$isos = HetznerLaravel::isos()->list();

// Find Ubuntu ISO
$ubuntuIso = null;
foreach ($isos as $iso) {
    if (strpos($iso->name, 'ubuntu') !== false) {
        $ubuntuIso = $iso;
        break;
    }
}

if ($ubuntuIso) {
    // Create server with ISO attached
    $server = HetznerLaravel::servers()->create([
        'name' => 'iso-server',
        'server_type' => 'cpx11',
        'image' => 'ubuntu-24.04',
        'location' => 'nbg1',
        'iso' => $ubuntuIso->id,
        'ssh_keys' => ['my-ssh-key'],
        'labels' => [
            'iso' => $ubuntuIso->name,
            'purpose' => 'testing'
        ]
    ]);
    
    echo "Created server with ISO: {$ubuntuIso->name}\n";
}
```

### 6. ISO Management Service

```php
<?php

namespace App\Services;

use Boci\HetznerLaravel\Facades\HetznerLaravel;

class IsoManagementService
{
    public function listIsos(array $filters = []): array
    {
        try {
            $isos = HetznerLaravel::isos()->list($filters);
            
            return [
                'success' => true,
                'isos' => $isos->toArray(),
                'count' => count($isos)
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function getIso(string $isoId): array
    {
        try {
            $iso = HetznerLaravel::isos()->retrieve($isoId);
            
            return [
                'success' => true,
                'iso' => $iso->toArray()
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function findIsoByName(string $name): ?object
    {
        try {
            $isos = HetznerLaravel::isos()->list();
            
            foreach ($isos as $iso) {
                if (strpos(strtolower($iso->name), strtolower($name)) !== false) {
                    return $iso;
                }
            }
            
            return null;
        } catch (\Exception $e) {
            return null;
        }
    }
    
    public function getIsosByArchitecture(string $architecture): array
    {
        try {
            $isos = HetznerLaravel::isos()->list();
            
            $filteredIsos = array_filter($isos, function($iso) use ($architecture) {
                return $iso->architecture === $architecture;
            });
            
            return [
                'success' => true,
                'isos' => array_values($filteredIsos),
                'count' => count($filteredIsos)
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
}
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $isos = HetznerLaravel::isos()->list();
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

- [Servers API](./servers) - Attach ISOs to servers
- [Images API](./images) - Create images from ISOs
- [Locations API](./locations) - Create servers with ISOs in specific locations
- [Actions API](./actions) - Monitor ISO attachment actions
