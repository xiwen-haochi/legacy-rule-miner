# Go — Language-Specific Analysis

Read this file when the project is detected as Go (go.mod or *.go files).

## Stack Detection

### Version Detection
```bash
# Go version
head -3 go.mod  # "go 1.xx" directive

# Framework/router detection
grep -i "gin-gonic\|gorilla/mux\|chi\|echo\|beego\|iris\|fiber" go.mod go.sum 2>/dev/null
```

### Common Stack Variants

| Variant | Key Indicators | Era |
|---------|---------------|-----|
| No framework (net/http) | Only stdlib in go.mod | Any |
| Gin | `github.com/gin-gonic/gin` | 2017+ |
| Beego | `github.com/beego/beego` | 2013–2019 |
| Echo | `github.com/labstack/echo` | 2016+ |
| Gorilla/Mux + negroni | `github.com/gorilla/mux` | 2013–2019 |
| Chi | `github.com/go-chi/chi` | 2017+ |
| Fiber | `github.com/gofiber/fiber` | 2020+ |
| gRPC | `google.golang.org/grpc` | 2016+ |

---

## Go-Specific Analysis Points

### 1. Project Structure

```bash
# Common Go project layouts
ls -la cmd/ internal/ pkg/ api/ 2>/dev/null
```

| Layout | Description |
|--------|------------|
| Flat | Everything in root package — small projects |
| cmd/pkg/internal | Standard Go project layout |
| cmd + domain dirs | DDD-inspired — `cmd/`, `user/`, `order/` |
| MVC-like | `controllers/`, `models/`, `services/` |

Check:
- [ ] Is `internal/` used? (package visibility constraint)
- [ ] `cmd/` for multiple binaries?
- [ ] `pkg/` for shared libraries?
- [ ] `api/` for protocol definitions (protobuf, OpenAPI)?

### 2. Error Handling Pattern

This is THE most critical Go convention to capture:

```bash
# Check error handling style
grep -rn "if err != nil" --include="*.go" | head -5
grep -rn "errors\.Wrap\|errors\.New\|fmt\.Errorf.*%w" --include="*.go" | head -10
grep -rn "pkg/errors\|golang\.org/x/xerrors" go.mod 2>/dev/null
```

| Pattern | Example |
|---------|---------|
| Standard | `if err != nil { return err }` |
| Wrapped (Go 1.13+) | `return fmt.Errorf("failed to create user: %w", err)` |
| pkg/errors | `return errors.Wrap(err, "create user")` |
| Custom error types | `type BusinessError struct { Code int; Msg string }` |
| Sentinel errors | `var ErrNotFound = errors.New("not found")` |

**Key**: Consistency matters enormously. If the project uses `fmt.Errorf("%w")`, don't use `pkg/errors.Wrap`. If the project returns bare `err`, don't add wrapping.

### 3. Dependency Injection / Initialization

```bash
# Check DI approach
grep -rn "wire\.\|fx\.\|dig\.\|inject\." --include="*.go" go.mod 2>/dev/null
# Check manual wiring
grep -rn "func New\|func Init" --include="*.go" | head -20
```

| Approach | Indicator |
|----------|-----------|
| Manual wiring | `NewXxxService(repo, logger)` constructors |
| Wire | `google.golang.org/wire` |
| Uber fx | `go.uber.org/fx` |
| Dig | `go.uber.org/dig` |

Most legacy Go projects use manual wiring. AI should follow whatever pattern exists.

### 4. Logging

```bash
# Check logging library
grep -rn "logrus\|zap\|log\.Print\|log\.Info\|slog\." --include="*.go" go.mod 2>/dev/null
```

| Library | Pattern |
|---------|---------|
| stdlib `log` | `log.Printf("message: %v", err)` |
| logrus | `logrus.WithField("key", val).Info("msg")` |
| zap | `logger.Info("msg", zap.String("key", val))` |
| zerolog | `log.Info().Str("key", val).Msg("msg")` |
| slog (Go 1.21+) | `slog.Info("msg", "key", val)` |

**Key**: Log style is one of the most inconsistent things in Go projects. Document the actual pattern used.

### 5. HTTP Handler Patterns

#### Standard net/http
```go
func userHandler(w http.ResponseWriter, r *http.Request) {
    // ... 
}
```

#### Gin
```go
func getUser(c *gin.Context) {
    id := c.Param("id")
    // ...
    c.JSON(200, gin.H{"code": 0, "data": user})
}
```

#### Echo
```go
func getUser(c echo.Context) error {
    id := c.Param("id")
    return c.JSON(200, map[string]interface{}{"code": 0, "data": user})
}
```

Check:
- [ ] Request parameter binding pattern
- [ ] Response format (JSON envelope, error codes)
- [ ] Middleware registration order
- [ ] Context usage (passing values, cancellation)

### 6. Database Patterns

```bash
# ORM/DB detection
grep -rn "gorm\|sqlx\|database/sql\|ent\.\|xorm\|go-pg" go.mod 2>/dev/null
```

| ORM | Pattern |
|-----|---------|
| database/sql | Raw SQL, manual scanning |
| sqlx | Named queries, struct scanning |
| GORM | `db.Where(...).Find(&users)` |
| GORM v2 | `db.Where(...).Find(&users)` (different import path) |
| Ent | Code-generated, schema-defined |
| xorm | `engine.Where(...).Find(&users)` |

Check:
- [ ] GORM v1 (`github.com/jinzhu/gorm`) vs v2 (`gorm.io/gorm`) — very different APIs
- [ ] Transaction pattern: `db.Transaction(func(tx) error { ... })`
- [ ] Migration approach: GORM AutoMigrate, goose, golang-migrate, manual SQL
- [ ] Connection pool settings

### 7. Package Organization

```bash
# Check interface definitions
grep -rn "type.*interface {" --include="*.go" | head -20

# Check struct definitions
grep -rn "type.*struct {" --include="*.go" | head -20
```

Go conventions to check:
- [ ] Interface naming: `XXXer` pattern or domain-specific
- [ ] Interface location: with implementation or separate package
- [ ] Unexported vs exported types
- [ ] Constructor pattern: `func NewXxx(...)` or `func New(...)`
- [ ] Method receiver style: pointer `(s *Service)` vs value `(s Service)`

### 8. Concurrency Patterns

```bash
grep -rn "go func\|sync\.Mutex\|sync\.RWMutex\|sync\.WaitGroup\|chan \|<-\s*chan\|select {" --include="*.go" | wc -l
```

Check:
- [ ] Goroutine spawning patterns (are they controlled? Is there a pool?)
- [ ] Channel usage patterns
- [ ] Mutex usage patterns
- [ ] Context propagation for cancellation
- [ ] Error group usage (`golang.org/x/sync/errgroup`)

**Critical**: Concurrency code in Go is extremely sensitive. Document exact patterns and mark as "do not modify" unless the user explicitly requests it.

### 9. Testing Patterns

```bash
# Test framework
grep -rn "testify\|gomock\|mockery\|goconvey\|ginkgo" go.mod 2>/dev/null

# Test file patterns
find . -name "*_test.go" | head -10
```

Check:
- [ ] Test naming: `TestXxx` (standard) or table-driven
- [ ] Mock generation: gomock, mockery, manual mocks
- [ ] Test helper patterns
- [ ] Integration test separation (build tags?)

### 10. Configuration

### 11. Import & Package Alias Consistency
- Check import grouping convention: stdlib / third-party / internal (Go standard)
- Check for package aliases: `import pg "github.com/lib/pq"` — are aliases consistent across files?
- Check for blank import patterns: `import _ "github.com/go-sql-driver/mysql"` — document all of them
- Dot imports (`import . "testing"`) — does the project use them? If not, AI must not introduce them
- Check if `goimports` or `gofumpt` is used (enforces import style automatically)

```bash
# Config library
grep -rn "viper\|envconfig\|godotenv\|toml\|yaml" go.mod 2>/dev/null

# Config file
find . -name "*.yaml" -o -name "*.toml" -o -name "*.json" | grep -i config | head -5
```

Patterns:
- Viper (multi-source config)
- Environment variables directly
- Config struct with envconfig tags
- YAML/TOML config files
- Custom config loading

---

## Common Anti-Patterns in Legacy Go

Things AI will want to "fix" but should NOT:

1. **init() functions** — AI will try to eliminate them. They're intentional in many Go projects.
2. **Global variables** — AI will try to inject dependencies. Follow the project's pattern.
3. **panic/recover** — AI will try to convert to error returns. Some frameworks (like Gin) use panic/recover intentionally.
4. **Bare error returns without wrapping** — AI will try to add error wrapping. Only if the project does it.
5. **String error messages** — AI will try to create sentinel errors. Only if the project uses them.
6. **Interface declarations with implementations** — AI may move interfaces to consumer packages. Follow the project's convention.
7. **Large switch statements** — AI will try to refactor into strategy pattern. Don't in Go.
8. **context.TODO()** — AI will try to replace with proper context. Only if there's a pattern for it in the project.

---

## Known Version Boundaries (Reference)

> The following are feature boundary references for known versions. If the project uses a version not listed here, analyze based on actually detected version features and note version constraints in the generated rules.

### Go 1.10–1.11
- No `go.mod` module support (or early experimental)
- GOPATH-based project layout
- No error wrapping with `%w`

### Go 1.13
- Error wrapping: `fmt.Errorf("%w", err)`, `errors.Is()`, `errors.As()`
- Module support stable

### Go 1.14–1.15
- Module stable, `GONOSUMCHECK` env
- Package embedding not yet available

### Go 1.16
- `//go:embed` for file embedding
- `os.ReadFile()`, `os.WriteFile()` (no more `ioutil`)
- Module-aware mode by default

### Go 1.18
- Generics
- `any` alias for `interface{}`
- `strings.Cut()`

### Go 1.21
- `slog` structured logging in stdlib
- `maps` and `slices` packages

Check `go.mod` for the Go version and ensure AI doesn't use features above that version.

### Other Versions
For Go 1.22+, or other versions not listed above — detect the actual language features from the project's `go.mod` and code. Record differences from the known versions above and flag as `LOW_CONFIDENCE` for user confirmation.

---

## Key Dependencies to Track

| Dependency | Why It Matters |
|-----------|---------------|
| `gin-gonic/gin` | HTTP framework — middleware chain pattern |
| `gorm.io/gorm` vs `jinzhu/gorm` | GORM v2 vs v1 — very different APIs |
| `go-redis/redis` | Redis client — v8 vs v9 differ |
| `uber-go/zap` | Structured logging |
| `spf13/viper` | Configuration management |
| `golang/protobuf` vs `google.golang.org/protobuf` | Protobuf — old vs new API |
| `grpc` | gRPC service patterns |
| `swaggo/swag` | Swagger doc generation |
| `casbin` | Authorization — common in Chinese Go projects |
| `golang-jwt/jwt` | JWT handling |
