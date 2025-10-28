---
sidebar_position: 2
---

# DNS Zones API Reference

The DNS Zones API allows you to manage DNS zones and records for your domains. This includes creating zones, managing DNS records, and configuring DNS settings.

## Available Methods

### `list(array $parameters = [])`

Get a list of all DNS zones.

**Parameters:**
- `name` (string, optional): Filter by zone name
- `per_page` (int, optional): Number of items per page (1-100)
- `page` (int, optional): Page number

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all DNS zones
$zones = HetznerLaravel::dnsZones()->list();

// Get zones with filters
$zones = HetznerLaravel::dnsZones()->list([
    'name' => 'example.com',
    'per_page' => 50,
]);
```

### `create(array $parameters)`

Create a new DNS zone.

**Required Parameters:**
- `name` (string): Domain name for the zone

**Optional Parameters:**
- `ttl` (int): Default TTL for records (default: 300)

**Example:**
```php
$zone = HetznerLaravel::dnsZones()->create([
    'name' => 'example.com',
    'mode' => 'primary',
    'ttl' => 3600,
]);
```

### `retrieve(string $zoneIdOrName)`

Get details of a specific DNS zone.

**Parameters:**
- `zoneIdOrName` (string): Zone ID or domain name

**Example:**
```php
$zone = HetznerLaravel::dnsZones()->retrieve('example.com');
echo $zone->name; // Domain name
echo $zone->ttl; // Default TTL
```

### `update(string $zoneIdOrName, array $parameters)`

Update DNS zone properties.

**Parameters:**
- `zoneIdOrName` (string): Zone ID or domain name
- `ttl` (int, optional): New default TTL

**Example:**
```php
$zone = HetznerLaravel::dnsZones()->update('example.com', [
    'ttl' => 600,
]);
```

### `delete(string $zoneIdOrName)`

Delete a DNS zone.

**Parameters:**
- `zoneIdOrName` (string): Zone ID or domain name

**Example:**
```php
HetznerLaravel::dnsZones()->delete('example.com');
```

## DNS Records (RRSets)

### `rrsets()`

Access DNS records (RRSets) for managing individual DNS records.

### `rrsets()->list(string $zoneIdOrName, array $parameters = [])`

List all DNS records in a zone.

**Parameters:**
- `zoneIdOrName` (string): Zone ID or domain name
- `type` (string, optional): Filter by record type
- `name` (string, optional): Filter by record name
- `per_page` (int, optional): Number of items per page
- `page` (int, optional): Page number

**Example:**
```php
$records = HetznerLaravel::dnsZones()->rrsets()->list('example.com');

// Filter by type
$aRecords = HetznerLaravel::dnsZones()->rrsets()->list('example.com', [
    'type' => 'A'
]);
```

### `rrsets()->create(string $zoneIdOrName, array $parameters)`

Create a new DNS record.

**Required Parameters:**
- `type` (string): Record type (A, AAAA, CNAME, MX, NS, PTR, SOA, SRV, TXT)
- `name` (string): Record name
- `value` (string): Record value
- `ttl` (int): TTL for the record

**Example:**
```php
// Create an A record
$record = HetznerLaravel::dnsZones()->rrsets()->create('example.com', [
    'type' => 'A',
    'name' => 'www',
    'records' => [
        [
            'value' => '1.2.3.4'
        ]
    ],
    'ttl' => 3600,
]);

// Create an MX record
$mxRecord = HetznerLaravel::dnsZones()->rrsets()->create('example.com', [
    'type' => 'MX',
    'name' => '@',
    'records' => [
        [
            'value' => '10 mail.example.com'
        ]
    ],
    'ttl' => 3600,
]);

// Create a CNAME record
$cnameRecord = HetznerLaravel::dnsZones()->rrsets()->create('example.com', [
    'type' => 'CNAME',
    'name' => 'blog',
    'records' => [
        [
            'value' => 'www.example.com'
        ]
    ],
    'ttl' => 3600,
]);
```

### `rrsets()->retrieve(string $zoneIdOrName, string $recordId)`

Get details of a specific DNS record.

**Parameters:**
- `zoneIdOrName` (string): Zone ID or domain name
- `recordId` (string): Record ID

**Example:**
```php
$record = HetznerLaravel::dnsZones()->rrsets()->retrieve('example.com', 'record-id');
```

### `rrsets()->update(string $zoneIdOrName, string $recordId, array $parameters)`

Update a DNS record.

**Parameters:**
- `zoneIdOrName` (string): Zone ID or domain name
- `recordId` (string): Record ID
- `type` (string, optional): New record type
- `name` (string, optional): New record name
- `value` (string, optional): New record value
- `ttl` (int, optional): New TTL

**Example:**
```php
$record = HetznerLaravel::dnsZones()->rrsets()->update('example.com', 'record-id', [
    'records' => [
        [
            'value' => '5.6.7.8'
        ]
    ],
    'ttl' => 600,
]);
```

### `rrsets()->delete(string $zoneIdOrName, string $recordId)`

Delete a DNS record.

**Parameters:**
- `zoneIdOrName` (string): Zone ID or domain name
- `recordId` (string): Record ID

**Example:**
```php
HetznerLaravel::dnsZones()->rrsets()->delete('example.com', 'record-id');
```

## Zone Actions

### `actions()`

Access zone actions for performing operations like changing primary nameserver.

**Example:**
```php
// Change primary nameserver
$action = HetznerLaravel::dnsZones()->actions()->changePrimaryNameServer('example.com', [
    'primary_name_server' => 'ns1.example.com'
]);
```

## Response Objects

### DNS Zone Object
- `id` (string): Zone ID
- `name` (string): Domain name
- `ttl` (int): Default TTL
- `created` (string): Creation timestamp
- `modified` (string): Last modification timestamp
- `legacy_dns_host` (string): Legacy DNS host
- `legacy_ns` (array): Legacy nameservers
- `ns` (array): Current nameservers
- `owner` (string): Zone owner
- `paused` (bool): Zone pause status
- `permission` (string): Zone permissions
- `project` (string): Project ID
- `registrar` (string): Registrar type
- `status` (string): Zone status
- `verified` (string): Verification timestamp
- `records_count` (int): Number of records
- `is_secondary_dns` (bool): Secondary DNS status

### DNS Record Object
- `id` (string): Record ID
- `type` (string): Record type
- `name` (string): Record name
- `value` (string): Record value
- `ttl` (int): TTL
- `zone_id` (string): Zone ID
- `created` (string): Creation timestamp
- `modified` (string): Last modification timestamp

## Common Use Cases

### 1. Set Up a Complete Domain

```php
// Create the zone
$zone = HetznerLaravel::dnsZones()->create([
    'name' => 'example.com',
    'mode' => 'primary',
    'ttl' => 3600,
]);

// Add A record for root domain
HetznerLaravel::dnsZones()->rrsets()->create('example.com', [
    'name' => '@',
    'type' => 'A',
    'records' => [
        [
            'value' => '1.2.3.4'
        ]
    ],
    'ttl' => 3600,
]);

// Add A record for www subdomain
HetznerLaravel::dnsZones()->rrsets()->create('example.com', [
    'type' => 'A',
    'name' => 'www',
    'records' => [
        [
            'value' => '1.2.3.4'
        ]
    ],
    'ttl' => 3600,
]);

// Add MX record for email
HetznerLaravel::dnsZones()->rrsets()->create('example.com', [
        'name' => '@',
        'type' => 'MX',
        'ttl' => 3600,
        'records' => [
            [
                'value' => '10 mail.example.com'
            ]
        ]
]);
```

### 2. Manage Subdomains

```php
// Create subdomain records
$subdomains = ['api', 'admin', 'staging', 'dev'];

foreach ($subdomains as $subdomain) {
    HetznerLaravel::dnsZones()->rrsets()->create('example.com', [
        'type' => 'A',
        'name' => $subdomain,
                'records' => [
            [
                'value' => '1.2.3.4'
            ]
        ]
        'ttl' => 3600,
    ]);
}
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $zone = HetznerLaravel::dnsZones()->create($parameters);
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

- [Servers API](./servers) - Manage servers that can be referenced in DNS
- [Load Balancers API](./load-balancers) - Manage load balancers for DNS targets
- [Floating IPs API](./floating-ips) - Manage floating IPs for DNS records
