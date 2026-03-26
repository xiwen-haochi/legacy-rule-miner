# PHP — Language-Specific Analysis

Read this file when the project is detected as PHP (composer.json, or .php files at root).

## Stack Detection

### Version Detection
```bash
# PHP version constraint
grep '"php"' composer.json

# Framework detection
grep -i "laravel\|thinkphp\|yii\|codeigniter\|symfony" composer.json

# ThinkPHP version
grep "topthink/framework" composer.json

# Laravel version
grep "laravel/framework" composer.json

# Yii version
grep "yiisoft/yii2" composer.json
```

### Common Stack Variants

| Variant | Key Indicators | Era |
|---------|---------------|-----|
| ThinkPHP 5.0 | `topthink/framework ^5.0` | 2016–2018 |
| ThinkPHP 5.1 | `topthink/framework ^5.1` | 2018–2020 |
| ThinkPHP 6.0 | `topthink/framework ^6.0` | 2020+ |
| Laravel 5.x | `laravel/framework 5.*` | 2015–2019 |
| Laravel 6.x | `laravel/framework ^6.0` | 2019–2020 |
| Laravel 7.x/8.x | `laravel/framework ^7.0 / ^8.0` | 2020–2021 |
| Yii 2 | `yiisoft/yii2 ~2.0` | 2014–2020 |
| Raw PHP | No framework, custom bootstrap | Any |

---

## PHP-Specific Analysis Points

### 1. Project Structure Patterns

#### Files to Exclude from Code Style Analysis
Skip these when sampling conventions (still read migration files for schema info):
- `vendor/` — Composer dependencies
- `storage/` — Laravel cache, logs, sessions
- `bootstrap/cache/` — Laravel compiled config/routes
- `runtime/` — ThinkPHP runtime cache, logs, temp files
- `database/migrations/` — Laravel migrations (read for schema only)
- `*.blade.php` compiled cache — not authored code
- `compiled.php` — framework compiled files
- `public/assets/`, `public/uploads/` — static files

#### ThinkPHP 5.x
```
application/
├── common/           # Common module
├── index/            # Frontend module
│   ├── controller/
│   │   ├── Index.php          # Frontend: homepage for web visitors
│   │   └── User.php           # Frontend: user operations for web/mobile
│   ├── model/
│   └── view/
├── admin/            # Admin module
│   ├── controller/
│   │   ├── Index.php          # Admin: dashboard for admin panel
│   │   └── User.php           # Admin: user management for administrators
│   ├── model/
│   └── view/
├── api/              # API module
│   ├── controller/
│   │   ├── User.php           # Third-party: user data API for external integrators
│   │   └── Order.php          # Third-party: order query API for partners
├── common.php        # Common functions
├── config.php        # Main config
├── database.php      # DB config
└── route.php         # Route definitions
```

**ThinkPHP module = audience boundary**: Each module (`index/`, `admin/`, `api/`) typically serves a different audience. Controllers with the same name (e.g., `User.php`) across modules handle the same entity but for different audiences — document this difference.

#### Laravel 5.x / 6.x
```
app/
├── Console/
├── Exceptions/
├── Http/
│   ├── Controllers/
│   │   ├── UserController.php         # Frontend: user CRUD for web/mobile
│   │   ├── Api/
│   │   │   └── UserController.php     # Third-party: user data for external API
│   │   └── Admin/
│   │       └── UserController.php     # Admin: user management for admin panel
│   ├── Middleware/
│   └── Requests/
├── Models/           # May be directly in app/ in Laravel 5.x
├── Providers/
├── Services/         # Custom, not part of default structure
│   ├── UserService.php            # API-layer: user business logic
│   ├── PaymentService.php         # Integration: wraps payment provider
│   └── NotificationService.php   # Internal-utility: handles email/SMS
├── Jobs/             # Scheduled-task / queue workers
│   └── SyncOrderJob.php          # Scheduled-task: sync orders from external
└── Listeners/
    └── PaymentReceivedListener.php # Webhook: handles payment callback
config/
database/migrations/
resources/views/
routes/
├── api.php            # Third-party API routes (auth:api middleware)
├── web.php            # Frontend routes (session auth middleware)
└── admin.php          # Admin routes (custom, if exists)
```

#### File Role Detection for PHP

| Clue | How to Detect | What It Means |
|------|--------------|---------------|
| Module directory (ThinkPHP) | `application/index/`, `application/admin/`, `application/api/` | Module = audience |
| Route file (Laravel) | `routes/api.php` vs `routes/web.php` vs `routes/admin.php` | Route file = audience grouping |
| Controller namespace | `App\Http\Controllers\Api\*` vs `App\Http\Controllers\Admin\*` | Namespace = audience |
| Middleware group | `['middleware' => 'auth:api']` vs `['middleware' => 'auth']` vs `['middleware' => 'admin']` | Auth type = audience |
| Base controller | `extends BaseApiController` vs `extends Controller` | Different behavior per audience |

**Disambiguation**: When `app/Http/Controllers/UserController.php` and `app/Http/Controllers/Api/UserController.php` both exist, compare:
- Route bindings (which route file registers them)
- Middleware (session auth vs API token/key)
- Response format (view vs JSON envelope)
- Document: "`UserController` renders Blade views for frontend; `Api\UserController` returns JSON for mobile/third-party consumers"

### 2. Routing Patterns

#### ThinkPHP 5.x
```php
// route.php — may use annotation or configuration
Route::rule('user/:id', 'index/User/read');
// Or controller auto-routing: /module/controller/action
// e.g., /admin/User/index maps to application/admin/controller/User.php::index()
```

#### Laravel
```php
// routes/api.php or routes/web.php
Route::group(['prefix' => 'api', 'middleware' => 'auth'], function () {
    Route::get('/users', 'UserController@index');
    Route::post('/users', 'UserController@store');
});
// Or resource routes
Route::resource('users', 'UserController');
```

**Key things to check:**
- Is route definition centralized or using auto-routing?
- Middleware applied at route group level or controller constructor?
- Route naming conventions

### 3. Database / ORM Patterns

#### ThinkPHP Model
```php
// Check if using Db facade or Model class
Db::table('user')->where('id', $id)->find();
// vs
User::get($id);
// vs
$this->where('id', $id)->find();
```

#### Laravel Eloquent
```php
// Query Builder
DB::table('users')->where('id', $id)->first();
// vs Eloquent Model
User::find($id);
User::where('status', 1)->get();
// vs Raw SQL
DB::select('SELECT * FROM users WHERE id = ?', [$id]);
```

Check:
- [ ] Which query style dominates (Query Builder vs Model vs Raw SQL)
- [ ] Table naming: singular vs plural
- [ ] Soft delete usage (`SoftDeletes` trait / `deleteTime` field)
- [ ] Timestamps: auto-managed or manual
- [ ] Model relationships defined or manual joins
- [ ] Pagination: `paginate()` vs manual LIMIT/OFFSET

### 4. Response Format

```bash
# Find response helpers
grep -rn "json(\|success(\|error(\|return \$this->success\|return json(" --include="*.php" | head -20
```

Common patterns:
```php
// ThinkPHP 5.x
return json(['code' => 1, 'msg' => 'success', 'data' => $data]);
// or using helper
$this->success('操作成功', $data);
$this->error('操作失败');

// Laravel
return response()->json(['code' => 0, 'message' => 'ok', 'data' => $data]);
// or custom ApiResponse trait
return $this->success($data);
```

### 5. Authentication Patterns

```bash
# ThinkPHP
grep -rn "session\|Session::\|cookie\|Cookie::\|jwt\|token" --include="*.php" | head -20

# Laravel
grep -rn "Auth::\|auth()\|middleware.*auth\|guard\|passport\|sanctum" --include="*.php" | head -20
```

Check:
- Session-based or Token-based (JWT)?
- Custom auth middleware or framework-provided?
- Multi-guard setup (admin vs user)?

### 6. Validation Patterns

#### ThinkPHP 5.x
```php
// Validate class
$validate = new UserValidate();
if (!$validate->check($data)) { ... }
// or inline
$this->validate($data, 'User');
```

#### Laravel
```php
// Form Request validation
public function store(CreateUserRequest $request) { ... }
// or inline
$request->validate(['name' => 'required|string|max:255']);
// or manual Validator
$validator = Validator::make($data, $rules);
```

### 7. Error Handling

```bash
# Find exception handling
find . -name "*.php" -path "*/exception*" -o -name "*.php" -path "*/Exception*" | head -10
grep -rn "try\s*{" --include="*.php" | wc -l
grep -rn "catch\s*(" --include="*.php" | wc -l
```

Check:
- Global exception handler location
- Custom exception classes
- Error code system
- How exceptions are logged

### 8. Template / View Patterns

```bash
# Check if frontend is separated
find . -name "*.blade.php" -o -name "*.html" -path "*/view*" | wc -l
```

If views exist (not API-only):
- Template engine: Blade (Laravel), Think template, Twig, raw PHP
- Layout/inheritance pattern
- Asset compilation (Mix, Webpack, or direct reference)

### 9. Namespace & Autoloading

```bash
# Check PSR-4 autoload config
grep -A 10 '"psr-4"' composer.json
```

Note:
- Root namespace mapping
- Whether classes follow PSR-4 strictly
- Common namespace prefixes

### 10. Configuration Management

### 11. Use Statement Consistency
- Check `use` alias patterns: `use App\Models\User` vs `use App\Models\User as UserModel`
- Identify whether the project prefers fully qualified class names or `use` imports
- Check for consistent `use function` / `use const` patterns
- Namespace alias conventions for long paths (e.g., `use Illuminate\Support\Facades\DB` everywhere or aliased)

#### ThinkPHP 5.x
```php
// config.php, database.php, or module-level config
config('database.hostname')
Config::get('app.debug')
```

#### Laravel
```php
// config/*.php + .env
config('app.debug')
env('DB_HOST')
```

Check:
- Environment-specific configuration approach (.env vs multiple config files)
- How secrets are managed
- Custom helper functions loaded from common.php or helpers.php

---

## Common Anti-Patterns in Legacy PHP

Things AI will want to "fix" but should NOT (unless specifically asked):

1. **SQL in controllers** — AI will try to move to model/repository. In ThinkPHP legacy code, this is often standard practice.
2. **Long controller methods** (500+ lines) — AI will try to extract services. Don't refactor the structure.
3. **Mixed HTML and PHP** — In old ThinkPHP projects with views, template code may be tightly coupled. Don't try to "separate concerns."
4. **Global functions in common.php** — AI will try to convert to class methods. These functions are used everywhere.
5. **Raw SQL queries** — AI will try to convert to ORM. If the project mixes both, maintain whichever style the file uses.
6. **No namespace** (very old projects) — AI must not add namespace declarations to files that don't have them.
7. **Array-style configs** — AI will try to use object/class-based configs.
8. **include/require files** — Some legacy projects use file inclusion instead of autoloading. Don't touch this.

---

## File Role & Usage Scenario Patterns

When analyzing a PHP project, classify every controller and service file by its role and audience.

### ThinkPHP: Module-Based Audience Separation

ThinkPHP uses modules as the primary audience separator:
```
application/
├── index/           # Frontend-facing (serves web users)
│   ├── controller/
│   │   ├── Index.php    # Homepage controller (Frontend)
│   │   ├── User.php     # User profile for frontend (Frontend)
│   │   └── Order.php    # Order operations for customers (Frontend)
│   └── model/
├── admin/           # Admin panel (serves admin users)
│   ├── controller/
│   │   ├── User.php     # User management for admin (Admin)
│   │   └── Order.php    # Order management for admin (Admin)
│   └── model/
├── api/             # API for mobile app or third-party (Third-party/Frontend)
│   ├── controller/
│   │   ├── User.php     # User API (API consumers)
│   │   └── Pay.php      # Payment API (API consumers)
│   └── model/
└── common/          # Shared logic (Internal-utility)
    ├── model/
    └── service/
```

**Key**: The same entity name (e.g., `User.php`) exists in multiple modules with different audiences. Each has different auth middleware, response format, and permission checks. Always note which module a file belongs to.

### Laravel: Route File + Namespace-Based Separation

**Route files determine audience:**
| Route File | Audience | Default Middleware | URL Prefix |
|-----------|----------|-------------------|------------|
| `routes/web.php` | Frontend | `web` (session, CSRF) | `/` |
| `routes/api.php` | API consumers | `api` (stateless, throttle) | `/api` |
| `routes/admin.php` | Admin | `web` + `auth:admin` | `/admin` |
| `routes/channels.php` | WebSocket | `auth` | broadcast channels |

**Controller namespace separation:**
```
app/Http/Controllers/
├── UserController.php           # Frontend (routes/web.php)
├── Api/
│   ├── UserController.php       # API consumers (routes/api.php)
│   └── V2/
│       └── UserController.php   # API v2 (versioned API)
├── Admin/
│   └── UserController.php       # Admin panel (routes/admin.php)
└── Webhook/
    └── PaymentController.php    # Webhook receiver (payment callback)
```

**Service layer disambiguation:**
| File | Role | Clues |
|------|------|-------|
| `app/Services/UserService.php` | API-layer service | Called by controllers |
| `app/Services/PaymentGatewayService.php` | Integration service | Calls third-party payment SDK |
| `app/Jobs/SyncOrderJob.php` | Scheduled task / queue worker | Implements `ShouldQueue` |
| `app/Listeners/OrderPaidListener.php` | Event-driven | Listens to `OrderPaid` event |
| `app/Console/Commands/CleanExpiredTokens.php` | Scheduled task | Registered in `Kernel.php` schedule |

### Detection Commands
```bash
# ThinkPHP: List all modules and their controllers
find application/ -name "*.php" -path "*/controller/*" | sort

# Laravel: List all route files and their prefixes
grep -rn "Route::group\|Route::prefix\|Route::middleware" routes/ | head -20

# Laravel: List controller namespaces
find app/Http/Controllers -name "*.php" | sort

# Find auth middleware differences
grep -rn "middleware.*auth\|middleware.*admin\|middleware.*api" routes/ | head -20
```

---

## Known Version Boundaries (Reference)

> The following are feature boundary references for known versions. If the project uses a version not listed here, analyze based on actually detected version features and note version constraints in the generated rules.

### PHP 7.0
- Scalar type declarations available but may not be used
- No nullable types (?string)
- No void return type
- No multi-catch (catch (TypeA | TypeB))

### PHP 7.1
- Nullable types available (?int)
- void return type
- Array destructuring with []
- Class constant visibility

### PHP 7.2
- object type hint
- No trailing comma in function calls

### PHP 7.4
- Arrow functions: `fn($x) => $x * 2`
- Typed properties
- Null coalescing assignment: `??=`

### PHP 8.0+
- Named arguments
- Match expression
- Constructor promotion
- Union types
- Nullsafe operator: `?->`

Check the `require` PHP version in composer.json and ensure AI doesn't use features above that version.

### Other Versions
For PHP 8.1+, ThinkPHP 6.x+, Laravel 9+, or other versions not listed above — detect the actual language features and framework API from the project's code. Record differences from the known versions above and flag as `LOW_CONFIDENCE` for user confirmation.

---

## Key Dependencies to Track

| Dependency | Why It Matters |
|-----------|---------------|
| `topthink/framework` | ThinkPHP version determines available features |
| `laravel/framework` | Laravel version — API changes significantly between versions |
| `overtrue/wechat` / `easywechat` | WeChat integration — extremely common in Chinese PHP projects |
| `tymon/jwt-auth` | JWT authentication for Laravel |
| `dingo/api` | API layer for Laravel — has own routing and response format |
| `intervention/image` | Image processing — version matters |
| `phpoffice/phpspreadsheet` | Excel import/export |
| `predis/predis` / `ext-redis` | Redis client — method naming differs |
| `guzzlehttp/guzzle` | HTTP client — version 6 vs 7 have different API |
| `topthink/think-queue` | ThinkPHP queue — specific job class patterns |
