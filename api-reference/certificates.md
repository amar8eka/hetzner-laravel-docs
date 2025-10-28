---
sidebar_position: 10
---

# Certificates API Reference

The Certificates API allows you to manage SSL/TLS certificates for secure HTTPS connections.

## Available Methods

### `list(array $parameters = [])`

Get a list of all certificates.

**Parameters:**
- `name` (string, optional): Filter by certificate name
- `label_selector` (string, optional): Filter by label selector
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all certificates
$certificates = HetznerLaravel::certificates()->list();

// Get certificates with filters
$certificates = HetznerLaravel::certificates()->list([
    'name' => 'web-cert',
    'per_page' => 50,
]);
```

### `create(array $parameters)`

Create a new certificate.

**Required Parameters:**
- `name` (string): Certificate name
- `type` (string): Certificate type ('uploaded' or 'managed')

**Optional Parameters:**
- `certificate` (string): Certificate content (for uploaded type)
- `private_key` (string): Private key content (for uploaded type)
- `labels` (array): Key-value pairs for labeling

**Example:**
```php
$certificate = HetznerLaravel::certificates()->create([
    'name' => 'my-certificate',
    'type' => 'uploaded',
    'certificate' => '-----BEGIN CERTIFICATE-----...',
    'private_key' => '-----BEGIN PRIVATE KEY-----...',
    'labels' => [
        'environment' => 'production',
        'domain' => 'example.com'
    ],
]);
```

### `retrieve(string $certificateId)`

Get details of a specific certificate.

**Parameters:**
- `certificateId` (string): Certificate ID

**Example:**
```php
$certificate = HetznerLaravel::certificates()->retrieve('12345');
echo $certificate->name; // Certificate name
echo $certificate->type; // Certificate type
```

### `update(string $certificateId, array $parameters)`

Update certificate properties.

**Parameters:**
- `certificateId` (string): Certificate ID
- `name` (string, optional): New certificate name
- `labels` (array, optional): New labels

**Example:**
```php
$certificate = HetznerLaravel::certificates()->update('12345', [
    'name' => 'updated-certificate',
    'labels' => [
        'environment' => 'staging',
        'updated' => 'true'
    ]
]);
```

### `delete(string $certificateId)`

Delete a certificate.

**Parameters:**
- `certificateId` (string): Certificate ID

**Example:**
```php
HetznerLaravel::certificates()->delete('12345');
```

## Response Objects

### Certificate Object
- `id` (int): Certificate ID
- `name` (string): Certificate name
- `type` (string): Certificate type ('uploaded' or 'managed')
- `certificate` (string): Certificate content
- `created` (string): Creation timestamp
- `not_valid_before` (string): Valid from date
- `not_valid_after` (string): Valid until date
- `fingerprint` (string): Certificate fingerprint
- `labels` (object): Certificate labels

## Common Use Cases

### 1. Upload SSL Certificate

```php
$certificate = HetznerLaravel::certificates()->create([
    'name' => 'example-com-ssl',
    'type' => 'uploaded',
    'certificate' => file_get_contents('/path/to/certificate.crt'),
    'private_key' => file_get_contents('/path/to/private.key'),
    'labels' => [
        'domain' => 'example.com',
        'environment' => 'production'
    ]
]);
```

### 2. Use Certificate with Load Balancer

```php
// Create certificate
$certificate = HetznerLaravel::certificates()->create([
    'name' => 'web-ssl-cert',
    'type' => 'uploaded',
    'certificate' => $certContent,
    'private_key' => $keyContent,
]);

// Create load balancer with HTTPS service
$loadBalancer = HetznerLaravel::loadBalancers()->create([
    'name' => 'web-load-balancer',
    'load_balancer_type' => 'lb11',
    'location' => 'nbg1',
    'services' => [
        [
            'protocol' => 'https',
            'listen_port' => 443,
            'destination_port' => 80,
            'https' => [
                'certificates' => [$certificate->id],
            ],
        ]
    ],
]);
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $certificate = HetznerLaravel::certificates()->create($parameters);
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

- [Load Balancers API](./load-balancers) - Use certificates with load balancers
- [Servers API](./servers) - Configure certificates on servers
- [DNS Zones API](./dns-zones) - Manage DNS for certificate validation
