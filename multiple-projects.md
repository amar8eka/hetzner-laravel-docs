---
sidebar_position: 4
---

# Multiple Projects

Every Hetzner Cloud API token belongs to exactly one project. If your application manages servers in more than one project — for example `production` and `staging`, or one project per customer — you can configure a token per project and switch between them.

## Configuration

Publish the configuration file if you haven't already:

```bash
php artisan vendor:publish --provider="Boci\HetznerLaravel\HetznerLaravelServiceProvider" --tag="config"
```

Add an entry for each project to `config/hetzner-laravel.php`:

```php
// config/hetzner-laravel.php
return [

    'default' => env('HETZNER_PROJECT', 'default'),

    'projects' => [

        'default' => [
            'token' => env('HETZNER_TOKEN'),
            'base_url' => env('HETZNER_BASE_URL', 'https://api.hetzner.cloud'),
        ],

        'staging' => [
            'token' => env('HETZNER_STAGING_TOKEN'),
        ],

    ],

];
```

And the tokens to your `.env` file:

```env
HETZNER_TOKEN=your-production-token
HETZNER_STAGING_TOKEN=your-staging-token
```

`default` is the project used whenever you don't name one. Set `HETZNER_PROJECT` to use another configured project as the default.

## Using a Project

Everything you already do with the facade keeps using the default project:

```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;

$servers = HetznerLaravel::servers()->list();
```

Call `project()` with the name of a configured project to use its token:

```php
$servers = HetznerLaravel::project('staging')->servers()->list();

$response = HetznerLaravel::project('staging')->servers()->create([
    'name' => 'staging-web-1',
    'server_type' => 'cpx22',
    'image' => 'ubuntu-24.04',
    'location' => 'nbg1',
]);
```

`project()` returns the same `Client` you get from `Client::factory()`, so every resource is available. The client of each project is created once and reused for the rest of the request.

## Runtime Tokens

When tokens are not in your config — for example each team stores its own Hetzner token in the database — use `withToken()`:

```php
$client = HetznerLaravel::withToken($team->hetzner_token);

foreach ($client->servers()->list()->servers() as $server) {
    echo $server->name() . "\n";
}
```

`withToken()` always returns a new client, so one team's token is never reused for another.

## Changing the Default Project

You can switch the default project at runtime, for example in a middleware that picks the project from the current tenant:

```php
use Boci\HetznerLaravel\Facades\HetznerLaravel;
use Closure;
use Illuminate\Http\Request;

class UseHetznerProject
{
    public function handle(Request $request, Closure $next)
    {
        HetznerLaravel::setDefaultProject($request->user()->team->hetzner_project);

        return $next($request);
    }
}
```

After this, `HetznerLaravel::servers()` and an injected `Client` both use that project.

## Dependency Injection

Injecting `Client` gives you the default project, as before. Inject `HetznerManager` when a class needs to work with several projects:

```php
use Boci\HetznerLaravel\HetznerManager;

class SyncServers
{
    public function __construct(
        private HetznerManager $hetzner
    ) {}

    public function handle(): void
    {
        foreach ($this->hetzner->projects() as $project) {
            $servers = $this->hetzner->project($project)->servers()->list();

            // ...
        }
    }
}
```

## Available Methods

| Method | Description |
| --- | --- |
| `project(?string $name = null)` | Get the client of a configured project, or of the default project |
| `withToken(string $token, ?string $baseUri = null)` | Get a new client for a token known only at runtime |
| `getDefaultProject()` | Get the name of the default project |
| `setDefaultProject(string $name)` | Change the default project |
| `projects()` | Get the names of all configured projects |
| `forgetProject(?string $name = null)` | Forget a resolved client, e.g. after its token changed |

## Errors

Using a project that isn't configured, or a project without a token, throws an `InvalidArgumentException`:

```php
HetznerLaravel::project('unknown');
// InvalidArgumentException: Hetzner project [unknown] is not configured.

HetznerLaravel::project('staging');
// InvalidArgumentException: Hetzner project [staging] has no API token.
```

## Upgrading

Configuration files published before multiple projects were supported only have a `token` key:

```php
return [
    'token' => env('HETZNER_TOKEN'),
];
```

They keep working without any change, and the token is used for the default project. Republish the configuration file with `--force` when you want to add more projects.
