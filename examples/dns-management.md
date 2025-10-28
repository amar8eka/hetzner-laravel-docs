---
sidebar_position: 2
---

# DNS Management Examples

This guide provides practical examples for managing DNS zones and records using the Hetzner Laravel package.

## Basic DNS Operations

### Create DNS Zone and Records

```php
<?php

namespace App\Http\Controllers;

use Boci\HetznerLaravel\Facades\HetznerLaravel;
use Illuminate\Http\Request;

class DnsController extends Controller
{
    public function createDomain(Request $request)
    {
        $validated = $request->validate([
            'domain' => 'required|string|max:255',
            'ip_address' => 'required|ip',
        ]);

        try {
            // Create DNS zone
            $zone = HetznerLaravel::dnsZones()->create([
                'name' => $validated['domain'],
                'mode' => 'primary',
                'ttl' => 3600,
            ]);

            // Add A record for root domain
            $rootRecord = HetznerLaravel::dnsZones()->rrsets()->create($zone->id, [
                'type' => 'A',
                'name' => '@',
                'records' => [
                    [
                        'value' => $validated['ip_address']
                    ]
                ],
                'ttl' => 3600,
            ]);

            // Add A record for www subdomain
            $wwwRecord = HetznerLaravel::dnsZones()->rrsets()->create($zone->id, [
                'type' => 'A',
                'name' => 'www',
                'records' => [
                    [
                        'value' => $validated['ip_address']
                    ]
                ],
                'ttl' => 3600,
            ]);

            return response()->json([
                'success' => true,
                'zone' => $zone->toArray(),
                'records' => [
                    'root' => $rootRecord->toArray(),
                    'www' => $wwwRecord->toArray(),
                ],
                'message' => 'Domain and DNS records created successfully!'
            ]);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'error' => $e->getMessage()
            ], 500);
        }
    }
}
```

### Bulk DNS Record Management

```php
public function createSubdomains(Request $request)
{
    $validated = $request->validate([
        'zone_id' => 'required|string',
        'subdomains' => 'required|array',
        'ip_address' => 'required|ip',
    ]);

    try {
        $createdRecords = [];

        foreach ($validated['subdomains'] as $subdomain) {
            $record = HetznerLaravel::dnsZones()->rrsets()->create($validated['zone_id'], [
                'type' => 'A',
                'name' => $subdomain,
                'records' => [
                    [
                        'value' => $validated['ip_address']
                    ]
                ],
                'ttl' => 3600,
            ]);

            $createdRecords[] = $record->toArray();
        }

        return response()->json([
            'success' => true,
            'records' => $createdRecords,
            'count' => count($createdRecords),
            'message' => 'Subdomains created successfully!'
        ]);

    } catch (\Exception $e) {
        return response()->json([
            'success' => false,
            'error' => $e->getMessage()
        ], 500);
    }
}
```

## Advanced DNS Management

### Email Server Configuration

```php
public function setupEmailServer(Request $request)
{
    $validated = $request->validate([
        'domain' => 'required|string',
        'mail_server' => 'required|string',
        'mx_priority' => 'required|integer|min:1|max:65535',
    ]);

    try {
        $zone = HetznerLaravel::dnsZones()->retrieve($validated['domain']);

        // Add MX record
        $mxRecord = HetznerLaravel::dnsZones()->rrsets()->create($zone->id, [
            'type' => 'MX',
            'name' => '@',
            'records' => [
                [
                    'value' => "{$validated['mx_priority']} {$validated['mail_server']}"
                ]
            ],
            'ttl' => 3600,
        ]);

        // Add SPF record
        $spfRecord = HetznerLaravel::dnsZones()->rrsets()->create($zone->id, [
            'type' => 'TXT',
            'name' => '@',
            'records' => [
                [
                    'value' => 'v=spf1 include:_spf.google.com ~all'
                ]
            ],
            'ttl' => 3600,
        ]);

        // Add DMARC record
        $dmarcRecord = HetznerLaravel::dnsZones()->rrsets()->create($zone->id, [
            'type' => 'TXT',
            'name' => '_dmarc',
            'records' => [
                [
                    'value' => 'v=DMARC1; p=quarantine; rua=mailto:dmarc@' . $validated['domain']
                ]
            ],
            'ttl' => 3600,
        ]);

        return response()->json([
            'success' => true,
            'records' => [
                'mx' => $mxRecord->toArray(),
                'spf' => $spfRecord->toArray(),
                'dmarc' => $dmarcRecord->toArray(),
            ],
            'message' => 'Email server configuration completed!'
        ]);

    } catch (\Exception $e) {
        return response()->json([
            'success' => false,
            'error' => $e->getMessage()
        ], 500);
    }
}
```

### CDN and Load Balancer DNS Setup

```php
public function setupCdnDns(Request $request)
{
    $validated = $request->validate([
        'domain' => 'required|string',
        'cdn_endpoint' => 'required|string',
        'origin_server' => 'required|ip',
    ]);

    try {
        $zone = HetznerLaravel::dnsZones()->retrieve($validated['domain']);

        // Add CNAME for CDN
        $cdnRecord = HetznerLaravel::dnsZones()->rrsets()->create($zone->id, [
            'type' => 'CNAME',
            'name' => 'cdn',
            'records' => [
                [
                    'value' => $validated['cdn_endpoint']
                ]
            ],
            'ttl' => 3600,
        ]);

        // Add A record for origin server
        $originRecord = HetznerLaravel::dnsZones()->rrsets()->create($zone->id, [
            'type' => 'A',
            'name' => 'origin',
            'records' => [
                [
                    'value' => $validated['origin_server']
                ]
            ],
            'ttl' => 3600,
        ]);

        return response()->json([
            'success' => true,
            'records' => [
                'cdn' => $cdnRecord->toArray(),
                'origin' => $originRecord->toArray(),
            ],
            'message' => 'CDN DNS configuration completed!'
        ]);

    } catch (\Exception $e) {
        return response()->json([
            'success' => false,
            'error' => $e->getMessage()
        ], 500);
    }
}
```

## DNS Service Class

```php
<?php

namespace App\Services;

use Boci\HetznerLaravel\Facades\HetznerLaravel;

class DnsManagementService
{
    public function createZone(string $domain, int $ttl = 300): array
    {
        try {
            $zone = HetznerLaravel::dnsZones()->create([
                'name' => $domain,
                'ttl' => $ttl,
            ]);

            return [
                'success' => true,
                'zone' => $zone->toArray(),
                'message' => 'DNS zone created successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function addARecord(string $zoneId, string $name, string $ip, int $ttl = 300): array
    {
        try {
            $record = HetznerLaravel::dnsZones()->rrsets()->create($zoneId, [
                'type' => 'A',
                'name' => $name,
                'records' => [
                    [
                        'value' => $ip
                    ]
                ],
                'ttl' => $ttl,
            ]);

            return [
                'success' => true,
                'record' => $record->toArray(),
                'message' => 'A record created successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function addCnameRecord(string $zoneId, string $name, string $target, int $ttl = 300): array
    {
        try {
            $record = HetznerLaravel::dnsZones()->rrsets()->create($zoneId, [
                'type' => 'CNAME',
                'name' => $name,
                'records' => [
                    [
                        'value' => $target
                    ]
                ],
                'ttl' => $ttl,
            ]);

            return [
                'success' => true,
                'record' => $record->toArray(),
                'message' => 'CNAME record created successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function addMxRecord(string $zoneId, string $mailServer, int $priority = 10, int $ttl = 300): array
    {
        try {
            $record = HetznerLaravel::dnsZones()->rrsets()->create($zoneId, [
                'type' => 'MX',
                'name' => '@',
                'records' => [
                    [
                        'value' => "{$priority} {$mailServer}"
                    ]
                ],
                'ttl' => $ttl,
            ]);

            return [
                'success' => true,
                'record' => $record->toArray(),
                'message' => 'MX record created successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function addTxtRecord(string $zoneId, string $name, string $value, int $ttl = 300): array
    {
        try {
            $record = HetznerLaravel::dnsZones()->rrsets()->create($zoneId, [
                'type' => 'TXT',
                'name' => $name,
                'records' => [
                    [
                        'value' => $value
                    ]
                ],
                'ttl' => $ttl,
            ]);

            return [
                'success' => true,
                'record' => $record->toArray(),
                'message' => 'TXT record created successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function listRecords(string $zoneId): array
    {
        try {
            $records = HetznerLaravel::dnsZones()->rrsets()->list($zoneId);

            return [
                'success' => true,
                'records' => $records->toArray(),
                'count' => count($records)
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function deleteRecord(string $zoneId, string $recordId): array
    {
        try {
            HetznerLaravel::dnsZones()->rrsets()->delete($zoneId, $recordId);

            return [
                'success' => true,
                'message' => 'DNS record deleted successfully'
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

## Related Resources

- [API Reference - DNS Zones](../api-reference/dns-zones) - Complete DNS API documentation
- [Server Management Examples](./server-management) - Manage servers with DNS
- [Network Setup Examples](./network-setup) - Configure networks with DNS
- [Error Handling](../error-handling) - Handle DNS errors gracefully
