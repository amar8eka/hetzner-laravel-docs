---
sidebar_position: 1
---

# Server Management Examples

This guide provides practical examples for managing Hetzner Cloud servers using the Hetzner Laravel package.

## Basic Server Operations

### Create a Web Server

```php
<?php

namespace App\Http\Controllers;

use Boci\HetznerLaravel\Facades\HetznerLaravel;
use Illuminate\Http\Request;

class ServerController extends Controller
{
    public function createWebServer(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'server_type' => 'required|string',
            'location' => 'required|string',
        ]);

        try {
            $server = HetznerLaravel::servers()->create([
                'name' => $validated['name'],
                'server_type' => $validated['server_type'],
                'image' => 'ubuntu-24.04',
                'location' => $validated['location'],
                'ssh_keys' => ['my-ssh-key'],
                'user_data' => $this->getWebServerUserData(),
                'labels' => [
                    'role' => 'web-server',
                    'environment' => 'production',
                    'created_by' => 'laravel-app'
                ]
            ]);

            return response()->json([
                'success' => true,
                'server' => $server->toArray(),
                'message' => 'Web server created successfully!'
            ]);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'error' => $e->getMessage()
            ], 500);
        }
    }

    private function getWebServerUserData(): string
    {
        return '#cloud-config
packages:
  - nginx
  - php8.1-fpm
  - php8.1-mysql
  - php8.1-curl
  - php8.1-gd
  - php8.1-mbstring
  - php8.1-xml
  - php8.1-zip
  - mysql-client
  - curl
  - git

runcmd:
  - systemctl enable nginx
  - systemctl start nginx
  - systemctl enable php8.1-fpm
  - systemctl start php8.1-fpm
  - echo "server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php index.html;
    
    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }
    
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        include fastcgi_params;
    }
}" > /etc/nginx/sites-available/default
  - systemctl reload nginx

write_files:
  - path: /var/www/html/index.php
    content: |
      <?php
      phpinfo();
    mode: "0644"
    owner: "www-data:www-data"';
    }
}
```

### Create a Database Server

```php
public function createDatabaseServer(Request $request)
{
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'server_type' => 'required|string',
        'location' => 'required|string',
        'volume_size' => 'required|integer|min:10',
    ]);

    try {
        // First create a volume for data storage
        $volume = HetznerLaravel::volumes()->create([
            'name' => $validated['name'] . '-data-volume',
            'size' => $validated['volume_size'],
            'location' => $validated['location'],
            'labels' => [
                'purpose' => 'database-storage',
                'server' => $validated['name']
            ]
        ]);

        // Then create the server with the volume
        $server = HetznerLaravel::servers()->create([
            'name' => $validated['name'],
            'server_type' => $validated['server_type'],
            'image' => 'ubuntu-24.04',
            'location' => $validated['location'],
            'ssh_keys' => ['my-ssh-key'],
            'volumes' => [$volume->id],
            'user_data' => $this->getDatabaseServerUserData(),
            'labels' => [
                'role' => 'database-server',
                'environment' => 'production',
                'created_by' => 'laravel-app'
            ]
        ]);

        return response()->json([
            'success' => true,
            'server' => $server->toArray(),
            'volume' => $volume->toArray(),
            'message' => 'Database server created successfully!'
        ]);

    } catch (\Exception $e) {
        return response()->json([
            'success' => false,
            'error' => $e->getMessage()
        ], 500);
    }
}

private function getDatabaseServerUserData(): string
{
    return '#cloud-config
packages:
  - mysql-server
  - mysql-client
  - php8.1-mysql
  - phpmyadmin
  - nginx

runcmd:
  - systemctl enable mysql
  - systemctl start mysql
  - systemctl enable nginx
  - systemctl start nginx
  
  # Configure MySQL
  - mysql -e "CREATE DATABASE IF NOT EXISTS app_database;"
  - mysql -e "CREATE USER IF NOT EXISTS '\''app_user'\''@'\''%'\'' IDENTIFIED BY '\''secure_password'\'';"
  - mysql -e "GRANT ALL PRIVILEGES ON app_database.* TO '\''app_user'\''@'\''%'\'';"
  - mysql -e "FLUSH PRIVILEGES;"
  
  # Mount the volume
  - mkdir -p /mnt/data
  - mount /dev/disk/by-id/scsi-0HC_Volume_' . $volume->id . ' /mnt/data
  - echo "/dev/disk/by-id/scsi-0HC_Volume_' . $volume->id . ' /mnt/data ext4 defaults 0 0" >> /etc/fstab
  
  # Move MySQL data to volume
  - systemctl stop mysql
  - cp -R /var/lib/mysql/* /mnt/data/
  - chown -R mysql:mysql /mnt/data
  - systemctl start mysql';
}
```

## Server Management Service

### Complete Server Management Service

```php
<?php

namespace App\Services;

use Boci\HetznerLaravel\Facades\HetznerLaravel;
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

class ServerManagementService
{
    public function createServer(array $data): array
    {
        try {
            $server = HetznerLaravel::servers()->create($data);
            
            return [
                'success' => true,
                'server' => $server->toArray(),
                'message' => 'Server created successfully'
            ];
        } catch (ErrorException $e) {
            return [
                'success' => false,
                'error' => 'API Error: ' . $e->getMessage(),
                'code' => $e->getCode()
            ];
        } catch (TransporterException $e) {
            return [
                'success' => false,
                'error' => 'Network Error: ' . $e->getMessage()
            ];
        }
    }

    public function getServer(string $serverId): array
    {
        try {
            $server = HetznerLaravel::servers()->retrieve($serverId);
            
            return [
                'success' => true,
                'server' => $server->toArray()
            ];
        } catch (ErrorException $e) {
            return [
                'success' => false,
                'error' => 'API Error: ' . $e->getMessage(),
                'code' => $e->getCode()
            ];
        }
    }

    public function listServers(array $filters = []): array
    {
        try {
            $servers = HetznerLaravel::servers()->list($filters);
            
            return [
                'success' => true,
                'servers' => $servers->toArray(),
                'count' => count($servers)
            ];
        } catch (ErrorException $e) {
            return [
                'success' => false,
                'error' => 'API Error: ' . $e->getMessage(),
                'code' => $e->getCode()
            ];
        }
    }

    public function updateServer(string $serverId, array $data): array
    {
        try {
            $server = HetznerLaravel::servers()->update($serverId, $data);
            
            return [
                'success' => true,
                'server' => $server->toArray(),
                'message' => 'Server updated successfully'
            ];
        } catch (ErrorException $e) {
            return [
                'success' => false,
                'error' => 'API Error: ' . $e->getMessage(),
                'code' => $e->getCode()
            ];
        }
    }

    public function deleteServer(string $serverId): array
    {
        try {
            HetznerLaravel::servers()->delete($serverId);
            
            return [
                'success' => true,
                'message' => 'Server deleted successfully'
            ];
        } catch (ErrorException $e) {
            return [
                'success' => false,
                'error' => 'API Error: ' . $e->getMessage(),
                'code' => $e->getCode()
            ];
        }
    }

    public function powerOnServer(string $serverId): array
    {
        try {
            $action = HetznerLaravel::servers()->actions()->powerOn($serverId);
            
            return [
                'success' => true,
                'action' => $action->toArray(),
                'message' => 'Server power on action initiated'
            ];
        } catch (ErrorException $e) {
            return [
                'success' => false,
                'error' => 'API Error: ' . $e->getMessage(),
                'code' => $e->getCode()
            ];
        }
    }

    public function rebootServer(string $serverId): array
    {
        try {
            $action = HetznerLaravel::servers()->actions()->reboot($serverId);
            
            return [
                'success' => true,
                'action' => $action->toArray(),
                'message' => 'Server reboot action initiated'
            ];
        } catch (ErrorException $e) {
            return [
                'success' => false,
                'error' => 'API Error: ' . $e->getMessage(),
                'code' => $e->getCode()
            ];
        }
    }

    public function shutdownServer(string $serverId): array
    {
        try {
            $action = HetznerLaravel::servers()->actions()->shutdown($serverId);
            
            return [
                'success' => true,
                'action' => $action->toArray(),
                'message' => 'Server shutdown action initiated'
            ];
        } catch (ErrorException $e) {
            return [
                'success' => false,
                'error' => 'API Error: ' . $e->getMessage(),
                'code' => $e->getCode()
            ];
        }
    }

    public function getServerMetrics(string $serverId, array $parameters = []): array
    {
        try {
            $metrics = HetznerLaravel::servers()->metrics($serverId, $parameters);
            
            return [
                'success' => true,
                'metrics' => $metrics->toArray()
            ];
        } catch (ErrorException $e) {
            return [
                'success' => false,
                'error' => 'API Error: ' . $e->getMessage(),
                'code' => $e->getCode()
            ];
        }
    }
}
```

## Server Templates

### Predefined Server Templates

```php
<?php

namespace App\Services;

use Boci\HetznerLaravel\Facades\HetznerLaravel;

class ServerTemplateService
{
    public function createWebServerTemplate(string $name, string $location): array
    {
        return [
            'name' => $name,
            'server_type' => 'cpx21', // 2 vCPU, 4 GB RAM
            'image' => 'ubuntu-24.04',
            'location' => $location,
            'ssh_keys' => ['my-ssh-key'],
            'user_data' => $this->getWebServerUserData(),
            'labels' => [
                'role' => 'web-server',
                'template' => 'web-server',
                'environment' => 'production'
            ]
        ];
    }

    public function createDatabaseServerTemplate(string $name, string $location): array
    {
        return [
            'name' => $name,
            'server_type' => 'cpx31', // 4 vCPU, 8 GB RAM
            'image' => 'ubuntu-24.04',
            'location' => $location,
            'ssh_keys' => ['my-ssh-key'],
            'user_data' => $this->getDatabaseServerUserData(),
            'labels' => [
                'role' => 'database-server',
                'template' => 'database-server',
                'environment' => 'production'
            ]
        ];
    }

    public function createLoadBalancerTemplate(string $name, string $location): array
    {
        return [
            'name' => $name,
            'server_type' => 'cpx11', // 1 vCPU, 2 GB RAM
            'image' => 'ubuntu-24.04',
            'location' => $location,
            'ssh_keys' => ['my-ssh-key'],
            'user_data' => $this->getLoadBalancerUserData(),
            'labels' => [
                'role' => 'load-balancer',
                'template' => 'load-balancer',
                'environment' => 'production'
            ]
        ];
    }

    private function getWebServerUserData(): string
    {
        return '#cloud-config
packages:
  - nginx
  - php8.1-fpm
  - php8.1-mysql
  - php8.1-curl
  - php8.1-gd
  - php8.1-mbstring
  - php8.1-xml
  - php8.1-zip
  - mysql-client
  - curl
  - git
  - unzip

runcmd:
  - systemctl enable nginx
  - systemctl start nginx
  - systemctl enable php8.1-fpm
  - systemctl start php8.1-fpm
  - mkdir -p /var/www/html
  - chown -R www-data:www-data /var/www/html';
    }

    private function getDatabaseServerUserData(): string
    {
        return '#cloud-config
packages:
  - mysql-server
  - mysql-client
  - php8.1-mysql
  - phpmyadmin
  - nginx

runcmd:
  - systemctl enable mysql
  - systemctl start mysql
  - systemctl enable nginx
  - systemctl start nginx
  
  # Configure MySQL
  - mysql -e "CREATE DATABASE IF NOT EXISTS app_database;"
  - mysql -e "CREATE USER IF NOT EXISTS '\''app_user'\''@'\''%'\'' IDENTIFIED BY '\''secure_password'\'';"
  - mysql -e "GRANT ALL PRIVILEGES ON app_database.* TO '\''app_user'\''@'\''%'\'';"
  - mysql -e "FLUSH PRIVILEGES;"';
    }

    private function getLoadBalancerUserData(): string
    {
        return '#cloud-config
packages:
  - nginx
  - haproxy
  - curl

runcmd:
  - systemctl enable nginx
  - systemctl start nginx
  - systemctl enable haproxy
  - systemctl start haproxy';
    }
}
```

## Usage in Controllers

```php
<?php

namespace App\Http\Controllers;

use App\Services\ServerManagementService;
use App\Services\ServerTemplateService;
use Illuminate\Http\Request;

class ServerController extends Controller
{
    public function __construct(
        private ServerManagementService $serverService,
        private ServerTemplateService $templateService
    ) {}

    public function createWebServer(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'location' => 'required|string',
        ]);

        $template = $this->templateService->createWebServerTemplate(
            $validated['name'],
            $validated['location']
        );

        $result = $this->serverService->createServer($template);

        return response()->json($result, $result['success'] ? 200 : 400);
    }

    public function createDatabaseServer(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'location' => 'required|string',
        ]);

        $template = $this->templateService->createDatabaseServerTemplate(
            $validated['name'],
            $validated['location']
        );

        $result = $this->serverService->createServer($template);

        return response()->json($result, $result['success'] ? 200 : 400);
    }

    public function listServers(Request $request)
    {
        $filters = $request->only(['name', 'status', 'label_selector']);
        $result = $this->serverService->listServers($filters);

        return response()->json($result, $result['success'] ? 200 : 400);
    }

    public function getServer(string $serverId)
    {
        $result = $this->serverService->getServer($serverId);

        return response()->json($result, $result['success'] ? 200 : 400);
    }

    public function updateServer(Request $request, string $serverId)
    {
        $validated = $request->validate([
            'name' => 'sometimes|string|max:255',
            'labels' => 'sometimes|array',
        ]);

        $result = $this->serverService->updateServer($serverId, $validated);

        return response()->json($result, $result['success'] ? 200 : 400);
    }

    public function deleteServer(string $serverId)
    {
        $result = $this->serverService->deleteServer($serverId);

        return response()->json($result, $result['success'] ? 200 : 400);
    }

    public function powerOnServer(string $serverId)
    {
        $result = $this->serverService->powerOnServer($serverId);

        return response()->json($result, $result['success'] ? 200 : 400);
    }

    public function rebootServer(string $serverId)
    {
        $result = $this->serverService->rebootServer($serverId);

        return response()->json($result, $result['success'] ? 200 : 400);
    }

    public function shutdownServer(string $serverId)
    {
        $result = $this->serverService->shutdownServer($serverId);

        return response()->json($result, $result['success'] ? 200 : 400);
    }
}
```

## Related Resources

- [API Reference - Servers](../api-reference/servers) - Complete server API documentation
- [DNS Management Examples](./dns-management) - Manage DNS for your servers
- [Network Setup Examples](./network-setup) - Configure private networks
- [Error Handling](../error-handling) - Handle errors gracefully
