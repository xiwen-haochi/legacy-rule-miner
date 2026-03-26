# Legacy Rule Miner

Automatically extract coding conventions, architecture patterns, and known pitfalls from legacy backend codebases, generating AI-consumable rule files so that AI coding assistants (Copilot / Cursor / Claude) can modify legacy code **correctly on the first attempt**.

## Core Philosophy

**Forward-compatible > Rewrite**

Generated rules teach AI how to work WITH the existing codebase, not against it.

## Supported Languages / Frameworks

| Language | Frameworks |
|----------|-----------|
| Java | Spring Boot, SSM, Spring Cloud |
| PHP | ThinkPHP, Laravel, Yii |
| Python | Django, Flask, FastAPI, Tornado |
| Node.js | Express, Koa, Egg.js, NestJS |
| Go | Gin, Beego, Echo, net/http |
| .NET | .NET Core, .NET 5+ |

## Workflow

1. **Automated Scan** (~80%) — Layer-by-layer analysis of project metadata, structure, configuration, API, business logic, data access, cross-cutting concerns, and infrastructure
2. **Interactive Refinement** (~20%) — Gather oral conventions, known pitfalls, business domain constraints, and other information that code analysis cannot detect

## Output

Generates a single rule file (with dimension-based sections) at the project root:

```
<project-root>/
├── .rules.md              # Main rule file (Project Overview / Architecture / Naming / Coding / API / Data / Security / Dependencies / Pitfalls / Custom Rules)
├── .rules-payment.md      # Module-level rules (only generated when a module has special conventions)
└── .rules-auth.md         # Module-level rules (only generated when a module has special conventions)
```

Features:
- A `## Custom Rules` section is preserved at the end of the file — users can add rules manually, and regeneration will not overwrite it
- When `.rules.md` already exists, performs incremental updates rather than full overwrite
- Module-level rules are updated if they exist, created if they don't

## Installation

```bash
npx skills add <your-github-username>/legacy-rule-miner
```

## Usage

Point this Skill at your legacy project in an AI coding IDE and run the analysis. The generated `.rules.md` can be directly copied to `.copilot-instructions.md`, `.cursorrules`, `CLAUDE.md`, or other AI config files.

## Five Core Principles

1. **Copy-Paste First** — New code mimics the most recent similar existing code
2. **No Surprise** — Never introduce patterns/libraries/syntax the project hasn't used
3. **Version Lock** — Never upgrade dependency versions
4. **Gradual Improvement Only** — Allow small improvements within methods, never refactor architecture
5. **Pitfall Awareness** — Document all known pitfalls

## File Structure

```
legacy-rule-miner/
├── SKILL.md                          # Main Skill file
├── references/
│   ├── analysis-playbook.md          # Analysis dimensions & sampling strategy
│   ├── rule-writing-guide.md         # Rule writing guide
│   ├── output-templates.md           # Output templates
│   ├── interview-questions.md        # Interactive question bank
│   ├── java-spring.md               # Java-specific
│   ├── php-thinkphp-laravel.md      # PHP-specific
│   ├── python-django-flask.md       # Python-specific
│   ├── nodejs-express-koa-egg.md    # Node.js-specific
│   ├── go-specifics.md              # Go-specific
│   └── dotnet-specifics.md          # .NET-specific
├── evals/                           # Test cases
└── README.md
```

## License

See [LICENSE](LICENSE) for details.
