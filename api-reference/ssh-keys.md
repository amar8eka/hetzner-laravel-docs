---
sidebar_position: 9
---

# SSH Keys API Reference

The SSH Keys API allows you to manage SSH keys for secure server access.

## Available Methods

### `list(array $parameters = [])`

Get a list of all SSH keys.

**Parameters:**
- `name` (string, optional): Filter by SSH key name
- `label_selector` (string, optional): Filter by label selector
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all SSH keys
$sshKeys = HetznerLaravel::sshKeys()->list();

// Get SSH keys with filters
$sshKeys = HetznerLaravel::sshKeys()->list([
    'name' => 'my-ssh-key',
    'per_page' => 50,
]);
```

### `create(array $parameters)`

Create a new SSH key.

**Required Parameters:**
- `name` (string): SSH key name
- `public_key` (string): Public key content

**Optional Parameters:**
- `labels` (array): Key-value pairs for labeling

**Example:**
```php
$sshKey = HetznerLaravel::sshKeys()->create([
    'name' => 'my-ssh-key',
    'public_key' => 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC... user@example.com',
    'labels' => [
        'environment' => 'production',
        'user' => 'admin'
    ],
]);
```

### `retrieve(string $sshKeyId)`

Get details of a specific SSH key.

**Parameters:**
- `sshKeyId` (string): SSH key ID

**Example:**
```php
$sshKey = HetznerLaravel::sshKeys()->retrieve('12345');
echo $sshKey->name; // SSH key name
echo $sshKey->fingerprint; // SSH key fingerprint
```

### `update(string $sshKeyId, array $parameters)`

Update SSH key properties.

**Parameters:**
- `sshKeyId` (string): SSH key ID
- `name` (string, optional): New SSH key name
- `labels` (array, optional): New labels

**Example:**
```php
$sshKey = HetznerLaravel::sshKeys()->update('12345', [
    'name' => 'updated-ssh-key',
    'labels' => [
        'environment' => 'staging',
        'updated' => 'true'
    ]
]);
```

### `delete(string $sshKeyId)`

Delete an SSH key.

**Parameters:**
- `sshKeyId` (string): SSH key ID

**Example:**
```php
HetznerLaravel::sshKeys()->delete('12345');
```

## Response Objects

### SSH Key Object
- `id` (int): SSH key ID
- `name` (string): SSH key name
- `fingerprint` (string): SSH key fingerprint
- `public_key` (string): Public key content
- `created` (string): Creation timestamp
- `labels` (object): SSH key labels

## Common Use Cases

### 1. Create SSH Key from File

```php
// Read SSH key from file
$publicKey = file_get_contents('/home/user/.ssh/id_rsa.pub');

$sshKey = HetznerLaravel::sshKeys()->create([
    'name' => 'user-ssh-key',
    'public_key' => $publicKey,
    'labels' => [
        'user' => 'admin',
        'environment' => 'production'
    ]
]);
```

### 2. Create Multiple SSH Keys

```php
$sshKeys = [
    'admin' => HetznerLaravel::sshKeys()->create([
        'name' => 'admin-ssh-key',
        'public_key' => 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC... admin@example.com',
        'labels' => [
            'role' => 'admin',
            'environment' => 'production'
        ]
    ]),
    'developer' => HetznerLaravel::sshKeys()->create([
        'name' => 'developer-ssh-key',
        'public_key' => 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC... developer@example.com',
        'labels' => [
            'role' => 'developer',
            'environment' => 'staging'
        ]
    ]),
    'deploy' => HetznerLaravel::sshKeys()->create([
        'name' => 'deploy-ssh-key',
        'public_key' => 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC... deploy@example.com',
        'labels' => [
            'role' => 'deploy',
            'environment' => 'production'
        ]
    ]),
];
```

### 3. Use SSH Key in Server Creation

```php
// Get SSH key
$sshKey = HetznerLaravel::sshKeys()->retrieve('12345');

// Create server with SSH key
$server = HetznerLaravel::servers()->create([
    'name' => 'my-server',
    'server_type' => 'cpx11',
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
    'ssh_keys' => [$sshKey->id],
    'labels' => [
        'environment' => 'production'
    ]
]);
```

### 4. Manage SSH Keys by Environment

```php
// Create SSH keys for different environments
$environments = ['production', 'staging', 'development'];

foreach ($environments as $env) {
    $sshKey = HetznerLaravel::sshKeys()->create([
        'name' => "{$env}-ssh-key",
        'public_key' => "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC... {$env}@example.com",
        'labels' => [
            'environment' => $env,
            'created_date' => date('Y-m-d')
        ]
    ]);
    
    echo "Created SSH key for {$env}: {$sshKey->name}\n";
}
```

### 5. SSH Key Rotation

```php
// Create new SSH key
$newSshKey = HetznerLaravel::sshKeys()->create([
    'name' => 'rotated-ssh-key-' . date('Y-m-d'),
    'public_key' => 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC... user@example.com',
    'labels' => [
        'rotation_date' => date('Y-m-d'),
        'status' => 'active'
    ]
]);

// Update old SSH key status
$oldSshKey = HetznerLaravel::sshKeys()->retrieve('12345');
HetznerLaravel::sshKeys()->update($oldSshKey->id, [
    'labels' => array_merge($oldSshKey->labels, [
        'status' => 'deprecated',
        'deprecated_date' => date('Y-m-d')
    ])
]);

// Delete deprecated SSH keys older than 30 days
$deprecatedKeys = HetznerLaravel::sshKeys()->list([
    'label_selector' => 'status=deprecated',
]);

$cutoffDate = date('Y-m-d', strtotime('-30 days'));

foreach ($deprecatedKeys as $key) {
    $deprecatedDate = $key->labels['deprecated_date'] ?? '';
    
    if ($deprecatedDate && $deprecatedDate < $cutoffDate) {
        HetznerLaravel::sshKeys()->delete($key->id);
        echo "Deleted deprecated SSH key: {$key->name}\n";
    }
}
```

### 6. SSH Key Validation

```php
function validateSshKey(string $publicKey): bool
{
    // Basic validation for SSH public key format
    $patterns = [
        '/^ssh-rsa\s+[A-Za-z0-9+\/]+=*\s+.*$/',
        '/^ssh-ed25519\s+[A-Za-z0-9+\/]+=*\s+.*$/',
        '/^ecdsa-sha2-nistp256\s+[A-Za-z0-9+\/]+=*\s+.*$/',
        '/^ecdsa-sha2-nistp384\s+[A-Za-z0-9+\/]+=*\s+.*$/',
        '/^ecdsa-sha2-nistp521\s+[A-Za-z0-9+\/]+=*\s+.*$/',
    ];
    
    foreach ($patterns as $pattern) {
        if (preg_match($pattern, $publicKey)) {
            return true;
        }
    }
    
    return false;
}

// Create SSH key with validation
$publicKey = 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC... user@example.com';

if (validateSshKey($publicKey)) {
    $sshKey = HetznerLaravel::sshKeys()->create([
        'name' => 'validated-ssh-key',
        'public_key' => $publicKey,
        'labels' => [
            'validated' => 'true',
            'created_date' => date('Y-m-d')
        ]
    ]);
} else {
    throw new InvalidArgumentException('Invalid SSH public key format');
}
```

### 7. SSH Key Management Service

```php
<?php

namespace App\Services;

use Boci\HetznerLaravel\Facades\HetznerLaravel;

class SshKeyManagementService
{
    public function createSshKey(string $name, string $publicKey, array $labels = []): array
    {
        try {
            $sshKey = HetznerLaravel::sshKeys()->create([
                'name' => $name,
                'public_key' => $publicKey,
                'labels' => $labels,
            ]);
            
            return [
                'success' => true,
                'ssh_key' => $sshKey->toArray(),
                'message' => 'SSH key created successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function getSshKey(string $sshKeyId): array
    {
        try {
            $sshKey = HetznerLaravel::sshKeys()->retrieve($sshKeyId);
            
            return [
                'success' => true,
                'ssh_key' => $sshKey->toArray()
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function listSshKeys(array $filters = []): array
    {
        try {
            $sshKeys = HetznerLaravel::sshKeys()->list($filters);
            
            return [
                'success' => true,
                'ssh_keys' => $sshKeys->toArray(),
                'count' => count($sshKeys)
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function updateSshKey(string $sshKeyId, array $data): array
    {
        try {
            $sshKey = HetznerLaravel::sshKeys()->update($sshKeyId, $data);
            
            return [
                'success' => true,
                'ssh_key' => $sshKey->toArray(),
                'message' => 'SSH key updated successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
    
    public function deleteSshKey(string $sshKeyId): array
    {
        try {
            HetznerLaravel::sshKeys()->delete($sshKeyId);
            
            return [
                'success' => true,
                'message' => 'SSH key deleted successfully'
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
    $sshKey = HetznerLaravel::sshKeys()->create($parameters);
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

- [Servers API](./servers) - Use SSH keys in server creation
- [Images API](./images) - Create images with SSH access
- [Firewalls API](./firewalls) - Secure SSH access
- [Networks API](./networks) - Configure private networks for SSH
