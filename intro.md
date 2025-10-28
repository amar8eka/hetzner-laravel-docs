---
sidebar_position: 1
---

# Introduction

Welcome to **Hetzner Laravel** ⚡ — an elegant, type-safe Laravel SDK that provides seamless integration with the Hetzner Cloud API. This package allows you to manage your Hetzner Cloud resources programmatically from your Laravel applications.

## What is Hetzner Laravel?

Hetzner Laravel is a comprehensive PHP SDK that wraps the Hetzner Cloud API, providing you with:

- **Type-safe operations** for all Hetzner Cloud resources
- **Elegant Laravel-style API** that feels natural to Laravel developers
- **Complete coverage** of Hetzner Cloud services
- **Robust error handling** with custom exceptions
- **Dependency injection** support for clean architecture
- **Flexible HTTP client** configuration

## Key Features

### 🚀 **Complete API Coverage**
Manage all Hetzner Cloud resources:
- **Actions**: Get multiple actions, get an action
- **Billing**: Get all prices
- **Certificates**: List, create, get, update, delete certificates
- **DNS Zones**: List, create, get, update, delete DNS zones + actions + RRSets
- **Firewalls**: List, create, get, update, delete firewalls + actions
- **Floating IPs**: List, create, get, update, delete floating IPs + actions
- **Images**: List, get, update, delete images + actions
- **ISOs**: List, get ISOs
- **Load Balancers**: List, create, get, update, delete load balancers + actions
- **Load Balancer Types**: List, get load balancer types
- **Locations**: List, get locations
- **Networks**: List, create, get, update, delete networks + actions
- **Placement Groups**: List, create, get, update, delete placement groups
- **Primary IPs**: List, create, get, update, delete primary IPs + actions
- **Servers**: List, create, get, update, delete servers + actions
- **Server Types**: List, get server types
- **SSH Keys**: List, create, get, update, delete SSH keys
- **Volumes**: List, create, get, update, delete volumes + actions

### 🛡️ **Type Safety**
Built with PHP 8+ features and proper type hints for better IDE support and fewer runtime errors.

### 🔧 **Laravel Integration**
- Service provider for automatic registration
- Facade support for easy access
- Dependency injection ready
- Configuration management

### ⚡ **Performance**
- Efficient HTTP client with connection pooling
- Request/response caching support
- Minimal overhead
- Async operations support

## Quick Example

Here's a simple example of what you can do with Hetzner Laravel:

```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

// Create a new server
$server = HetznerLaravel::servers()->create([
    'name' => 'my-web-server',
    'server_type' => 'cpx11',
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
    'ssh_keys' => ['my-ssh-key'],
]);

// List all servers
$servers = HetznerLaravel::servers()->list();

// Create a DNS zone
$dnsZone = HetznerLaravel::dnsZones()->create([
    'name' => 'example.com',
    'mode' => 'primary',
    'ttl' => 3600,
]);
```

## Why Choose Hetzner Laravel?

### ✅ **Developer Experience**
- Intuitive API that follows Laravel conventions
- Comprehensive documentation with examples
- IDE autocompletion and type hints
- Clear error messages and debugging support

### ✅ **Production Ready**
- Thoroughly tested with PHPUnit
- Follows PSR standards
- Security-focused design
- Regular updates and maintenance

### ✅ **Community Driven**
- Open source with MIT license
- Active community support
- Contributing guidelines
- Issue tracking and feature requests

## What's Next?

Ready to get started? Here's what you can do next:

1. **[Installation Guide](./installation)** - Set up the package in your Laravel project
2. **[Quick Start](./quickstart)** - Create your first server in minutes
3. **[API Reference](./api-reference/servers)** - Explore all available methods
4. **[Examples](./examples/server-management)** - See real-world usage patterns

## Support

- 📚 **Documentation**: This site provides comprehensive guides and API reference
- 🌐 **Official Hetzner Cloud Docs**: For detailed API specifications, visit [docs.hetzner.cloud](https://docs.hetzner.cloud/)
- 🐛 **Issues**: Report bugs or request features on [GitHub](https://github.com/amar8eka/hetzner-laravel/issues)
- 💬 **Discussions**: Join the community discussions
- 📧 **Security**: Report security issues to amar8eka@gmail.com

## License

This package is open-sourced software licensed under the [MIT license](https://github.com/amar8eka/hetzner-laravel/blob/main/LICENSE.md).

---

**Ready to supercharge your Hetzner Cloud management?** Let's get started with the [installation guide](./installation)!