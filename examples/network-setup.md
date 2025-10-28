---
sidebar_position: 3
---

# Network Setup Examples

This guide provides practical examples for setting up private networks and configuring network security using the Hetzner Laravel package.

## Basic Network Operations

### Create Private Network

```php
<?php

namespace App\Http\Controllers;

use Boci\HetznerLaravel\Facades\HetznerLaravel;
use Illuminate\Http\Request;

class NetworkController extends Controller
{
    public function createPrivateNetwork(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'ip_range' => 'required|string',
            'location' => 'required|string',
        ]);

        try {
            $network = HetznerLaravel::networks()->create([
                'name' => $validated['name'],
                'ip_range' => $validated['ip_range'],
                'subnets' => [
                    [
                        'type' => 'cloud',
                        'ip_range' => $validated['ip_range'],
                        'network_zone' => $validated['location'],
                    ]
                ],
                'labels' => [
                    'environment' => 'production',
                    'purpose' => 'private-network'
                ]
            ]);

            return response()->json([
                'success' => true,
                'network' => $network->toArray(),
                'message' => 'Private network created successfully!'
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

### Multi-Zone Network Setup

```php
public function createMultiZoneNetwork(Request $request)
{
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'base_ip_range' => 'required|string',
        'zones' => 'required|array',
    ]);

    try {
        $subnets = [];
        $subnetIndex = 1;

        foreach ($validated['zones'] as $zone) {
            $subnets[] = [
                'type' => 'cloud',
                'ip_range' => "{$validated['base_ip_range']}.{$subnetIndex}.0/24",
                'network_zone' => $zone,
            ];
            $subnetIndex++;
        }

        $network = HetznerLaravel::networks()->create([
            'name' => $validated['name'],
            'ip_range' => $validated['base_ip_range'] . '.0/16',
            'subnets' => $subnets,
            'labels' => [
                'environment' => 'production',
                'multi-zone' => 'true'
            ]
        ]);

        return response()->json([
            'success' => true,
            'network' => $network->toArray(),
            'subnets' => $subnets,
            'message' => 'Multi-zone network created successfully!'
        ]);

    } catch (\Exception $e) {
        return response()->json([
            'success' => false,
            'error' => $e->getMessage()
        ], 500);
    }
}
```

## Advanced Network Configuration

### Tiered Network Architecture

```php
public function createTieredNetwork(Request $request)
{
    $validated = $request->validate([
        'base_name' => 'required|string|max:255',
        'location' => 'required|string',
    ]);

    try {
        $networks = [];
        $tiers = [
            'web' => ['ip_range' => '10.1.0.0/16', 'description' => 'Web tier network'],
            'app' => ['ip_range' => '10.2.0.0/16', 'description' => 'Application tier network'],
            'db' => ['ip_range' => '10.3.0.0/16', 'description' => 'Database tier network'],
        ];

        foreach ($tiers as $tier => $config) {
            $network = HetznerLaravel::networks()->create([
                'name' => "{$validated['base_name']}-{$tier}-tier",
                'ip_range' => $config['ip_range'],
                'subnets' => [
                    [
                        'type' => 'cloud',
                        'ip_range' => str_replace('/16', '/24', $config['ip_range']),
                        'network_zone' => $validated['location'],
                    ]
                ],
                'labels' => [
                    'tier' => $tier,
                    'environment' => 'production',
                    'description' => $config['description']
                ]
            ]);

            $networks[$tier] = $network->toArray();
        }

        return response()->json([
            'success' => true,
            'networks' => $networks,
            'message' => 'Tiered network architecture created successfully!'
        ]);

    } catch (\Exception $e) {
        return response()->json([
            'success' => false,
            'error' => $e->getMessage()
        ], 500);
    }
}
```

### Network with Firewall Rules

```php
public function createSecureNetwork(Request $request)
{
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'ip_range' => 'required|string',
        'location' => 'required|string',
    ]);

    try {
        // Create network
        $network = HetznerLaravel::networks()->create([
            'name' => $validated['name'],
            'ip_range' => $validated['ip_range'],
            'subnets' => [
                [
                    'type' => 'cloud',
                    'ip_range' => $validated['ip_range'],
                    'network_zone' => $validated['location'],
                ]
            ],
        ]);

        // Create firewall for the network
        $firewall = HetznerLaravel::firewalls()->create([
            'name' => "{$validated['name']}-firewall",
            'rules' => [
                [
                    'direction' => 'in',
                    'source_ips' => [$validated['ip_range']],
                    'destination_ips' => [],
                    'protocol' => 'tcp',
                    'port' => '22',
                    'description' => 'Allow SSH within network',
                ],
                [
                    'direction' => 'in',
                    'source_ips' => [$validated['ip_range']],
                    'destination_ips' => [],
                    'protocol' => 'tcp',
                    'port' => '80',
                    'description' => 'Allow HTTP within network',
                ],
                [
                    'direction' => 'in',
                    'source_ips' => [$validated['ip_range']],
                    'destination_ips' => [],
                    'protocol' => 'tcp',
                    'port' => '443',
                    'description' => 'Allow HTTPS within network',
                ],
            ],
            'labels' => [
                'network_id' => $network->id,
                'environment' => 'production'
            ]
        ]);

        return response()->json([
            'success' => true,
            'network' => $network->toArray(),
            'firewall' => $firewall->toArray(),
            'message' => 'Secure network created successfully!'
        ]);

    } catch (\Exception $e) {
        return response()->json([
            'success' => false,
            'error' => $e->getMessage()
        ], 500);
    }
}
```

## Network Service Class

```php
<?php

namespace App\Services;

use Boci\HetznerLaravel\Facades\HetznerLaravel;

class NetworkManagementService
{
    public function createNetwork(string $name, string $ipRange, string $location): array
    {
        try {
            $network = HetznerLaravel::networks()->create([
                'name' => $name,
                'ip_range' => $ipRange,
                'subnets' => [
                    [
                        'type' => 'cloud',
                        'ip_range' => $ipRange,
                        'network_zone' => $location,
                    ]
                ],
            ]);

            return [
                'success' => true,
                'network' => $network->toArray(),
                'message' => 'Network created successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function createServerInNetwork(string $serverName, string $serverType, string $image, string $location, string $networkId): array
    {
        try {
            $server = HetznerLaravel::servers()->create([
                'name' => $serverName,
                'server_type' => $serverType,
                'image' => $image,
                'location' => $location,
                'networks' => [$networkId],
                'ssh_keys' => ['my-ssh-key'],
            ]);

            return [
                'success' => true,
                'server' => $server->toArray(),
                'message' => 'Server created in network successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function attachLoadBalancerToNetwork(string $lbName, string $lbType, string $location, string $networkId): array
    {
        try {
            $loadBalancer = HetznerLaravel::loadBalancers()->create([
                'name' => $lbName,
                'load_balancer_type' => $lbType,
                'location' => $location,
                'network' => $networkId,
                'targets' => [
                    [
                        'type' => 'label_selector',
                        'label_selector' => [
                            'selector' => 'network=' . $networkId,
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
            ]);

            return [
                'success' => true,
                'load_balancer' => $loadBalancer->toArray(),
                'message' => 'Load balancer attached to network successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function listNetworks(): array
    {
        try {
            $networks = HetznerLaravel::networks()->list();

            return [
                'success' => true,
                'networks' => $networks->toArray(),
                'count' => count($networks)
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function getNetwork(string $networkId): array
    {
        try {
            $network = HetznerLaravel::networks()->retrieve($networkId);

            return [
                'success' => true,
                'network' => $network->toArray()
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function updateNetwork(string $networkId, array $data): array
    {
        try {
            $network = HetznerLaravel::networks()->update($networkId, $data);

            return [
                'success' => true,
                'network' => $network->toArray(),
                'message' => 'Network updated successfully'
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }

    public function deleteNetwork(string $networkId): array
    {
        try {
            HetznerLaravel::networks()->delete($networkId);

            return [
                'success' => true,
                'message' => 'Network deleted successfully'
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

- [API Reference - Networks](../api-reference/networks) - Complete network API documentation
- [API Reference - Firewalls](../api-reference/firewalls) - Secure network traffic
- [Server Management Examples](./server-management) - Create servers in networks
- [Load Balancer Configuration Examples](./load-balancer-configuration) - Configure load balancers in networks
