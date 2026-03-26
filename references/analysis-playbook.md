# Analysis Playbook

Detailed checklists for each analysis layer. Read this during Step 3 of the workflow.

## Table of Contents
1. [L0 — Project Metadata](#l0--project-metadata)
2. [L1 — Project Structure](#l1--project-structure)
3. [L2 — Configuration Layer](#l2--configuration-layer)
4. [L3 — API / Route Layer](#l3--api--route-layer)
5. [L4 — Business Logic Layer](#l4--business-logic-layer)
6. [L5 — Data Access Layer](#l5--data-access-layer)
7. [L6 — Cross-Cutting Concerns](#l6--cross-cutting-concerns)
8. [L7 — Infrastructure](#l7--infrastructure)
9. [L8 — Hidden Patterns (Interview)](#l8--hidden-patterns-interview)
10. [Sampling Strategy](#sampling-strategy)
11. [Hidden-Point Mining Techniques](#hidden-point-mining-techniques)

---

## L0 — Project Metadata

### What to read
- Build files: `pom.xml`, `build.gradle`, `composer.json`, `package.json`, `requirements.txt`, `go.mod`, `*.csproj`
- Lock files: `package-lock.json`, `yarn.lock`, `composer.lock`, `Pipfile.lock`, `go.sum`
- Editor configs: `.editorconfig`, `.prettierrc`, `.eslintrc`, `tslint.json`, `checkstyle.xml`
- Ignore files: `.gitignore`, `.dockerignore`
- CI configs: `Jenkinsfile`, `.gitlab-ci.yml`, `.github/workflows/*.yml`, `.travis.yml`

### What to extract
- [ ] Language and runtime version (e.g., Java 8, Node 14, Python 3.7)
- [ ] Framework and framework version (e.g., Spring Boot 2.1.6, Django 2.2)
- [ ] Key dependencies with exact versions (top 15–20)
- [ ] Build command(s) and test command(s)
- [ ] Code style tools in use (ESLint, Prettier, Checkstyle, Black, gofmt)
- [ ] Environment variable patterns (from CI config, Dockerfile, .env.example)
- [ ] Deployment target (Docker, K8s, bare metal, cloud service)

### Rules to generate
→ `project-overview.md`: tech stack summary, versions, build commands
→ `dependencies.md`: dependency inventory with version constraints

---

## L1 — Project Structure

### What to do
1. Generate directory tree at depth 3–4: `find . -maxdepth 4 -type d | head -80`
2. Count files per directory: `find . -type f | sed 's|/[^/]*$||' | sort | uniq -c | sort -rn | head -30`

### What to extract
- [ ] Layering pattern: MVC / DDD / flat / hexagonal / other
- [ ] Module structure: mono-module, multi-module (Maven modules, Gradle subprojects, Go packages, monorepo workspaces)
- [ ] Source root: `src/main/java`, `app/`, `src/`, etc.
- [ ] Test root and test naming convention (e.g., `*Test.java`, `*.spec.ts`, `test_*.py`)
- [ ] Resource locations: templates, static files, SQL migrations, i18n files
- [ ] Configuration file locations
- [ ] "God directories" — directories with 50+ files (signals poor organization, important for rules)

### Rules to generate
→ `architecture.md`: layer pattern, module map, where to put new code
→ `naming-conventions.md`: file/directory naming patterns

---

## L2 — Configuration Layer

### What to read
- Application configs: `application.yml`, `application.properties`, `config.py`, `settings.py`, `.env`, `config/*.js`
- Multi-environment configs: `application-dev.yml`, `application-prod.yml`, `.env.development`, `.env.production`
- Framework-specific: `web.xml`, `config/routes.rb`, `config/app.php`

### What to extract
- [ ] Configuration loading mechanism (env vars, config files, config center)
- [ ] Multi-environment strategy (profile-based, file-based, env-var-based)
- [ ] Database connection configuration pattern
- [ ] Cache configuration (Redis, Memcached, in-memory)
- [ ] Message queue configuration (RabbitMQ, Kafka, Redis)
- [ ] Third-party service configurations (payment, SMS, OSS, email)
- [ ] Secrets management approach (plain text in config vs. env vars vs. vault)
- [ ] Custom configuration properties and their naming patterns

### Rules to generate
→ `project-overview.md`: environment variable patterns
→ `coding-patterns.md`: how to read/reference configuration values
→ `security.md`: secrets management approach

---

## L3 — API / Route Layer

### What to read
Sample 2–3 controller/handler files per module. Also check:
- Route definition files (if centralized)
- Request/Response DTO or Form classes
- Middleware / interceptor / filter definitions
- API documentation configs (Swagger, Knife4j)

### What to extract
- [ ] URL pattern style: `/api/v1/users`, `/user/list`, `api.php?m=user&a=list`
- [ ] HTTP method usage: REST-style or all-POST
- [ ] Request parameter binding: query params, body JSON, form data
- [ ] Response envelope: `{code: 0, msg: "ok", data: ...}` or raw
- [ ] Error response format: error codes, error message standardization
- [ ] Pagination pattern: `page/size`, `offset/limit`, cursor-based
- [ ] Authentication middleware: where applied, how configured
- [ ] Request validation: annotation-based, manual, framework validator
- [ ] API versioning strategy

### Pattern extraction
For each controller sampled:
1. Note the class-level annotations/decorators
2. Note method-level annotations/decorators
3. Capture the parameter binding style
4. Capture the return type wrapping pattern
5. Check if there's consistent logging at entry/exit

### Rules to generate
→ `api-design.md`: URL patterns, response format, error codes, pagination
→ `coding-patterns.md`: request validation approach

---

## L4 — Business Logic Layer

### What to read
Sample 2–3 service/business classes per module. Look for:
- Service class structure and organization
- Transaction boundaries
- Inter-service dependencies
- Business validation logic
- Event/message publishing patterns

### What to extract
- [ ] Service class naming: `*Service`, `*Manager`, `*Handler`, `*Facade`
- [ ] Method naming patterns: `getXxx`, `findXxx`, `queryXxx`, `createXxx`, `doXxx`
- [ ] Dependency injection style: constructor, field, setter
- [ ] Transaction management: annotation-based, programmatic, or absent
- [ ] Transaction propagation patterns (if annotation-based, what attributes are used)
- [ ] Business validation approach: guard clauses, validator classes, inline checks
- [ ] Return value patterns: direct return, wrapped Result/Response, Optional
- [ ] Exception throwing patterns: custom exceptions, error codes, bare RuntimeException
- [ ] Logging patterns: what's logged at entry/exit, log level conventions
- [ ] Inter-service communication: direct method calls, events, message queue
- [ ] Concurrency control: locks, optimistic locking, distributed locks

#### Import / Alias Consistency
- [ ] Scan how the same module/package is imported across different files
  - e.g., Python: `from datetime import datetime` vs `import datetime as dt` — which is the project convention?
  - e.g., Node.js: `const moment = require('moment')` vs `const m = require('moment')`
  - e.g., Java: wildcard imports `import java.util.*` vs explicit `import java.util.List`
- [ ] Check import ordering convention (stdlib / third-party / local grouping)
- [ ] Identify the dominant pattern — generate MUST rules for it
- [ ] Flag inconsistencies — generate DO/DON'T examples from actual project code

#### Constant & Configuration Duplication
- [ ] Search for constants defined in multiple locations (same name, same value)
  ```bash
  grep -rn "const \|final static\|define(\|UPPER_CASE_VAR =" --include="*.java" --include="*.py" --include="*.js" --include="*.php" --include="*.go" --include="*.cs" | sort
  ```
- [ ] Identify the canonical source for each constant category (error codes, config values, magic numbers)
- [ ] Generate rules: "Error codes MUST come from [file], not redefined locally"
- [ ] Check for duplicated configuration values between config files and code

### Pattern extraction
For each service sampled:
1. Count methods — if > 20, note it (class may be too large, but this is the current convention)
2. Identify the longest method — note its length (calibrates expectations)
3. Look for repeated boilerplate (try-catch wrapping, logging, null checks)
4. Check if business rules are encoded as complex if-else chains

### Rules to generate
→ `coding-patterns.md`: service patterns, error handling, logging, validation, import conventions, constant management
→ `architecture.md`: inter-service communication style
→ `pitfalls.md`: any unusual business logic patterns

---

## L5 — Data Access Layer

### What to read
Sample 2–3 DAO/Repository/Mapper files per module. Also check:
- Entity/Model class definitions
- Database migration files (if any)
- SQL mapping files (MyBatis XML, etc.)
- Database schema or DDL scripts

### What to extract
- [ ] ORM framework: MyBatis, Hibernate/JPA, Eloquent, SQLAlchemy, Sequelize, GORM, EF Core
- [ ] Query style: annotation queries, XML mapping, query builder, raw SQL, Criteria API
- [ ] Entity naming vs. table naming mapping
- [ ] Column naming convention (camelCase, snake_case)
- [ ] Primary key strategy: auto-increment, UUID, snowflake, custom
- [ ] Soft delete pattern: `is_deleted`, `deleted_at`, `status` field
- [ ] Audit fields: `created_at`, `updated_at`, `created_by`, `updated_by`
- [ ] Association handling: eager/lazy loading patterns
- [ ] Pagination implementation at DAO level
- [ ] Custom SQL patterns: complex joins, subqueries, stored procedure calls
- [ ] Connection pool configuration

#### Cross-Table Field Dependencies
- [ ] Analyze foreign key relationships in schema/migration files
- [ ] Search for business code that updates multiple tables together for the same entity change
  ```bash
  # Look for multiple UPDATE/save/update calls in the same method
  grep -rn "UPDATE \|\.save(\|\.update(" --include="*.java" --include="*.py" --include="*.js" --include="*.php" --include="*.go" --include="*.cs"
  ```
- [ ] Identify field pairs that must be updated together (e.g., changing `user.name` requires updating `order.buyer_name`)
- [ ] Check for denormalized fields that mirror data from other tables
- [ ] Map trigger-based or listener-based cascading updates

#### Database Field Change Strategy
- [ ] Check if the project follows "add > modify" for schema changes:
  - New columns added with defaults rather than altering existing column types/names
  - Old columns deprecated via naming convention (e.g., `old_field_deprecated`) or commented
  - Migration files show pattern of ADD COLUMN vs ALTER COLUMN
- [ ] Document the project's actual migration approach (manual SQL, framework migration tool, DBA-managed)
- [ ] Check for column rename history — if columns were ever renamed, how was it handled?

### Hidden patterns to detect
- SQL injection risks (string concatenation in queries)
- N+1 query patterns
- Missing indexes (if schema available)
- Inconsistent null handling

### Rules to generate
→ `data-access.md`: ORM patterns, query style, transaction rules, entity conventions, field change strategy, cross-table dependencies
→ `naming-conventions.md`: table/column naming standards

---

## L6 — Cross-Cutting Concerns

### What to read
- Global exception handlers / error middleware
- Logging configuration and usage patterns
- Security configuration (Spring Security, auth middleware, JWT handling)
- Cache usage patterns
- Utility/Helper classes
- Constants/Enum definitions
- AOP aspects (if Java/Spring)

### What to extract

#### Error Handling
- [ ] Global exception handler location and pattern
- [ ] Custom exception hierarchy
- [ ] Error code system (numeric codes, string codes, enum)
- [ ] How exceptions are logged vs. returned to client
- [ ] Stack trace exposure policy (dev vs. prod)

#### Logging
- [ ] Logging framework (Log4j, SLF4J, Logback, Winston, Python logging)
- [ ] Log level usage conventions (when DEBUG, when INFO, when ERROR)
- [ ] Structured logging or free-form
- [ ] Sensitive data masking in logs
- [ ] Request/Response logging middleware

#### Security
- [ ] Authentication mechanism (Session, JWT, OAuth, Basic Auth)
- [ ] Authorization model (RBAC, ABAC, simple role checks)
- [ ] Password handling (hashing algorithm, salt strategy)
- [ ] CSRF protection
- [ ] CORS configuration
- [ ] Input sanitization / XSS prevention
- [ ] SQL injection prevention (parameterized queries)

#### Caching
- [ ] Cache framework/tool (Redis, Memcached, in-memory)
- [ ] Cache key naming pattern
- [ ] TTL conventions
- [ ] Cache invalidation strategy

### Rules to generate
→ `coding-patterns.md`: error handling, logging patterns
→ `security.md`: auth, validation, data protection
→ `pitfalls.md`: security-related notes

---

## L7 — Infrastructure

### What to read
- `Dockerfile`, `docker-compose.yml`
- Kubernetes manifests (`*.yaml` in k8s/, deploy/, helm/)
- CI/CD pipeline configs
- Health check endpoints
- Monitoring/alerting configs
- Deployment scripts

### What to extract
- [ ] Container base image and build process
- [ ] Environment variable injection method
- [ ] Health check / readiness probe pattern
- [ ] Deployment strategy (rolling, blue-green, canary)
- [ ] Port conventions
- [ ] Volume mounts and data persistence
- [ ] Service discovery pattern (if microservices)

### Rules to generate
→ `project-overview.md`: build and deployment commands
→ `pitfalls.md`: deployment-related constraints

---

## L8 — Hidden Patterns (Interview)

Things code analysis cannot detect. Must be gathered through user interview.

See `references/interview-questions.md` for the full question bank.

Key areas:
- Oral conventions not in code (e.g., "we never touch the billing module")
- Known production incidents and their workarounds
- Business domain constraints (e.g., "orders older than 30 days cannot be modified")
- Team-specific practices (e.g., "we always pair-review changes to the auth module")
- External system constraints (e.g., "the payment gateway has a 5-second timeout")
- Data migration history (e.g., "some user records have legacy format phone numbers")

### Rules to generate
→ `pitfalls.md`: all interview-discovered constraints
→ Module-specific rule files `.rules-{module}.md` if applicable

---

## Sampling Strategy

### File Selection Criteria

**Prefer:**
- Files modified in the last 6 months (reflect current conventions)
- Files with 100–500 lines (representative complexity)
- Files in core business modules (not utility/infrastructure)
- Files with good test coverage (likely well-maintained)

**Avoid:**
- Generated files (protobuf, Swagger codegen, etc.)
- Vendor/node_modules directories
- Test fixtures and mock data
- Database migration files (read for schema info, but don't model code style from them)

### Project Size Calibration

| Size | Total Files | Samples per Layer per Module | Total Reads |
|------|------------|------------------------------|-------------|
| Small | < 100 | 3–5 | ~25 |
| Medium | 100–500 | 2–3 | ~30–40 |
| Large | 500–2000 | 2 (core modules only) | ~25–35 |
| Very Large | 2000+ | 2 (top 3–4 modules) | ~25 |

For very large projects, focus on the modules the user cares about most.

### Parallel Sampling with Subagents

For large projects, use subagents to parallelize:
- Subagent 1: L2 (config) + L7 (infra)
- Subagent 2: L3 (API) + L6 (cross-cutting)
- Subagent 3: L4 (business) + L5 (data)

---

## Hidden-Point Mining Techniques

These techniques specifically target the "things you'd miss" that make AI-generated code fail.

### 1. Comment Archaeology
```bash
# Search for warning markers
grep -rn "TODO\|FIXME\|HACK\|WORKAROUND\|XXX\|WARN\|CAUTION\|@deprecated" --include="*.java" --include="*.py" --include="*.php" --include="*.js" --include="*.ts" --include="*.go" --include="*.cs"
```
Each hit is a potential rule. Categorize by:
- **Active workaround** → Must be preserved in `pitfalls.md`
- **Technical debt** → Note but don't try to fix
- **Deprecated code** → Mark as do-not-use in rules

### 2. Custom Exception Analysis
Map the exception class hierarchy. Each custom exception type reveals a business constraint:
- `InsufficientBalanceException` → Balance validation is a domain rule
- `OrderLockedException` → Orders have lock states
- `DuplicateSubmissionException` → Idempotency is required

### 3. Database Constraint Mining
Read schema/migration files for:
- `UNIQUE` constraints → Business uniqueness rules
- `CHECK` constraints → Value-range rules
- `FOREIGN KEY` with `CASCADE`/`RESTRICT` → Deletion dependencies
- Triggers and stored procedures → Hidden business logic

### 4. Integration Point Mapping
Search for HTTP client / SDK usage to map third-party integrations:
```bash
grep -rn "HttpClient\|RestTemplate\|WebClient\|OkHttp\|requests\.\|axios\.\|fetch(" --include="*.java" --include="*.py" --include="*.js" --include="*.ts"
```
Each integration point needs:
- Timeout configuration
- Retry policy
- Error handling pattern
- Circuit breaker usage

### 5. Transaction Boundary Detection
```bash
# Java
grep -rn "@Transactional" --include="*.java"
# PHP
grep -rn "DB::transaction\|beginTransaction" --include="*.php"
# Python
grep -rn "@transaction\.atomic\|with transaction" --include="*.py"
```
Map which methods are transactional and which are not. Inconsistencies are often intentional (long-running operations, external calls within transactions).

### 6. Scheduled Task Inventory
```bash
grep -rn "@Scheduled\|cron\|schedule\|setInterval\|crontab" --include="*.java" --include="*.py" --include="*.js" --include="*.php"
```
Document all scheduled tasks — they have data dependencies with the main application that AI must not break.

### 7. Hardcoded Value Detection
```bash
# Magic numbers and hardcoded strings
grep -rn "http://\|https://\|[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}" --include="*.java" --include="*.py" --include="*.js" --include="*.php" --include="*.go"
```
Hardcoded values are often there for a reason in legacy code. Document them rather than "fixing" them.

### 8. Concurrency Pattern Detection
```bash
# Java
grep -rn "synchronized\|ReentrantLock\|AtomicInteger\|ConcurrentHashMap\|volatile" --include="*.java"
# Go
grep -rn "sync\.Mutex\|sync\.RWMutex\|sync\.WaitGroup\|chan " --include="*.go"
```
Concurrency code is the #1 place where "improvements" cause production issues.

### 9. Environment-Specific Divergence
Compare dev and prod configurations:
- Different database drivers
- Different cache backends
- Feature flags
- Rate limiting differences

These divergences are often intentional and encode implicit rules.

### 10. Dead-Code Boundary Detection
Look for commented-out code blocks, especially near active code. These often indicate:
- Features that were rolled back but may come back
- Alternative implementations kept as reference
- Code the team is afraid to delete

Document their existence in `pitfalls.md` rather than suggesting deletion.

### 11. Cross-Table Field Dependency Detection
Search for patterns where modifying one table requires updating another:
```bash
# Look for methods that update multiple tables/models
grep -rn "UPDATE.*SET\|save()\|update()\|persist()" --include="*.java" --include="*.py" --include="*.php" --include="*.js" --include="*.go" --include="*.cs" -l | xargs -I {} grep -c "UPDATE\|save\|update" {} | sort -t: -k2 -rn
```
Also check:
- Denormalized fields (same data in multiple tables, e.g., `order.buyer_name` mirrors `user.name`)
- Cascade configurations in ORM (onDelete, onUpdate)
- Application-level cascading in service code (one method calling multiple DAO updates)
- Event listeners / observers that propagate field changes

Each dependency found → rule in `data-access.md`: "When modifying [field] in [table A], MUST also update [field] in [table B]"
