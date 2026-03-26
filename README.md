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

Generates rule files in the IDE-native rules directory (auto-detected):

| IDE/Tool | Output Directory |
|----------|------------------|
| Cursor | `.cursor/rules/` |
| Trae | `.trae/rules/` |
| GitHub Copilot (VS Code) | `.github/instructions/` |
| Claude Code | `.claude/rules/` |
| Windsurf | `.windsurf/rules/` |
| Unknown / Fallback | `.ruleminer/` |

```
{output-dir}/
├── legacy-rules.md              # Main rule file (Project Overview / Architecture / Naming / Coding / API / Data / Security / Dependencies / Pitfalls / Custom Rules)
├── legacy-rules-payment.md      # Module-level rules (only generated when a module has special conventions)
└── legacy-rules-auth.md         # Module-level rules (only generated when a module has special conventions)
```

Features:
- A `## Custom Rules` section is preserved at the end of the file — users can add rules manually, and regeneration will not overwrite it
- When `legacy-rules.md` already exists, performs incremental updates rather than full overwrite
- Module-level rules are updated if they exist, created if they don't

## Installation

### Via skills.sh (Recommended)

Run in your terminal:

```bash
npx skills add xiwen-haochi/legacy-rule-miner
```

This will install the skill into your IDE's skill directory automatically.

### Manual Installation

1. Clone this repo:
   ```bash
   git clone https://github.com/xiwen-haochi/legacy-rule-miner.git
   ```
2. Copy the entire folder into your IDE's skill directory:
   - **VS Code (Copilot):** `<project-root>/.github/skills/legacy-rule-miner/`
   - **Cursor:** `<project-root>/.cursor/skills/legacy-rule-miner/`
   - **Claude Code:** `<project-root>/.agents/skills/legacy-rule-miner/`

## Usage

### Quick Start

1. **Open your legacy project** in an AI coding IDE (VS Code with Copilot / Cursor / Claude Code)
2. **Make sure the skill is installed** (see Installation above)
3. **Start a chat session** and ask the AI to analyze your project:
   ```
   Analyze this legacy project and generate legacy-rules.md
   ```
4. The skill will automatically:
   - Detect your IDE and choose the appropriate output directory
   - Scan your project structure, configs, dependencies, and source code (~80% automated)
   - Ask you follow-up questions about oral conventions, known pitfalls, and business constraints (~20% interactive)
   - Generate `legacy-rules.md` in the IDE-native rules directory

### Re-running / Updating

- If `legacy-rules.md` already exists, the skill performs **incremental updates** — only changed sections are updated
- The `## Custom Rules` section at the end of the file is **never overwritten**, so your manual additions are safe
- To force a full regeneration, delete `legacy-rules.md` and re-run

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
