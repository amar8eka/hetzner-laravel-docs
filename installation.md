---
sidebar_position: 2
---

# Installation

This guide will walk you through installing and configuring the Hetzner Laravel package in your Laravel application.

## Requirements

Before installing Hetzner Laravel, make sure your environment meets these requirements:

- **PHP**: 8.1 or higher
- **Laravel**: 9.0 or higher
- **Composer**: Latest version
- **Hetzner Cloud API Token**: Required for authentication

## Installation Steps

### 1. Install via Composer

Install the package using Composer:

```bash
composer require boci/hetzner-laravel
```

### 2. Publish Configuration (Optional)

The package will work with default settings, but you can publish the configuration file to customize it:

```bash
php artisan vendor:publish --provider="Boci\HetznerLaravel\HetznerLaravelServiceProvider" --tag="config"
```

This will create a `hetzner-laravel.php` configuration file in your `config` directory.

### 3. Set Up Environment Variables

Add your Hetzner Cloud API token to your `.env` file:

```env
HETZNER_TOKEN=your-api-token-here
```

You can get your API token from the [Hetzner Cloud Console](https://console.hetzner.cloud/). For more information about API authentication, see the [official Hetzner Cloud documentation](https://docs.hetzner.cloud/).

### 4. Configure the Package

#### Using Environment Variables (Recommended)

The package will automatically use the `HETZNER_TOKEN` environment variable. No additional configuration is needed.

#### Using Configuration File

If you published the configuration file, you can customize the settings:

```php
// config/hetzner-laravel.php
<?php

/*
 * You can place your custom package configuration in here.
 */
return [
    'token' => env('HETZNER_TOKEN'),
];
```

## Usage Methods

The package provides multiple ways to interact with the Hetzner Cloud API:

### 1. Using the Facade (Recommended)

```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Create a server
$server = HetznerLaravel::servers()->create([
    'name' => 'my-server',
    'server_type' => 'cpx11',
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
]);
```

### 2. Using Dependency Injection

```php
use Boci\HetznerLaravel\Client;

class ServerController
{
    public function __construct(
        private Client $hetznerClient
    ) {}

    public function createServer()
    {
        $response = $this->hetznerClient->servers()->create([
            'name' => 'my-server',
            'server_type' => 'cpx11',
            'image' => 'ubuntu-24.04',
            'location' => 'nbg1',
        ]);

        return response()->json($response->toArray());
    }
}
```

### 3. Using the Service Container

```php
use Boci\HetznerLaravel\Client;

// Resolve from container
$client = app(Client::class);
```

## Verification

To verify that the installation is working correctly, you can test the connection:

```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

try {
    // Test the connection by listing servers
    $servers = HetznerLaravel::servers()->list();
    echo "Connection successful! Found " . count($servers) . " servers.";
} catch (Exception $e) {
    echo "Connection failed: " . $e->getMessage();
}
```

## Troubleshooting

### Common Issues

#### 1. API Token Not Found

**Error**: `API token not provided`

**Solution**: Make sure your `HETZNER_TOKEN` is set in your `.env` file and run:

```bash
php artisan config:clear
php artisan cache:clear
```

#### 2. Connection Issues

**Error**: `Connection failed` or `Network Error`

**Solution**: Check your internet connection and verify your API token is valid. You can test your token at the [Hetzner Cloud Console](https://console.hetzner.cloud/). For detailed troubleshooting, refer to the [official Hetzner Cloud documentation](https://docs.hetzner.cloud/).

## Next Steps

Now that you have Hetzner Laravel installed and configured, you're ready to:

1. **[Quick Start Guide](./quickstart)** - Create your first server
2. **[API Reference](./api-reference/servers)** - Explore all available methods
3. **[Examples](./examples/server-management)** - See practical usage patterns

## Support

If you encounter any issues during installation:

- Check the [troubleshooting guide](./troubleshooting)
- Review the [error handling documentation](./error-handling)
- Visit [docs.hetzner.cloud](https://docs.hetzner.cloud/) for official Hetzner Cloud documentation
- Open an issue on [GitHub](https://github.com/amar8eka/hetzner-laravel/issues)
- Contact support at amar8eka@gmail.com
