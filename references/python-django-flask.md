# Python — Language-Specific Analysis

Read this file when the project is detected as Python (requirements.txt, Pipfile, pyproject.toml, setup.py, or Django/Flask indicators).

## Stack Detection

### Version Detection
```bash
# Python version
grep "python_requires" setup.py setup.cfg pyproject.toml 2>/dev/null
cat runtime.txt 2>/dev/null  # Heroku-style
cat .python-version 2>/dev/null  # pyenv

# Django version
grep -i "django" requirements.txt Pipfile pyproject.toml 2>/dev/null

# Flask version
grep -i "flask" requirements.txt Pipfile pyproject.toml 2>/dev/null
```

### Common Stack Variants

| Variant | Key Indicators | Era |
|---------|---------------|-----|
| Django 1.11 | `Django>=1.11,<2.0` | 2017–2018 |
| Django 2.0–2.2 | `Django>=2.0,<3.0` | 2018–2020 |
| Django 3.0–3.2 | `Django>=3.0,<4.0` | 2020–2022 |
| Flask 0.12–1.0 | `Flask>=0.12,<1.1` | 2016–2019 |
| Flask 1.x–2.x | `Flask>=1.0,<3.0` | 2019–2022 |
| FastAPI | `fastapi` in deps | 2020+ |
| Tornado | `tornado` | 2012–2018 |

---

## Python-Specific Analysis Points

### 1. Project Structure Patterns

#### Files to Exclude from Code Style Analysis
Skip these when sampling conventions (still read migration files for schema info):
- `__pycache__/`, `*.pyc`, `*.pyo` — compiled bytecode
- `*/migrations/*.py` (except `__init__.py`) — Django auto-generated migrations (read for schema only)
- `*.egg-info/`, `dist/`, `build/` — packaging output
- `.tox/`, `.venv/`, `venv/`, `env/` — virtual environments
- `static/`, `media/`, `staticfiles/` — collected static files
- `*.min.js`, `*.min.css` in static dirs — minified assets

#### Django
```
project/
├── manage.py
├── project/              # Project settings package
│   ├── __init__.py
│   ├── settings.py       # or settings/ directory
│   ├── urls.py           # Root URL conf — routes to app URLs
│   └── wsgi.py
├── app1/                 # Django app
│   ├── __init__.py
│   ├── admin.py          # Admin: registers models in Django admin panel
│   ├── apps.py           # App configuration
│   ├── models.py         # Data access: ORM model definitions
│   ├── serializers.py    # DRF: request/response serialization
│   ├── urls.py           # URL routing for this app
│   ├── views.py          # Frontend/API: request handlers
│   └── tests.py
├── app2/
├── requirements.txt
└── templates/
```

#### Flask
```
project/
├── app/
│   ├── __init__.py       # App factory
│   ├── models/
│   │   ├── user.py       # Data access: user model
│   │   └── order.py      # Data access: order model
│   ├── views/ or routes/
│   │   ├── user.py       # Frontend: user views for web/mobile clients
│   │   ├── api.py        # Third-party: API endpoints for external integrators
│   │   └── admin.py      # Admin: admin panel views
│   ├── services/
│   │   ├── user_service.py        # API-layer: user business logic
│   │   ├── payment_service.py     # Integration: wraps payment provider
│   │   └── notification_service.py # Internal-utility: email/SMS sender
│   ├── tasks/
│   │   └── sync_orders.py         # Scheduled-task: Celery task for order sync
│   ├── utils/
│   └── templates/
├── config.py
├── requirements.txt
└── run.py / wsgi.py
```

#### File Role Detection for Python

| Clue | How to Detect | What It Means |
|------|--------------|---------------|
| URL prefix | `url(r'^api/v1/', ...)`, `@app.route('/admin/')`, `@app.route('/open/')` | Route prefix = audience |
| Blueprint name | `api_bp = Blueprint('api', ...)`, `admin_bp = Blueprint('admin', ...)` | Blueprint = audience grouping |
| DRF ViewSet base class | `ModelViewSet` vs `ReadOnlyModelViewSet` vs `APIView` | Different permission/auth patterns |
| Decorator auth | `@login_required` vs `@api_key_required` vs `@admin_required` | Auth type = audience |
| Django URL namespace | `namespace='api'`, `namespace='admin'` | Namespace = audience |
| File location | `views/api.py` vs `views/admin.py` vs `views/user.py` | Directory = audience |

**Disambiguation example**: When both `views/api.py` and `views/user.py` have user-related endpoints:
- Compare URL prefixes (`/api/v1/users/` vs `/users/`)
- Compare auth decorators (`@api_key_required` vs `@login_required`)
- Compare response format (JSON API envelope vs rendered template or DRF response)
- Document: "`api.py` exposes user data to third-party integrators via API key; `user.py` serves user pages/operations for the frontend SPA via session auth"

### 2. Settings / Configuration

#### Django
```bash
# Check settings structure
find . -name "settings.py" -o -name "settings" -type d
# Check for environment-split settings
ls -la */settings/ 2>/dev/null
```

Patterns:
- Single `settings.py` with env-based switches
- `settings/` package with `base.py`, `dev.py`, `prod.py`
- django-environ / python-decouple for env vars
- `DJANGO_SETTINGS_MODULE` environment variable

#### Flask
```bash
# Check config pattern
grep -rn "app.config\|from_object\|from_envvar" --include="*.py" | head -10
```

Patterns:
- Config class (DevelopmentConfig, ProductionConfig)
- Config dict
- Environment variables via `os.environ`

### 3. ORM / Database Patterns

#### Django ORM
```python
# Check model patterns
grep -rn "class.*models\.Model" --include="*.py" | head -20

# Check query patterns
grep -rn "\.objects\.\|\.filter(\|\.exclude(\|\.aggregate(" --include="*.py" | head -20
```

Check:
- [ ] Model field naming: `snake_case` (standard) or deviations
- [ ] Manager classes (custom managers)
- [ ] QuerySet chaining style
- [ ] Raw SQL usage: `connection.cursor()` or `.raw()`
- [ ] Migrations: Django migrations or manual SQL
- [ ] Soft delete: custom field or django-safedelete
- [ ] Abstract base models (audit fields: created_at, updated_at)

#### SQLAlchemy (Flask)
```python
# Check SQLAlchemy patterns
grep -rn "db\.Column\|db\.relationship\|db\.session" --include="*.py" | head -20
```

Check:
- [ ] Declarative vs classical mapping
- [ ] Session management pattern (scoped_session, session per request)
- [ ] Query pattern: model.query vs session.query(Model)
- [ ] Migration tool: Alembic or Flask-Migrate

### 4. View / Route Patterns

#### Django
```python
# Check view style
grep -rn "class.*View\|class.*ViewSet\|def.*request" --include="*.py" | head -20

# DRF detection
grep -rn "rest_framework\|APIView\|ModelViewSet\|serializers" --include="*.py" | head -10
```

View styles:
- Function-Based Views (FBV): `def user_list(request):`
- Class-Based Views (CBV): `class UserListView(ListView):`
- DRF ViewSets: `class UserViewSet(ModelViewSet):`
- DRF APIView: `class UserView(APIView):`

#### Flask
```python
# Check route patterns
grep -rn "@app.route\|@blueprint\|@bp\." --include="*.py" | head -20
```

Route styles:
- Direct `@app.route`
- Blueprint-based: `@bp.route`
- Flask-RESTful: `class UserResource(Resource):`
- MethodView: `class UserView(MethodView):`

### 5. Response Format

```bash
# Django/DRF
grep -rn "JsonResponse\|Response(\|return Response" --include="*.py" | head -20

# Flask
grep -rn "jsonify\|make_response\|return.*json" --include="*.py" | head -20
```

Common patterns:
```python
# Custom response wrapper
def success(data=None, msg='ok'):
    return JsonResponse({'code': 0, 'msg': msg, 'data': data})

# DRF Response
return Response({'code': 0, 'data': serializer.data})

# Flask jsonify
return jsonify(code=0, msg='success', data=data)
```

### 6. Authentication

```bash
# Django
grep -rn "login_required\|permission_required\|IsAuthenticated\|JWT\|token" --include="*.py" | head -20

# Check auth backend
grep -rn "AUTHENTICATION_BACKENDS\|REST_FRAMEWORK.*DEFAULT_AUTHENTICATION" --include="*.py" | head -10
```

Patterns:
- Session-based (Django default)
- Token-based (DRF TokenAuthentication)
- JWT (djangorestframework-simplejwt, PyJWT)
- Custom middleware

### 7. Error Handling

```bash
# Custom exception classes
grep -rn "class.*Exception\|class.*Error" --include="*.py" | head -20

# Exception middleware
grep -rn "exception_handler\|handle_exception\|errorhandler" --include="*.py" | head -10
```

Check:
- Global exception handler (DRF `EXCEPTION_HANDLER`, Flask `@app.errorhandler`)
- Custom exception hierarchy
- Error response format consistency

### 8. Import Conventions

```bash
# Check import style in a sample file
head -30 app/views.py  # or equivalent
```

Check:
- Absolute vs relative imports: `from app.models import User` vs `from .models import User`
- Import ordering: stdlib → third-party → local (isort configuration?)
- Star imports: `from models import *` (common in old code)
- Circular import workarounds (late imports inside functions)

### 9. Type Hints

```bash
# Check type hint usage
grep -rn "def.*->.*:" --include="*.py" | wc -l
grep -rn ": str\|: int\|: List\|: Dict\|: Optional" --include="*.py" | wc -l
```

If the project doesn't use type hints, AI should not add them.
If partially used, follow the pattern of which files have them.

### 10. Celery / Task Queue

### 11. Import Alias Consistency (Critical for Python)
Python import inconsistency is one of the most common causes of AI-generated code conflicts. Analyze thoroughly:
- Scan ALL import statements across the project to build an import alias map:
  - `import datetime` vs `from datetime import datetime` vs `from datetime import datetime as dt`
  - `import numpy as np` (standard alias) vs `import numpy`
  - `from collections import OrderedDict` vs `import collections`
- Check `from X import *` usage — if absent in the project, AI must not introduce it
- Check import ordering: `isort` configuration if present, or detect the convention:
  - stdlib → third-party → local (PEP 8 style)
  - Or custom ordering
- Check relative vs absolute imports within the project's own packages
- Generate MUST rules for every common import with its project-standard alias

```bash
# Check for async task patterns
grep -rn "celery\|@task\|@shared_task\|delay()\|apply_async" --include="*.py" | head -10
```

If Celery is present:
- Task definition pattern
- Broker configuration
- Result backend
- Retry policies

---

## Common Anti-Patterns in Legacy Python

Things AI will want to "fix" but should NOT:

1. **No type hints** — AI will try to add them everywhere. Don't.
2. **print() for logging** — Some old projects use `print()` instead of `logging`. If that's the pattern, follow it.
3. **Function-based views** — AI will try to convert to class-based. Don't.
4. **No docstrings** — AI will try to add them. Don't unless asked.
5. **String formatting with %** — AI will try to convert to f-strings. Only if the Python version supports f-strings AND the codebase uses them elsewhere.
6. **Mutable default arguments** — AI should fix this one — `def f(items=[])` ← this is always a bug.
7. **Single-file applications** — Some Flask apps are one giant `app.py`. Don't try to restructure.
8. **Manual SQL queries** — AI will try to convert to ORM. If the project mixes both, follow the file's style.
9. **No virtual environment** — project may have system-wide installs. Don't try to "fix" the deployment.
10. **requirements.txt without pinned versions** — AI should not pin versions unless the project does.

---

## File Role & Usage Scenario Patterns

When analyzing a Python project, classify every view/route and service file by its role and audience.

### Django: View File Disambiguation

**Common pattern — `views.py` vs `api.py` in the same app:**
```
app/
├── views.py          # Frontend: serves HTML templates or frontend SPA (session auth)
├── api.py            # Third-party/API: serves external integrators (API key / token auth)
├── admin_views.py    # Admin: custom admin views beyond Django admin
└── webhooks.py       # Webhook: receives callbacks from external services
```

**URL configuration reveals audience:**
```python
# urls.py — different prefixes for different audiences
urlpatterns = [
    path('users/', views.UserListView.as_view()),           # Frontend
    path('api/v1/users/', api.UserApiView.as_view()),       # Third-party API
    path('admin/users/', admin_views.UserAdminView.as_view()),  # Admin
    path('webhook/payment/', webhooks.payment_callback),    # Webhook
    path('internal/sync/', internal.sync_users),            # Internal service
]
```

**DRF ViewSet vs standard views:**
| File / Class | Audience | Clues |
|-------------|----------|-------|
| `views.py` → `UserListView(ListView)` | Frontend | Inherits Django generic views, renders templates |
| `views.py` → `UserView(TemplateView)` | Frontend | Returns `render()` or `TemplateResponse` |
| `api.py` → `UserViewSet(ModelViewSet)` | API consumers | DRF ViewSet, returns `Response()`, registered in DRF router |
| `api.py` → `UserApiView(APIView)` | API consumers | DRF APIView, token/key auth |
| `webhooks.py` → `payment_callback(request)` | Webhook | `@csrf_exempt`, signature verification |
| `tasks.py` → `sync_orders()` | Scheduled task | `@shared_task` (Celery), `@periodic_task` |

**URL file splitting:**
```
app/
├── urls.py             # Main URL config (may include all below)
├── api_urls.py         # API-specific URLs (/api/v1/...)
├── admin_urls.py       # Admin-specific URLs (/admin/...)
└── webhook_urls.py     # Webhook URLs (/webhook/...)
```

### Flask: Blueprint-Based Audience Separation

**Blueprint prefix and middleware determine audience:**
```python
# Separate blueprints for different audiences
api_bp = Blueprint('api', __name__, url_prefix='/api/v1')    # Third-party API
admin_bp = Blueprint('admin', __name__, url_prefix='/admin')  # Admin panel
web_bp = Blueprint('web', __name__, url_prefix='/')           # Frontend

# Different auth decorators per blueprint
@api_bp.before_request
def require_api_key(): ...    # API key auth

@admin_bp.before_request
def require_admin_login(): ... # Session + admin role check
```

**File naming per blueprint:**
```
app/
├── api/
│   ├── __init__.py       # API blueprint registration
│   ├── users.py          # User API endpoints (Third-party)
│   └── orders.py         # Order API endpoints (Third-party)
├── admin/
│   ├── __init__.py       # Admin blueprint registration
│   └── users.py          # User admin views (Admin)
├── views/
│   ├── __init__.py       # Web blueprint registration
│   └── users.py          # User frontend views (Frontend)
└── tasks/
    └── sync.py           # Celery tasks (Scheduled-task)
```

### Service Layer Disambiguation

| File | Role | Clues |
|------|------|-------|
| `services/user_service.py` | API-layer | Called by view/API controllers |
| `services/payment_service.py` | Integration | Calls external payment API |
| `services/notification_service.py` | Internal-utility | Sends emails/SMS, called by multiple services |
| `tasks.py` / `tasks/xxx.py` | Scheduled-task | Celery `@shared_task` decorator |
| `management/commands/xxx.py` | Scheduled-task | Django management commands |
| `signals.py` / `receivers.py` | Event-driven | Django signals (`@receiver`) |

### Detection Commands
```bash
# List all view/API files
find . -name "views.py" -o -name "api.py" -o -name "*_views.py" -o -name "*_api.py" -o -name "webhooks.py" | sort

# Check URL patterns per file
grep -rn "path(\|url(\|re_path(" --include="*.py" */urls*.py | head -30

# Find view base classes to determine type
grep -rn "class.*View\|class.*ViewSet\|class.*APIView\|class.*Resource" --include="*.py" | head -30

# Find Celery tasks and management commands
grep -rn "@shared_task\|@periodic_task\|BaseCommand" --include="*.py" | head -20

# Find auth decorators to determine audience
grep -rn "@login_required\|@permission_required\|@api_view\|@csrf_exempt\|@authentication_classes" --include="*.py" | head -30
```

---

## Known Version Boundaries (Reference)

> The following are feature boundary references for known versions. If the project uses a version not listed here, analyze based on actually detected version features and note version constraints in the generated rules.

### Python 3.6
- f-strings available (basic form)
- No walrus operator (:=)
- No `dict` ordering guaranteed (CPython does, but not spec)
- `asyncio.run()` not available (use `loop.run_until_complete()`)
- No `dataclasses`

### Python 3.7
- `dataclasses` module
- `dict` ordering guaranteed by spec
- `asyncio.run()` available
- No walrus operator
- No union type syntax `X | Y`

### Python 3.8
- Walrus operator `:=`
- Positional-only parameters `/`
- f-string `=` debugging: `f"{x=}"`

### Python 3.9
- `dict | dict` merge operator
- `list[int]` instead of `List[int]` for type hints
- `str.removeprefix()`, `str.removesuffix()`

Check the project's Python version and ensure AI doesn't use features above that version.

### Other Versions
For Python 3.10+, Django 4.x+, Flask 3.x+, or other versions not listed above — detect the actual language features and framework API from the project's code. Record differences from the known versions above and flag as `LOW_CONFIDENCE` for user confirmation.

---

## Key Dependencies to Track

| Dependency | Why It Matters |
|-----------|---------------|
| `Django` | Version determines ORM features, URL patterns, middleware |
| `djangorestframework` | DRF version — serializer API changes between versions |
| `Flask` | Version determines app factory pattern, context handling |
| `SQLAlchemy` | Version 1.x vs 2.x have very different APIs |
| `celery` | Task queue — version determines configuration format |
| `redis` / `django-redis` | Cache/broker — connection pattern |
| `gunicorn` / `uwsgi` | WSGI server — affects deployment config |
| `pytest` / `unittest` | Test framework — determines test structure |
| `requests` | HTTP client — timeout/retry patterns |
| `Pillow` | Image processing |
| `pandas` / `numpy` | If data processing is part of the web app |
| `alipay-sdk` / `wechatpay` | Payment integration — Chinese ecosystem |
