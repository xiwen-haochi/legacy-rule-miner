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

#### Django
```
project/
├── manage.py
├── project/              # Project settings package
│   ├── __init__.py
│   ├── settings.py       # or settings/ directory
│   ├── urls.py
│   └── wsgi.py
├── app1/                 # Django app
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── serializers.py    # DRF
│   ├── urls.py
│   ├── views.py
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
│   ├── views/ or routes/
│   ├── services/
│   ├── utils/
│   └── templates/
├── config.py
├── requirements.txt
└── run.py / wsgi.py
```

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
