# Java / Spring Boot — Language-Specific Analysis

Read this file when the project is detected as Java (Maven/Gradle with Spring Boot or SSM).

## Stack Detection

### Version Detection
```
# Spring Boot version
grep -r "spring-boot-starter-parent" pom.xml | grep "version"
grep -r "org.springframework.boot" build.gradle

# Java version
grep -r "<java.version>" pom.xml
grep -r "sourceCompatibility" build.gradle

# Spring Cloud (if present)
grep -r "spring-cloud" pom.xml
```

### Common Stack Variants

| Variant | Key Indicators | Era |
|---------|---------------|-----|
| SSM (Spring MVC + MyBatis) | `spring-webmvc` + `mybatis-spring`, no `spring-boot` | 2012–2016 |
| Spring Boot 1.x | `spring-boot 1.*`, Java 7/8 | 2015–2017 |
| Spring Boot 2.0–2.3 | `spring-boot 2.0–2.3`, Java 8 | 2018–2020 |
| Spring Boot 2.4–2.7 | `spring-boot 2.4–2.7`, Java 8/11 | 2020–2022 |
| Spring Cloud Dalston/Edgware | `spring-cloud-dependencies Dalston/Edgware` | 2017–2018 |
| Spring Cloud Finchley/Greenwich | `spring-cloud-dependencies Finchley/Greenwich` | 2018–2020 |

---

## Java-Specific Analysis Points

### 1. Project Structure & File Role Inventory

#### Files to Exclude from Code Style Analysis
Skip these when sampling conventions (still read migration/schema files for data model info):
- `target/`, `build/`, `out/` — compiled output
- `*.class`, `*.jar`, `*.war` — binary artifacts
- `generated-sources/`, `generated-test-sources/` — code generators
- `**/src/main/resources/db/migration/**` — Flyway/Liquibase migrations (read for schema only)

#### Common Java/Spring Project Layouts

**Single-module Spring Boot:**
```
src/main/java/com/example/project/
├── controller/
│   ├── UserController.java        # Frontend: user CRUD for web/mobile
│   ├── api/
│   │   └── UserApiController.java # Third-party: user data API for external integrators
│   └── admin/
│       └── UserAdminController.java # Admin: user management for admin panel
├── service/
│   ├── UserService.java           # API-layer: user business logic
│   ├── impl/
│   │   └── UserServiceImpl.java   # API-layer: user service implementation
│   ├── PaymentGatewayService.java # Integration: wraps third-party payment API
│   └── NotificationService.java   # Internal-utility: shared notification sender
├── mapper/ or dao/ or repository/
│   ├── UserMapper.java            # Data access: user table CRUD
│   └── OrderMapper.java           # Data access: order table CRUD
├── entity/ or model/ or domain/
├── dto/ or vo/
├── config/
├── interceptor/ or filter/
├── util/ or common/
└── task/ or job/ or schedule/
    └── OrderSyncTask.java         # Scheduled-task: sync orders via cron
```

#### File Role Detection for Java/Spring

For EVERY code file in each package, determine its role using these clues:

| Clue | How to Detect | What It Means |
|------|--------------|---------------|
| URL prefix | `@RequestMapping("/api/v1/")` vs `@RequestMapping("/admin/")` vs `@RequestMapping("/open/")` | Identifies audience: frontend / admin / third-party |
| Auth annotation | `@PreAuthorize`, custom `@RequiresRole`, `@ApiKeyAuth` | Distinguishes auth model per controller |
| Response wrapper | `Result<T>` vs `ApiResponse<T>` vs `ResponseEntity<T>` | Different audiences may use different envelopes |
| Base class | `extends BaseController` vs `extends BaseApiController` vs `extends BaseAdminController` | Inherited behavior differs by audience |
| Package location | `controller.api.*` vs `controller.admin.*` vs `controller.web.*` | Explicit audience separation |
| Swagger annotations | `@Api(tags = "Open API")` vs `@Api(tags = "Admin")` | Documents intended audience |

**Disambiguation example**: When both `UserController` and `UserApiController` exist:
- Compare `@RequestMapping` prefixes (e.g., `/user/` vs `/api/v1/user/`)
- Compare auth middleware (session vs API key)
- Compare response format (same `Result<T>` or different envelope)
- Document: "`UserController` serves frontend SPA via session auth; `UserApiController` exposes user data to third-party integrators via API key"

### 2. Spring Annotation Patterns

Check which injection style the project uses:

```java
// Field injection (common in legacy)
@Autowired
private UserService userService;

// Constructor injection (modern)
private final UserService userService;
public UserController(UserService userService) { ... }

// Setter injection (rare)
@Autowired
public void setUserService(UserService userService) { ... }
```

**Rule generation**: Whichever style dominates, that's the rule. Don't let AI "improve" field injection to constructor injection.

### 3. Configuration Style

Check what the project uses:
- `application.properties` vs `application.yml`
- `@Value("${...}")` vs `@ConfigurationProperties`
- XML bean definitions vs Java `@Configuration` classes
- Component scanning scope

**Key file patterns to check:**
```bash
find . -name "*.xml" -path "*/resources/*" | head -20
find . -name "*Config*.java" -o -name "*Configuration*.java" | head -20
```

### 4. MyBatis Patterns (if applicable)

MyBatis is the dominant ORM in Chinese Java projects. Check:

```bash
# XML mappers vs annotation-based
find . -name "*Mapper.xml" | wc -l
grep -rn "@Select\|@Insert\|@Update\|@Delete" --include="*.java" | wc -l
```

| Pattern | What to Look For |
|---------|-----------------|
| XML mapper location | `resources/mapper/**/*.xml` or `resources/mybatis/**/*.xml` |
| ResultMap vs resultType | Which is preferred for complex queries |
| Dynamic SQL | `<if>`, `<choose>`, `<foreach>` usage patterns |
| PageHelper | `PageHelper.startPage()` before queries |
| TypeHandler | Custom type handlers for enum, JSON, etc. |
| Interceptor/Plugin | Audit fields, tenant isolation, pagination |

**Critical**: If the project uses MyBatis XML mappers, AI must NOT generate JPA-style code. Check for JPA annotations that might be inactive (leftover from migration attempts).

### 5. Service Layer Patterns

```bash
# Transaction annotation usage
grep -rn "@Transactional" --include="*.java" | head -20

# Service method patterns
grep -rn "public.*Service" --include="*.java" -l | head -10
```

Check:
- Transaction propagation: `REQUIRED` (default), `REQUIRES_NEW`, `NOT_SUPPORTED`
- Rollback rules: `rollbackFor = Exception.class` vs default (only RuntimeException)
- Transaction on interface vs implementation class
- Programmatic transaction with `TransactionTemplate`

### 6. Exception Handling

```bash
# Custom exceptions
find . -name "*Exception.java" | head -20

# Global exception handler
grep -rn "@ControllerAdvice\|@RestControllerAdvice\|@ExceptionHandler" --include="*.java"
```

Map the exception hierarchy:
- Base exception class
- Business vs. system exceptions
- Error code mechanism (enum, constants, or inline strings)

### 7. DTO/VO/Entity Conventions

```bash
# Find entity-related classes
find . -name "*DTO.java" -o -name "*VO.java" -o -name "*Entity.java" -o -name "*PO.java" -o -name "*DO.java" | head -30
```

| Convention | Variants |
|-----------|----------|
| Database entity | Entity, PO, DO, Model |
| Data transfer | DTO, Param, Request, Form |
| View output | VO, Response, Result |
| Between layers | BO (Business Object) |

Check: Does the project use BeanUtils.copyProperties, MapStruct, manual conversion, or Lombok's @Builder?

### 8. Lombok Usage

```bash
grep -rn "@Data\|@Getter\|@Setter\|@Builder\|@Slf4j\|@AllArgsConstructor" --include="*.java" | wc -l
```

If Lombok is used, note which annotations. If NOT used, AI must not introduce it.

### 9. Utility Classes

```bash
find . -name "*Utils.java" -o -name "*Util.java" -o -name "*Helper.java" | head -20
```

Which utility libraries:
- StringUtils: Apache Commons Lang (lang vs lang3!) vs Spring's StringUtils vs Guava
- CollectionUtils: Apache Commons vs Spring vs Guava
- JSON: FastJSON (1.x vs 2.x) vs Jackson vs Gson
- Date: java.util.Date vs Joda-Time vs java.time

**Critical**: The JSON library choice is a hard constraint. If the project uses FastJSON, all JSON operations must use FastJSON.

### 10. Response Wrapper

```bash
# Find Result/Response wrapper class
find . -name "Result.java" -o -name "Response.java" -o -name "ApiResult.java" -o -name "R.java" -o -name "CommonResult.java" | head -10
```

Read the wrapper class to understand:
- Field names: `code`/`status`, `msg`/`message`, `data`
- Success code: `0`, `200`, `"0000"`, `"SUCCESS"`
- Static factory methods: `success()`, `fail()`, `ok()`, `error()`

### 11. Multi-Module Project Structure

### 12. Import Consistency
- Check wildcard import usage: `import java.util.*` vs explicit imports `import java.util.List`
- Identify the project's import ordering convention (java.* → javax.* → third-party → project packages)
- Check for IDE-generated import order vs manual conventions
- Static import patterns: does the project use `import static` and for what?
- Check if the project uses full paths or short forms for common classes (e.g., `org.apache.commons.lang3.StringUtils` vs `StringUtils`)

For Maven multi-module:
```bash
# Find module definitions
grep -A 10 "<modules>" pom.xml
```

Common module structures:
- `xxx-api` (interfaces, DTOs)
- `xxx-service` (implementation)
- `xxx-web` / `xxx-controller` (controllers)
- `xxx-common` / `xxx-core` (shared utilities)
- `xxx-dao` / `xxx-mapper` (data access)

---

## Common Anti-Patterns in Legacy Java

These are things AI will want to "fix" but should NOT:

1. **Field injection everywhere** — AI will try to convert to constructor injection
2. **Giant service classes** (1000+ lines) — AI will try to extract sub-services
3. **Business logic in controllers** — AI will try to move to service layer
4. **Manual transaction management** — AI will try to add @Transactional
5. **String concatenation for SQL** (in very old code) — AI should NOT just add parameterized queries without understanding the context
6. **Checked exceptions** — AI will try to convert to runtime exceptions
7. **Raw types** (no generics) — AI will try to add generic parameters
8. **Date/Calendar usage** — AI will try to convert to java.time

For each of these: document the current pattern and generate a SHOULD rule to continue it, or a PREFER rule to allow gradual improvement.

---

## File Role & Usage Scenario Patterns

When analyzing a Java/Spring project, classify every controller, service, and component file by its role and audience.

### Controller Layer Disambiguation

**By package/directory:**
```
com.example.project.controller/          # Frontend-facing (default)
com.example.project.controller.api/       # Third-party/Open API
com.example.project.controller.admin/     # Admin panel
com.example.project.controller.internal/  # Internal microservice
com.example.project.controller.open/      # Open API for external integrators
```

**By class naming:**
| Pattern | Typical Audience | Clues |
|---------|-----------------|-------|
| `UserController` | Frontend | `@RequestMapping("/user")`, session/cookie auth |
| `UserApiController` | Third-party | `@RequestMapping("/api/user")`, API key / OAuth |
| `UserAdminController` | Admin | `@RequestMapping("/admin/user")`, admin role check |
| `UserOpenController` | Open API | `@RequestMapping("/open/user")`, signature verification |
| `UserInternalController` | Internal service | `@RequestMapping("/internal/user")`, no auth or service token |

**By annotation/middleware:**
- `@RequiresPermissions("admin:user:list")` → Admin
- `@ApiKey` or custom `@OpenApi` → Third-party
- `@Inner` or `@InternalAccess` → Internal microservice
- `@Login` or `@RequiresAuthentication` → Frontend

### Service Layer Disambiguation

| Pattern | Role | Clues |
|---------|------|-------|
| `UserService` | API-layer service | Called by controllers, standard CRUD |
| `UserFacade` | External facade | Aggregates multiple services for a single API response |
| `UserRpcService` / `UserFeignClient` | Microservice client | `@FeignClient`, `@DubboReference`, calls remote services |
| `UserJobService` / `UserTaskService` | Scheduled task | Called by `@Scheduled` / `@XxlJob` methods |
| `UserExportService` | Utility service | Excel/CSV export, called by admin controllers |
| `SmsService` / `OssService` | Integration service | Wraps third-party SDK calls |
| `UserEventListener` / `UserMqConsumer` | Event-driven | `@EventListener`, `@RabbitListener`, `@KafkaListener` |

### Multi-Module Project Disambiguation

In multi-module Spring projects, file audience is often determined by the module:
```
xxx-web/         → Frontend controllers (serves web/mobile)
xxx-admin/       → Admin controllers (serves admin panel)
xxx-api/         → Open API controllers (serves third-party integrators)
xxx-service/     → Business logic (shared by all controller modules)
xxx-job/         → Scheduled tasks and queue consumers
xxx-gateway/     → API gateway routing
```

### Detection Commands
```bash
# Find all controller files and their request mapping prefixes
grep -rn "@RequestMapping\|@GetMapping\|@PostMapping" --include="*.java" -l | xargs -I{} sh -c 'echo "--- {} ---"; grep "@RequestMapping" "{}"'

# Find auth annotations to determine audience
grep -rn "@RequiresPermissions\|@PreAuthorize\|@Secured\|@Inner\|@ApiKey" --include="*.java" | head -30

# Find service callers
grep -rn "@Scheduled\|@XxlJob\|@RabbitListener\|@KafkaListener\|@EventListener" --include="*.java" | head -20
```

---

## Known Version Boundaries (Reference)

> The following are feature boundary references for known versions. If the project uses a version not listed here, analyze based on actually detected version features and note version constraints in the generated rules.

### Spring Boot 1.x
- Uses `@SpringBootApplication` but configuration may be partially XML-based
- `WebMvcConfigurerAdapter` (deprecated in 2.x, use `WebMvcConfigurer`)
- `spring.datasource.url` vs `spring.datasource.jdbc-url`
- No reactive support

### Spring Boot 2.0–2.3
- HikariCP is default connection pool
- Spring Security config has changed from 1.x
- Actuator endpoint paths changed (`/actuator/health` not `/health`)
- `spring.profiles` → `spring.config.activate.on-profile` (2.4+)

### Spring Boot 2.4+
- Config file processing changed (no more `spring.profiles.include` in application.yml)
- JUnit 5 is default (4 may still be in use)

### Other Versions
For Spring Boot 3.x, Java 17+, or other versions not listed above — detect the actual framework API and language features from the project's code and dependencies. Record differences from the known versions above and flag as `LOW_CONFIDENCE` for user confirmation.

---

## Key Dependencies to Track

| Dependency | Why It Matters |
|-----------|---------------|
| `mybatis-spring-boot-starter` | Version determines available features |
| `pagehelper-spring-boot-starter` | Pagination plugin — version must match MyBatis |
| `druid-spring-boot-starter` | Connection pool with monitoring — has its own config namespace |
| `fastjson` / `fastjson2` | JSON library — 1.x has known security issues, 2.x has different API |
| `hutool` | Chinese utility library — if present, prefer its APIs |
| `knife4j` / `springfox-swagger2` | API documentation — annotation style matters |
| `spring-boot-starter-security` | Auth config pattern varies significantly by version |
| `redisson` / `spring-data-redis` | Redis client — determines distributed lock pattern |
| `rocketmq-spring-boot-starter` / `spring-kafka` | Message queue — determines async patterns |
| `spring-cloud-starter-alibaba-*` | Nacos, Sentinel, Seata — these have specific usage patterns |
