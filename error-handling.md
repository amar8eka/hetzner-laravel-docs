---
sidebar_position: 4
---

# Error Handling

Proper error handling is crucial when working with the Hetzner Laravel package. This guide covers how to handle different types of errors and implement robust error management in your applications.

## Exception Types

The Hetzner Laravel package provides specific exception types for different error scenarios:

### 1. ErrorException

Thrown when the Hetzner Cloud API returns an error response (4xx, 5xx status codes).

```php
use Boci\HetznerLaravel\Exceptions\ErrorException;

try {
    $server = HetznerLaravel::servers()->create($parameters);
} catch (ErrorException $e) {
    // Handle API errors
    echo "API Error: " . $e->getMessage();
    echo "Error Code: " . $e->getCode();
    echo "Response Body: " . $e->getResponseBody();
}
```

### 2. TransporterException

Thrown when there are network or transport-related errors (timeouts, connection issues, etc.).

```php
use Boci\HetznerLaravel\Exceptions\TransporterException;

try {
    $server = HetznerLaravel::servers()->create($parameters);
} catch (TransporterException $e) {
    // Handle network/transport errors
    echo "Network Error: " . $e->getMessage();
}
```

## Common Error Scenarios

### 1. Authentication Errors

**Error Code**: 401 Unauthorized

```php
try {
    $servers = HetznerLaravel::servers()->list();
} catch (ErrorException $e) {
    if ($e->getCode() === 401) {
        // Invalid or missing API token
        return response()->json([
            'error' => 'Authentication failed. Please check your API token.',
            'code' => 401
        ], 401);
    }
}
```

### 2. Resource Not Found

**Error Code**: 404 Not Found

```php
try {
    $server = HetznerLaravel::servers()->ret('non-existent-id');
} catch (ErrorException $e) {
    if ($e->getCode() === 404) {
        return response()->json([
            'error' => 'Server not found.',
            'code' => 404
        ], 404);
    }
}
```

### 3. Validation Errors

**Error Code**: 422 Unprocessable Entity

```php
try {
    $server = HetznerLaravel::servers()->create([
        'name' => '', // Invalid empty name
        'server_type' => 'invalid-type',
    ]);
} catch (ErrorException $e) {
    if ($e->getCode() === 422) {
        $responseBody = json_decode($e->getResponseBody(), true);
        $errors = $responseBody['errors'] ?? [];
        
        return response()->json([
            'error' => 'Validation failed',
            'validation_errors' => $errors,
            'code' => 422
        ], 422);
    }
}
```

### 4. Rate Limiting

**Error Code**: 429 Too Many Requests

```php
try {
    $servers = HetznerLaravel::servers()->list();
} catch (ErrorException $e) {
    if ($e->getCode() === 429) {
        return response()->json([
            'error' => 'Rate limit exceeded. Please try again later.',
            'code' => 429
        ], 429);
    }
}
```

### 5. Server Errors

**Error Code**: 5xx Server Errors

```php
try {
    $server = HetznerLaravel::servers()->create($parameters);
} catch (ErrorException $e) {
    if ($e->getCode() >= 500) {
        return response()->json([
            'error' => 'Hetzner Cloud service is temporarily unavailable.',
            'code' => $e->getCode()
        ], 503);
    }
}
```

## Comprehensive Error Handler

### Service-Level Error Handling

```php
<?php

namespace App\Services;

use Boci\HetznerLaravel\Facades\HetznerLaravel;
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;
use Illuminate\Support\Facades\Log;

class HetznerErrorHandler
{
    public function handleApiCall(callable $apiCall, string $operation = 'API call'): array
    {
        try {
            $result = $apiCall();
            
            return [
                'success' => true,
                'data' => $result,
                'message' => ucfirst($operation) . ' completed successfully'
            ];
            
        } catch (ErrorException $e) {
            return $this->handleApiError($e, $operation);
            
        } catch (TransporterException $e) {
            return $this->handleTransportError($e, $operation);
            
        } catch (\Exception $e) {
            return $this->handleGenericError($e, $operation);
        }
    }

    private function handleApiError(ErrorException $e, string $operation): array
    {
        $errorCode = $e->getCode();
        $errorMessage = $e->getMessage();
        $responseBody = $e->getResponseBody();

        // Log the error
        Log::error("Hetzner API Error during {$operation}", [
            'code' => $errorCode,
            'message' => $errorMessage,
            'response_body' => $responseBody,
            'operation' => $operation
        ]);

        switch ($errorCode) {
            case 401:
                return [
                    'success' => false,
                    'error' => 'Authentication failed. Please check your API token.',
                    'code' => $errorCode,
                    'type' => 'authentication_error'
                ];

            case 403:
                return [
                    'success' => false,
                    'error' => 'Access forbidden. You do not have permission to perform this action.',
                    'code' => $errorCode,
                    'type' => 'permission_error'
                ];

            case 404:
                return [
                    'success' => false,
                    'error' => 'Resource not found.',
                    'code' => $errorCode,
                    'type' => 'not_found_error'
                ];

            case 422:
                $validationErrors = $this->parseValidationErrors($responseBody);
                return [
                    'success' => false,
                    'error' => 'Validation failed.',
                    'code' => $errorCode,
                    'type' => 'validation_error',
                    'validation_errors' => $validationErrors
                ];

            case 429:
                return [
                    'success' => false,
                    'error' => 'Rate limit exceeded. Please try again later.',
                    'code' => $errorCode,
                    'type' => 'rate_limit_error'
                ];

            case 500:
            case 502:
            case 503:
            case 504:
                return [
                    'success' => false,
                    'error' => 'Hetzner Cloud service is temporarily unavailable. Please try again later.',
                    'code' => $errorCode,
                    'type' => 'server_error'
                ];

            default:
                return [
                    'success' => false,
                    'error' => "API Error: {$errorMessage}",
                    'code' => $errorCode,
                    'type' => 'api_error'
                ];
        }
    }

    private function handleTransportError(TransporterException $e, string $operation): array
    {
        Log::error("Hetzner Transport Error during {$operation}", [
            'message' => $e->getMessage(),
            'operation' => $operation
        ]);

        return [
            'success' => false,
            'error' => 'Network error occurred. Please check your connection and try again.',
            'code' => 0,
            'type' => 'transport_error'
        ];
    }

    private function handleGenericError(\Exception $e, string $operation): array
    {
        Log::error("Unexpected Error during {$operation}", [
            'message' => $e->getMessage(),
            'operation' => $operation,
            'trace' => $e->getTraceAsString()
        ]);

        return [
            'success' => false,
            'error' => 'An unexpected error occurred. Please try again.',
            'code' => 0,
            'type' => 'generic_error'
        ];
    }

    private function parseValidationErrors(string $responseBody): array
    {
        $decoded = json_decode($responseBody, true);
        return $decoded['errors'] ?? [];
    }
}
```

### Usage in Services

```php
<?php

namespace App\Services;

use Boci\HetznerLaravel\Facades\HetznerLaravel;

class ServerService
{
    public function __construct(
        private HetznerErrorHandler $errorHandler
    ) {}

    public function createServer(array $data): array
    {
        return $this->errorHandler->handleApiCall(
            fn() => HetznerLaravel::servers()->create($data),
            'server creation'
        );
    }

    public function getServer(string $serverId): array
    {
        return $this->errorHandler->handleApiCall(
            fn() => HetznerLaravel::servers()->retrieve($serverId),
            'server retrieval'
        );
    }

    public function listServers(array $filters = []): array
    {
        return $this->errorHandler->handleApiCall(
            fn() => HetznerLaravel::servers()->list($filters),
            'server listing'
        );
    }

    public function updateServer(string $serverId, array $data): array
    {
        return $this->errorHandler->handleApiCall(
            fn() => HetznerLaravel::servers()->update($serverId, $data),
            'server update'
        );
    }

    public function deleteServer(string $serverId): array
    {
        return $this->errorHandler->handleApiCall(
            fn() => HetznerLaravel::servers()->delete($serverId),
            'server deletion'
        );
    }
}
```

## Controller Error Handling

```php
<?php

namespace App\Http\Controllers;

use App\Services\ServerService;
use Illuminate\Http\Request;

class ServerController extends Controller
{
    public function __construct(
        private ServerService $serverService
    ) {}

    public function create(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'server_type' => 'required|string',
            'image' => 'required|string',
            'location' => 'required|string',
        ]);

        $result = $this->serverService->createServer($validated);

        if (!$result['success']) {
            return $this->handleErrorResponse($result);
        }

        return response()->json($result, 201);
    }

    public function show(string $serverId)
    {
        $result = $this->serverService->getServer($serverId);

        if (!$result['success']) {
            return $this->handleErrorResponse($result);
        }

        return response()->json($result);
    }

    public function index(Request $request)
    {
        $filters = $request->only(['name', 'status', 'label_selector']);
        $result = $this->serverService->listServers($filters);

        if (!$result['success']) {
            return $this->handleErrorResponse($result);
        }

        return response()->json($result);
    }

    public function update(Request $request, string $serverId)
    {
        $validated = $request->validate([
            'name' => 'sometimes|string|max:255',
            'labels' => 'sometimes|array',
        ]);

        $result = $this->serverService->updateServer($serverId, $validated);

        if (!$result['success']) {
            return $this->handleErrorResponse($result);
        }

        return response()->json($result);
    }

    public function destroy(string $serverId)
    {
        $result = $this->serverService->deleteServer($serverId);

        if (!$result['success']) {
            return $this->handleErrorResponse($result);
        }

        return response()->json($result);
    }

    private function handleErrorResponse(array $result): \Illuminate\Http\JsonResponse
    {
        $errorType = $result['type'] ?? 'unknown_error';
        $errorCode = $result['code'] ?? 500;

        // Map error types to HTTP status codes
        $statusCodeMap = [
            'authentication_error' => 401,
            'permission_error' => 403,
            'not_found_error' => 404,
            'validation_error' => 422,
            'rate_limit_error' => 429,
            'server_error' => 503,
            'transport_error' => 503,
            'generic_error' => 500,
        ];

        $statusCode = $statusCodeMap[$errorType] ?? 500;

        return response()->json([
            'success' => false,
            'error' => $result['error'],
            'type' => $errorType,
            'code' => $errorCode,
            'validation_errors' => $result['validation_errors'] ?? null,
        ], $statusCode);
    }
}
```

## Retry Logic

For transient errors, implement retry logic:

```php
<?php

namespace App\Services;

use Boci\HetznerLaravel\Facades\HetznerLaravel;
use Boci\HetznerLaravel\Exceptions\ErrorException;
use Boci\HetznerLaravel\Exceptions\TransporterException;

class RetryableHetznerService
{
    private int $maxRetries = 3;
    private int $retryDelay = 1000; // milliseconds

    public function createServerWithRetry(array $data): array
    {
        $attempt = 0;
        $lastError = null;

        while ($attempt < $this->maxRetries) {
            try {
                $server = HetznerLaravel::servers()->create($data);
                
                return [
                    'success' => true,
                    'server' => $server->toArray(),
                    'attempts' => $attempt + 1
                ];
                
            } catch (ErrorException $e) {
                $lastError = $e;
                
                // Don't retry on client errors (4xx)
                if ($e->getCode() < 500) {
                    break;
                }
                
                // Retry on server errors (5xx)
                if ($e->getCode() >= 500) {
                    $attempt++;
                    if ($attempt < $this->maxRetries) {
                        usleep($this->retryDelay * 1000); // Convert to microseconds
                        continue;
                    }
                }
                
            } catch (TransporterException $e) {
                $lastError = $e;
                $attempt++;
                
                if ($attempt < $this->maxRetries) {
                    usleep($this->retryDelay * 1000);
                    continue;
                }
            }
        }

        return [
            'success' => false,
            'error' => $lastError->getMessage(),
            'attempts' => $attempt,
            'type' => 'retry_exhausted'
        ];
    }
}
```

## Logging and Monitoring

### Custom Log Channel

```php
// config/logging.php
'channels' => [
    'hetzner' => [
        'driver' => 'daily',
        'path' => storage_path('logs/hetzner.log'),
        'level' => 'error',
        'days' => 14,
    ],
],
```

### Error Monitoring Service

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Mail;

class ErrorMonitoringService
{
    public function logError(string $operation, \Exception $e, array $context = []): void
    {
        Log::channel('hetzner')->error("Hetzner Error in {$operation}", [
            'message' => $e->getMessage(),
            'code' => $e->getCode(),
            'file' => $e->getFile(),
            'line' => $e->getLine(),
            'trace' => $e->getTraceAsString(),
            'context' => $context,
        ]);

        // Send alert for critical errors
        if ($this->isCriticalError($e)) {
            $this->sendAlert($operation, $e, $context);
        }
    }

    private function isCriticalError(\Exception $e): bool
    {
        // Define what constitutes a critical error
        return $e->getCode() >= 500 || 
               str_contains($e->getMessage(), 'authentication') ||
               str_contains($e->getMessage(), 'permission');
    }

    private function sendAlert(string $operation, \Exception $e, array $context): void
    {
        // Send email alert to administrators
        Mail::raw("Critical Hetzner Error Alert\n\nOperation: {$operation}\nError: {$e->getMessage()}\nCode: {$e->getCode()}", function ($message) {
            $message->to('admin@example.com')
                   ->subject('Critical Hetzner Error Alert');
        });
    }
}
```

## Best Practices

### 1. Always Use Try-Catch Blocks

```php
// Good
try {
    $server = HetznerLaravel::servers()->create($data);
} catch (ErrorException $e) {
    // Handle error
}

// Bad
$server = HetznerLaravel::servers()->create($data); // Can throw exceptions
```

### 2. Provide User-Friendly Error Messages

```php
try {
    $server = HetznerLaravel::servers()->create($data);
} catch (ErrorException $e) {
    if ($e->getCode() === 422) {
        return response()->json([
            'error' => 'Please check your input data and try again.',
            'details' => $e->getMessage()
        ], 422);
    }
}
```

### 3. Log Errors for Debugging

```php
try {
    $server = HetznerLaravel::servers()->create($data);
} catch (\Exception $e) {
    Log::error('Failed to create server', [
        'error' => $e->getMessage(),
        'data' => $data
    ]);
    
    throw $e; // Re-throw or handle appropriately
}
```

### 4. Implement Circuit Breaker Pattern

For high-traffic applications, consider implementing a circuit breaker pattern to prevent cascading failures.

## Related Resources

- [Troubleshooting Guide](./troubleshooting) - Common issues and solutions
- [API Reference](../api-reference/servers) - Complete API documentation
- [Examples](../examples/server-management) - Practical usage examples
