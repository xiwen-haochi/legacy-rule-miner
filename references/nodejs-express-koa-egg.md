# Node.js — Language-Specific Analysis

Read this file when the project is detected as Node.js (package.json present with backend frameworks).

## Stack Detection

### Version Detection
```bash
# Node version
cat .nvmrc .node-version 2>/dev/null
grep '"node"' package.json
grep '"engines"' -A 5 package.json

# Framework detection
grep -i '"express"\|"koa"\|"egg"\|"nest"\|"fastify"\|"hapi"\|"midway"' package.json
```

### Common Stack Variants

| Variant | Key Indicators | Era |
|---------|---------------|-----|
| Express 4.x | `"express": "^4"` | 2014–present |
| Koa 1.x | `"koa": "^1"`, generator-based | 2015–2017 |
| Koa 2.x | `"koa": "^2"`, async/await | 2017–present |
| Egg.js 1.x | `"egg": "^1"` | 2017–2018 |
| Egg.js 2.x | `"egg": "^2"` | 2018–2021 |
| Midway.js | `"@midwayjs/core"` | 2019+ |
| NestJS | `"@nestjs/core"` | 2018+ |

---

## Node.js-Specific Analysis Points

### 1. Module System

```bash
# Check CommonJS vs ES Modules
grep '"type"' package.json  # "module" = ESM, absent or "commonjs" = CJS
head -5 src/*.js 2>/dev/null  # Check for require() vs import
```

| System | Indicator | Pattern |
|--------|-----------|---------|
| CommonJS | `require()`, `module.exports` | Legacy standard |
| ES Modules | `import/export`, `"type": "module"` | Modern |
| TypeScript | `.ts` files, `tsconfig.json` | May use either |

**Critical**: If the project uses CommonJS, AI must NOT write `import/export`. This is one of the most common AI mistakes with Node.js code.

### 2. Async Pattern

This is THE most important thing to detect in legacy Node.js:

```bash
# Callback style
grep -rn "function.*callback\|function.*cb)\|err, " --include="*.js" | wc -l

# Promise style
grep -rn "\.then(\|\.catch(\|new Promise" --include="*.js" | wc -l

# Async/await style
grep -rn "async function\|await " --include="*.js" --include="*.ts" | wc -l
```

| Pattern | Era | Example |
|---------|-----|---------|
| Callbacks | Node 0.x–6.x | `fs.readFile(path, function(err, data) { ... })` |
| Promises | Node 4+ | `readFile(path).then(data => ...).catch(err => ...)` |
| Async/Await | Node 8+ | `const data = await readFile(path)` |

**Rules to generate:**
- If callbacks dominate: AI must use callback pattern, not async/await
- If mixed: follow the pattern of the specific file being modified
- If async/await: AI can use async/await but should match exact error-handling style

### 3. Project Structure

#### Express
```
project/
├── app.js / server.js / index.js   # Entry point
├── routes/
│   ├── index.js
│   └── users.js
├── controllers/                     # Sometimes combined with routes
├── middleware/
├── models/
├── services/                        # Sometimes called "logic"
├── utils/ or helpers/
├── config/
├── public/                          # Static files
└── views/                           # If server-rendered
```

#### Koa
```
project/
├── app.js
├── routes/
├── controllers/
├── middleware/
├── models/
├── services/
└── config/
```

#### Egg.js
```
project/
├── app/
│   ├── controller/
│   ├── service/
│   ├── middleware/
│   ├── model/                       # If egg-sequelize
│   ├── router.js
│   └── extend/
│       ├── helper.js
│       ├── context.js
│       └── application.js
├── config/
│   ├── config.default.js
│   ├── config.prod.js
│   └── plugin.js
└── test/
```

**Key**: Egg.js has strict conventions about file placement. If it's Egg, these conventions are mandatory rules.

### 4. Database / ORM Patterns

```bash
# Detect ORM
grep -i '"sequelize"\|"mongoose"\|"typeorm"\|"prisma"\|"knex"\|"mysql2"\|"pg"\|"mongodb"' package.json
```

| ORM | Pattern |
|-----|---------|
| Sequelize | Model.define(), model classes, migrations |
| Mongoose | Schema + Model, middleware hooks |
| TypeORM | Decorator-based entities (TypeScript) |
| Knex | Query builder, migrations |
| Raw mysql2/pg | Direct SQL queries |

Check:
- [ ] Model definition style
- [ ] Query pattern (ORM vs query builder vs raw SQL)
- [ ] Migration approach
- [ ] Connection pool configuration
- [ ] Transaction handling pattern

### 5. Middleware Chain

```bash
# Express
grep -rn "app.use(\|router.use(" --include="*.js" | head -20

# Koa
grep -rn "app.use(\|router.use(" --include="*.js" | head -20

# Egg.js — middleware config
cat config/plugin.js 2>/dev/null
```

Document the middleware order — it matters in Express/Koa:
1. Body parser
2. CORS
3. Authentication
4. Logging
5. Error handler (last in Express, first in Koa)

### 6. Error Handling

```bash
# Express error middleware (4 params)
grep -rn "function.*err.*req.*res.*next\|err, req, res, next" --include="*.js" | head -10

# Try/catch patterns
grep -rn "try\s*{" --include="*.js" --include="*.ts" | wc -l
```

Common patterns:
```javascript
// Express error middleware
app.use((err, req, res, next) => {
  logger.error(err);
  res.status(err.status || 500).json({ code: -1, msg: err.message });
});

// Koa error middleware (wraps downstream)
app.use(async (ctx, next) => {
  try { await next(); }
  catch (err) { ctx.body = { code: -1, msg: err.message }; }
});

// Egg.js — onerror plugin or custom middleware
```

### 7. Response Format

```bash
grep -rn "res\.json(\|res\.send(\|ctx\.body\s*=" --include="*.js" --include="*.ts" | head -20
```

Common legacy patterns:
```javascript
// Direct response
res.json({ code: 0, message: 'success', data: result });

// Helper function
const success = (res, data) => res.json({ code: 0, msg: 'ok', data });
const fail = (res, msg, code = -1) => res.json({ code, msg });

// Koa
ctx.body = { code: 0, data: result };
```

### 8. Configuration Management

```bash
# Config patterns
find . -name "config.*" -not -path "*/node_modules/*" | head -10
grep -rn "process\.env\.\|dotenv\|config\.get(" --include="*.js" --include="*.ts" | head -20
```

Patterns:
- `process.env.XXX` directly
- `dotenv` + `.env` files
- Config module (`config` package)
- Custom config.js exporting environment-based config
- Egg.js `config/config.{env}.js`

### 9. Logging

```bash
# Check logging library
grep -i '"winston"\|"pino"\|"bunyan"\|"log4js"\|"debug"' package.json
# Check console.log usage
grep -rn "console\.log\|console\.error\|console\.warn" --include="*.js" --include="*.ts" | wc -l
```

**Key**: Many legacy Node.js projects use `console.log` for logging. If that's the pattern, don't introduce Winston/Pino.

### 10. Package Management

### 11. Require/Import Alias Consistency
- Check if `require` variable names are consistent across files:
  - `const express = require('express')` (always? or sometimes `const app = require('express')`?)
  - `const _ = require('lodash')` vs `const lodash = require('lodash')`
  - `const { Router } = require('express')` (destructuring) vs `const Router = require('express').Router`
- For ESM projects, check import alias consistency similarly
- Module re-export patterns: does the project use `index.js` barrel exports?
- Check for mixed CommonJS/ESM patterns and which files use which

```bash
# Lock file determines package manager
ls package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null
```

| Lock File | Package Manager |
|-----------|----------------|
| `package-lock.json` | npm |
| `yarn.lock` | yarn |
| `pnpm-lock.yaml` | pnpm |

**Critical**: AI must use the same package manager. Do not suggest `yarn add` if the project uses npm.

---

## Common Anti-Patterns in Legacy Node.js

Things AI will want to "fix" but should NOT:

1. **Callback functions** — AI will convert to async/await. Don't if the file uses callbacks.
2. **var declarations** — AI will use `const`/`let`. Only if the file already uses them.
3. **require() instead of import** — AI will convert to ES modules. Never do this.
4. **console.log for logging** — AI will introduce a logging library. Don't.
5. **No TypeScript** — AI will try to add types. Don't.
6. **Deeply nested callbacks** — AI will try to flatten with async/await. Only match the file's style.
7. **module.exports = function** — AI will try to export classes or named exports. Follow the file's pattern.
8. **String concatenation** — AI will use template literals. Only if the file already uses them.
9. **No error handling in .then()** — AI will add .catch(). Follow the project's pattern for this.
10. **Global state** — Some Node.js projects use global variables or module-level state. Don't try to refactor this into dependency injection.

---

## Known Version Boundaries (Reference)

> The following are feature boundary references for known versions. If the project uses a version not listed here, analyze based on actually detected version features and note version constraints in the generated rules.

### Node 8.x
- async/await supported natively
- No optional chaining (?.)
- No nullish coalescing (??)
- No `for await...of` (added in 10)
- No `Array.flat/flatMap`

### Node 10.x
- `for await...of`
- `Array.flat()`, `Array.flatMap()`
- No optional chaining (?.)
- No nullish coalescing (??)

### Node 12.x
- Optional chaining (?.) with `--harmony-optional-chaining` flag
- Private class fields (partial)
- `globalThis`

### Node 14.x
- Optional chaining (?.) and nullish coalescing (??) natively
- `String.matchAll()`
- Top-level await (ESM only)

### Node 16.x
- `Array.at()`
- `structuredClone()`
- Stable Timers Promises API

Check the Node version in `.nvmrc`, `.node-version`, or `engines` in package.json.

### Other Versions
For Node 18+, Express 5.x, NestJS, or other versions not listed above — detect the actual runtime features and framework API from the project's code. Record differences from the known versions above and flag as `LOW_CONFIDENCE` for user confirmation.

---

## Key Dependencies to Track

| Dependency | Why It Matters |
|-----------|---------------|
| `express` | Version 4 vs 5 — middleware and routing differences |
| `koa` | Version 1 (generators) vs 2 (async/await) |
| `egg` | Convention-over-configuration — strict file structure |
| `sequelize` | Version 4/5/6 have different API patterns |
| `mongoose` | Version 5 vs 6 — connection and query API changes |
| `mysql2` / `pg` | Raw database drivers |
| `redis` / `ioredis` | Redis client — different API |
| `jsonwebtoken` | JWT handling |
| `axios` / `node-fetch` / `request` | HTTP client — `request` is deprecated |
| `lodash` | Utility functions — if present, use it instead of native methods for consistency |
| `moment` / `dayjs` | Date library — moment is in maintenance mode but widely used |
| `pm2` | Process manager — affects deployment and env vars |
