---
name: legacy-rule-miner
description: >
  Analyze legacy backend codebases and generate comprehensive AI-facing rule files (Markdown) that enable AI coding assistants to modify code correctly on the first attempt. Covers Java/Spring, PHP/ThinkPHP/Laravel, Python/Django/Flask, Node.js/Express/Koa/Egg, Go, and .NET projects. USE THIS SKILL whenever the user wants to: generate rules/instructions for an old or legacy project; create .cursorrules, .copilot-instructions.md, CLAUDE.md, or similar AI rule files from existing code; analyze a codebase to extract coding conventions, patterns, and pitfalls; document an undocumented project for AI consumption; or prepare a legacy codebase so AI can safely modify it. Also use when the user mentions "屎山", "老项目", "遗留代码", "legacy code", "rule mining", "extract conventions", or wants to ensure AI modifications are compatible with existing code style.
---

# Legacy Rule Miner

Mine conventions, patterns, and pitfalls from legacy backend codebases. Generate rule files that AI coding assistants can consume to modify code correctly on the first attempt — preserving style, respecting constraints, and avoiding known traps.

## Core Philosophy

**Forward-compatible > Rewrite.** The generated rules teach AI to work *with* the existing codebase, not against it.

Five principles govern every rule produced:

1. **Copy-Paste First** — New code mimics the nearest existing code of the same type. Find the most recent, closest example and follow it.
2. **No Surprise** — Never introduce patterns, libraries, syntax features, or architectural styles the project has never used.
3. **Version Lock** — Never upgrade dependency versions unless explicitly required. When a new dependency is unavoidable, pick versions compatible with the project's existing dependency tree.
4. **Gradual Improvement Only** — Safer idioms within a method are OK (e.g., try-with-resources replacing manual close). Refactoring method signatures, class hierarchies, or architecture is not.
5. **Pitfall Awareness** — Known traps, workarounds, and "do not touch" zones are explicitly called out. AI must respect them.

## Workflow Overview

The skill operates in two phases:

```
Phase 1 — Automated Scan (~80%)        Phase 2 — Interactive Refinement (~20%)
┌──────────────────────────────┐        ┌──────────────────────────────┐
│ L0  Detect stack & versions  │        │ L8  Interview user           │
│ L1  Map project structure    │        │     - Oral conventions       │
│ L2  Analyze configurations   │        │     - Known pitfalls         │
│ L3  Sample API / routes      │        │     - Business constraints   │
│ L4  Sample business logic    │        │     - Module-specific rules  │
│ L5  Sample data access       │        │     - Improvement boundaries │
│ L6  Cross-cutting concerns   │        │                              │
│ L7  Infrastructure / CI-CD   │        │ Refine rules based on        │
│                              │        │ user feedback                │
│ Generate draft rules         │        │ Output final rule files      │
└──────────────────────────────┘        └──────────────────────────────┘
```

## Step-by-Step Execution

### Step 0: Determine Scope

Ask the user (or infer from their prompt):
- **Whole project** or **specific module(s)**?
- If module-level: still run L0–L1 on the whole project for global context, then deep-dive into the specified module(s).

### Step 1: Detect Stack & Versions (L0)

Read build/manifest files to identify:

| Language | Files to Check |
|----------|---------------|
| Java | `pom.xml`, `build.gradle`, `build.gradle.kts` |
| PHP | `composer.json`, `composer.lock` |
| Python | `requirements.txt`, `Pipfile`, `pyproject.toml`, `setup.py` |
| Node.js | `package.json`, `package-lock.json`, `yarn.lock` |
| Go | `go.mod`, `go.sum` |
| .NET | `*.csproj`, `*.sln`, `global.json` |

Also check: `.editorconfig`, `.gitignore`, CI configs (`Jenkinsfile`, `.gitlab-ci.yml`, `.github/workflows/`), `Dockerfile`, `Makefile`.

**After detecting the language, read the corresponding language-specific reference file:**
- Java → read `references/java-spring.md`
- PHP → read `references/php-thinkphp-laravel.md`
- Python → read `references/python-django-flask.md`
- Node.js → read `references/nodejs-express-koa-egg.md`
- Go → read `references/go-specifics.md`
- .NET → read `references/dotnet-specifics.md`

Record: language, framework, framework version, runtime version, key dependencies + versions.

**Version handling**: Extract the actual version numbers from build files as-is. Do NOT assume or hardcode version numbers. If the detected version is not listed in the language reference file's "Known Version Boundaries" section, still proceed with the standard analysis flow — flag version-specific findings as `LOW_CONFIDENCE` for user confirmation.

### Step 2: Map Project Structure (L1)

Generate a directory tree (depth 3–4). Identify:
- Layering pattern (MVC? DDD? Flat?)
- Module boundaries (multi-module? monolith?)
- Resource locations (templates, static files, SQL migrations)
- Test directory structure

Create a sampling plan: for each layer (L2–L7), pick 2–3 representative files per module. Prefer files that are:
- Recently modified (more likely to reflect current conventions)
- Medium-sized (not trivially small, not monstrously large)
- In different modules (to detect cross-module consistency)

### Step 3: Layer-by-Layer Sampling (L2–L7)

Read the `references/analysis-playbook.md` for detailed analysis checklists per layer.

For each layer, extract:
1. **Patterns** — What's the common way to do X in this project?
2. **Naming conventions** — Classes, methods, variables, files, database columns
3. **Code style** — Formatting, import ordering, comment style
4. **Real code snippets** — At least 2 DO/DON'T examples per pattern found
5. **Inconsistencies** — Where patterns break (often signals important exceptions)

**Comment archaeology**: Search for `TODO`, `FIXME`, `HACK`, `WORKAROUND`, `XXX`, `@deprecated` — these reveal hidden constraints and known issues.

### Step 4: Generate Draft Rules

Read `references/rule-writing-guide.md` for rule formatting standards.
Read `references/output-templates.md` for the skeleton of each output file.

Generate a single rule file `.rules.md` in the project root (or user-specified location). The file is organized by `##` sections:

| Section | Content | Minimum Requirements |
|---------|---------|---------------------|
| `## Project Overview` | Tech stack, versions, modules, build commands, env vars | Stack + version + build command |
| `## Architecture` | Layer rules, module boundaries, dependency directions, where to put new code | Layer pattern + new-code placement |
| `## Naming Conventions` | Naming rules with real examples for each entity type | ≥3 entity types with examples |
| `## Coding Patterns` | Error handling, logging, validation, import conventions, return wrapping — with DO/DON'T | ≥3 patterns with code examples |
| `## API Design` | URL patterns, request/response format, error codes, middleware | URL pattern + response format |
| `## Data Access` | ORM usage, SQL patterns, transaction management, DB naming, field change strategy | ORM pattern + transaction rule |
| `## Security` | Auth pattern, input validation, sensitive data handling | Auth mechanism + validation approach |
| `## Dependencies` | Dependency inventory (with versions), new-dep policy, compat constraints | Top 10 deps with versions |
| `## Known Pitfalls` | Known traps, historical workarounds, do-not-touch zones | All items from comment archaeology |
| `## Custom Rules` | Reserved for user-added rules. **Never overwrite this section.** | Empty placeholder on first generation |

Mark uncertain items with `<!-- LOW_CONFIDENCE: reason -->` for user review in Phase 2.

**Incremental update**: If `.rules.md` already exists, update auto-generated sections (Project Overview through Known Pitfalls) while **preserving** the `## Custom Rules` section and any content the user has added there.

### Step 5: Interactive Refinement (L8)

Read `references/interview-questions.md` for the question bank.

Present a summary of discovered rules to the user. Then ask targeted questions about:
- Items marked `LOW_CONFIDENCE`
- Oral conventions that code analysis cannot detect
- Business domain constraints
- Known pitfalls and workarounds not captured in comments
- Which modules need module-specific rules
- Boundaries of acceptable "gradual improvement"

### Step 6: Finalize

Apply user feedback. Generate module-specific rules if needed as separate files: `.rules-{module}.md` (e.g., `.rules-payment.md`, `.rules-api.md`). If a module rule file already exists, update it; if not, create a new one.

Output a brief summary: files generated/updated, key findings, confidence level, and any remaining unknowns.

---

## Sampling Strategy for Large Projects

| Project Size | Files | Samples per Layer |
|-------------|-------|------------------|
| Small | < 100 | 3–5 files |
| Medium | 100–500 | 2–3 files per module |
| Large | 500+ | 2 files per module, focus on core modules |

For very large projects (1000+ files), use subagents to parallelize L2–L7 analysis across modules.

## Reference Files

Read these as needed during analysis:

| File | When to Read |
|------|-------------|
| `references/analysis-playbook.md` | Step 3 — detailed checklists for each analysis layer |
| `references/rule-writing-guide.md` | Step 4 — how to write effective AI-facing rules |
| `references/output-templates.md` | Step 4 — output file skeletons and minimum requirements |
| `references/interview-questions.md` | Step 5 — question bank for interactive refinement |
| `references/java-spring.md` | Step 1 — when Java/Spring detected |
| `references/php-thinkphp-laravel.md` | Step 1 — when PHP detected |
| `references/python-django-flask.md` | Step 1 — when Python detected |
| `references/nodejs-express-koa-egg.md` | Step 1 — when Node.js detected |
| `references/go-specifics.md` | Step 1 — when Go detected |
| `references/dotnet-specifics.md` | Step 1 — when .NET detected |

## Output Location

By default, create files in the project root. The user can override this.

```
<project-root>/
├── .rules.md              # Main rule file (single file, sections by dimension)
├── .rules-payment.md      # Module-specific rules (only if needed)
└── .rules-auth.md         # Module-specific rules (only if needed)
```

`.rules.md` structure:
```markdown
# {Project Name} — AI Coding Rules
## Project Overview
## Architecture
## Naming Conventions
## Coding Patterns
## API Design
## Data Access
## Security
## Dependencies
## Known Pitfalls
## Custom Rules          ← user-maintained, never overwritten
```

The user copies this into their AI tool's config (`.copilot-instructions.md`, `.cursorrules`, `CLAUDE.md`, etc.) or references it directly.
