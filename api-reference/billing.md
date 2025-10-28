---
sidebar_position: 12
---

# Billing API Reference

The Billing API allows you to retrieve pricing information for Hetzner Cloud services.

## Available Methods

### `listPricing()`

Get all pricing information for Hetzner Cloud services.

**Example:**
```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Get all pricing information
$pricing = HetznerLaravel::billing()->listPricing();

// Access specific pricing information
echo $pricing->currency; // Currency (EUR)
echo $pricing->vat_rate; // VAT rate

// Server type pricing
foreach ($pricing->server_types as $serverType) {
    echo "{$serverType->name}: {$serverType->prices->hourly->gross} EUR/hour\n";
}

// Volume pricing
echo "Volume pricing: {$pricing->volume->price_per_gb_month->gross} EUR/GB/month\n";

// Load balancer pricing
foreach ($pricing->load_balancer_types as $lbType) {
    echo "{$lbType->name}: {$lbType->prices->hourly->gross} EUR/hour\n";
}
```

## Response Objects

### Pricing Object
- `currency` (string): Currency (EUR)
- `vat_rate` (string): VAT rate
- `server_types` (array): Server type pricing
- `volume` (object): Volume pricing
- `load_balancer_types` (array): Load balancer type pricing
- `floating_ips` (array): Floating IP pricing
- `primary_ips` (array): Primary IP pricing
- `traffic` (object): Traffic pricing
- `certificates` (object): Certificate pricing

## Common Use Cases

### 1. Calculate Server Costs

```php
$pricing = HetznerLaravel::billing()->listPricing();

// Calculate monthly cost for a server
$serverType = 'cpx21';
$monthlyHours = 24 * 30; // 24 hours * 30 days

foreach ($pricing->server_types as $type) {
    if ($type->name === $serverType) {
        $hourlyCost = $type->prices->hourly->gross;
        $monthlyCost = $hourlyCost * $monthlyHours;
        
        echo "Server type: {$serverType}\n";
        echo "Hourly cost: {$hourlyCost} EUR\n";
        echo "Monthly cost: {$monthlyCost} EUR\n";
        break;
    }
}
```

### 2. Compare Server Type Costs

```php
$pricing = HetznerLaravel::billing()->listPricing();

echo "Server Type Pricing Comparison:\n";
echo "===============================\n";

foreach ($pricing->server_types as $type) {
    $hourlyCost = $type->prices->hourly->gross;
    $monthlyCost = $hourlyCost * 24 * 30;
    
    echo sprintf("%-10s: %8.4f EUR/hour (%8.2f EUR/month)\n", 
        $type->name, $hourlyCost, $monthlyCost);
}
```

### 3. Calculate Storage Costs

```php
$pricing = HetznerLaravel::billing()->listPricing();

// Calculate volume costs
$volumeSize = 100; // GB
$volumeCostPerGB = $pricing->volume->price_per_gb_month->gross;
$monthlyVolumeCost = $volumeSize * $volumeCostPerGB;

echo "Volume Storage Costs:\n";
echo "====================\n";
echo "Volume size: {$volumeSize} GB\n";
echo "Cost per GB/month: {$volumeCostPerGB} EUR\n";
echo "Monthly cost: {$monthlyVolumeCost} EUR\n";
```

### 4. Calculate Load Balancer Costs

```php
$pricing = HetznerLaravel::billing()->listPricing();

echo "Load Balancer Pricing:\n";
echo "=====================\n";

foreach ($pricing->load_balancer_types as $type) {
    $hourlyCost = $type->prices->hourly->gross;
    $monthlyCost = $hourlyCost * 24 * 30;
    
    echo sprintf("%-10s: %8.4f EUR/hour (%8.2f EUR/month)\n", 
        $type->name, $hourlyCost, $monthlyCost);
}
```

### 5. Calculate Total Infrastructure Costs

```php
$pricing = HetznerLaravel::billing()->listPricing();

// Infrastructure configuration
$servers = [
    ['type' => 'cpx21', 'count' => 2],
    ['type' => 'cpx31', 'count' => 1],
];
$volumeSize = 50; // GB
$loadBalancerType = 'lb11';

$totalMonthlyCost = 0;

// Calculate server costs
foreach ($servers as $server) {
    foreach ($pricing->server_types as $type) {
        if ($type->name === $server['type']) {
            $hourlyCost = $type->prices->hourly->gross;
            $monthlyCost = $hourlyCost * 24 * 30 * $server['count'];
            $totalMonthlyCost += $monthlyCost;
            
            echo "{$server['count']}x {$server['type']}: {$monthlyCost} EUR/month\n";
            break;
        }
    }
}

// Calculate volume costs
$volumeCostPerGB = $pricing->volume->price_per_gb_month->gross;
$volumeCost = $volumeSize * $volumeCostPerGB;
$totalMonthlyCost += $volumeCost;
echo "Volume ({$volumeSize}GB): {$volumeCost} EUR/month\n";

// Calculate load balancer costs
foreach ($pricing->load_balancer_types as $type) {
    if ($type->name === $loadBalancerType) {
        $hourlyCost = $type->prices->hourly->gross;
        $monthlyCost = $hourlyCost * 24 * 30;
        $totalMonthlyCost += $monthlyCost;
        
        echo "Load Balancer ({$loadBalancerType}): {$monthlyCost} EUR/month\n";
        break;
    }
}

echo "===============================\n";
echo "Total Monthly Cost: {$totalMonthlyCost} EUR\n";
```

## Error Handling

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $pricing = HetznerLaravel::billing()->listPricing();
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

- [Servers API](./servers) - Get server pricing information
- [Volumes API](./volumes) - Get volume pricing information
- [Load Balancers API](./load-balancers) - Get load balancer pricing information
- [Floating IPs API](./floating-ips) - Get floating IP pricing information
