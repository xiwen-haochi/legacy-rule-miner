# Rule Writing Guide

How to write effective AI-facing rules for legacy backend projects. Read this during Step 4 of the workflow.

## Core Principle

Every rule exists to help an AI coding assistant modify this project's code **correctly on the first attempt**. Rules are commands to the AI, not documentation for humans.

---

## Rule Severity Levels

Use these four levels consistently:

| Level | Meaning | When to Use |
|-------|---------|-------------|
| **MUST** | Violation will break the build, cause bugs, or violate a hard constraint | Compilation requirements, framework mandates, version locks |
| **NEVER** | Doing this will cause harm; the inverse of MUST | Forbidden patterns, dependency upgrades, architecture changes |
| **SHOULD** | Strong preference; deviations need justification | Naming conventions, code style, common patterns |
| **PREFER** | Soft preference; the project's current way is better but alternatives won't break things | Minor style choices, gradual improvements |

**Calibration:** 60% of rules should be SHOULD, 20% MUST, 10% NEVER, 10% PREFER. If you have 50% MUST rules, the AI will treat none of them as truly critical.

---

## Rule Format

Each rule follows this structure:

```markdown
### [Short descriptive title]

**[LEVEL]**: [Imperative statement — what the AI must do or not do]

**Why**: [1–2 sentences explaining the reason. This helps the AI generalize to cases the rule doesn't explicitly cover.]

**DO:**
```[language]
// Real code from the project showing the correct pattern
```

**DON'T:**
```[language]
// What the AI might write if left to its own preferences
```
```

### Compact format for simple rules

When a rule is straightforward and doesn't need code examples:

```markdown
- **MUST**: Use `snake_case` for all database column names.
- **SHOULD**: Log at INFO level on service method entry with method name and key parameters.
- **NEVER**: Import `org.apache.commons.lang.StringUtils` — use `org.apache.commons.lang3.StringUtils` instead.
```

---

## Writing Effective Rules

### 1. Be specific, not abstract

Bad:
> SHOULD: Follow consistent naming conventions.

Good:
> SHOULD: Name service methods with the verb prefixes used in this project: `get*` for single-entity retrieval, `list*` for collection retrieval, `create*` for inserts, `update*` for modifications, `delete*` for removals. Do not use `find*`, `fetch*`, `save*`, or `remove*`.

### 2. Include the project's real code

Rules without code examples are vague. Always extract real snippets:

```markdown
### Return value wrapping

**MUST**: All controller methods return `Result<T>` using the project's `Result.success()` / `Result.fail()` static factories.

**Why**: The frontend depends on the `{code, msg, data}` envelope structure. Returning raw objects or ResponseEntity will break the frontend.

**DO:**
```java
@PostMapping("/users")
public Result<UserVO> createUser(@RequestBody @Valid CreateUserDTO dto) {
    UserVO user = userService.createUser(dto);
    return Result.success(user);
}
```

**DON'T:**
```java
@PostMapping("/users")
public ResponseEntity<UserVO> createUser(@RequestBody CreateUserDTO dto) {
    UserVO user = userService.createUser(dto);
    return ResponseEntity.ok(user);
}
```
```

### 3. Explain WHY — it helps the AI generalize

The AI will encounter cases your rules don't explicitly cover. If it understands *why* a rule exists, it can make better judgments:

```markdown
**Why**: The project uses MyBatis XML mappers exclusively. JPA annotations exist on some entity classes 
as leftover from a migration attempt that was abandoned in 2019 — they are not functional. 
If you see @Entity annotations, they are decorative only.
```

### 4. Call out the "AI instinct" traps

AI assistants have strong priors — patterns they'll reach for unless told otherwise. Anticipate these:

| AI Instinct | Rule to Counter |
|------------|----------------|
| Use the latest version of a library | **NEVER**: Upgrade dependency versions |
| Extract a helper method | **PREFER**: Keep code inline if the project's existing methods in the same class are inline |
| Add null checks everywhere | **SHOULD**: Follow the project's existing null-handling pattern (which may be "let it NPE and catch upstream") |
| Use Optional<T> | **SHOULD**: Use Optional only if the project already uses it; otherwise return null |
| Add @Autowired to fields | **MUST**: Use constructor injection if the file's other classes use it |
| Write unit tests | **SHOULD**: Follow the existing test pattern — if tests use Mockito, use Mockito; don't introduce AssertJ if JUnit assertions are the norm |
| Use Stream API | **PREFER**: Use streams only if the project's Java version supports them AND existing code uses them |
| Pick arbitrary import alias | **MUST**: Follow the project's existing import alias convention (e.g., if the project uses `from datetime import datetime`, don't use `import datetime as dt`) |
| ALTER COLUMN to rename/retype | **PREFER**: Add new column + deprecate old column instead of altering existing columns directly. Follow the project's migration convention. |
| Define constants locally | **MUST**: Use the project's canonical constant source (e.g., `ErrorCode` class). Never redefine constants in multiple files. |

### 5. Address the "Copy-Paste First" principle

The most powerful rule pattern for legacy code:

```markdown
### Adding a new API endpoint

**MUST**: Before writing a new controller method, find the most similar existing endpoint in the same 
controller (or a neighboring controller). Copy its structure exactly:
- Same annotation style
- Same parameter binding approach  
- Same return type wrapping
- Same exception handling pattern
- Same logging calls

Then modify only the business-specific parts.

**Why**: This project has organically accumulated patterns. Copying from a recent, working example is 
safer than writing from scratch, even if the AI "knows" a better way.
```

---

## Organizing Rules by Section

All rules go into a single `legacy-rules.md` file with `##` sections. Module-specific rules go into separate `legacy-rules-{module}.md` files.

### Project Overview
- Use plain description, not rules
- Include: language, framework, version, module map, build commands, test commands
- This file is the AI's "first read" — keep it scannable

### Architecture
- Focus on "where to put new code" decisions
- Include module dependency diagram (text-based)
- List prohibited patterns (what the project does NOT use)

### Naming Conventions
- Organize by entity type: class, method, variable, constant, file, package/namespace, database table, database column, API URL
- For each: give 3+ real examples from the project
- Call out naming inconsistencies if they exist (e.g., "some old services use `*Manager`, new ones use `*Service` — use `*Service` for new code")

### Coding Patterns
- Group by concern: error handling, logging, validation, return values, null handling, date/time handling, import conventions, constant management
- Each group: show the project's actual pattern with DO/DON'T
- This is often the largest file — keep it focused on patterns that affect day-to-day coding

### API Design
- URL patterns with examples
- Request/response format specification
- Error code table (if one exists)
- Pagination conventions
- Authentication/authorization annotations or middleware

### Data Access
- ORM usage patterns with examples
- Transaction management rules
- Query patterns (builder vs raw SQL vs XML)
- Entity/Model conventions
- Database naming standards
- Field change strategy (add > modify)
- Cross-table field dependencies

### Security
- Auth flow description
- Input validation rules
- Sensitive data handling
- Known security considerations

### Dependencies
- Table of key dependencies with pinned versions
- Policy for introducing new dependencies
- Known compatibility constraints between libraries

### Known Pitfalls
- Zero-tolerance for ambiguity here
- Each pitfall: what it is, where it is, what happens if violated
- Include file paths and line numbers where possible
- Sort by severity (production-breaking first)

### Module-Level Rules (legacy-rules-{module}.md)
- Only created for modules with rules that differ from the global rules
- Reference the global rules ("Inherits all rules from `legacy-rules.md`. The following are additional/override rules")

---

## Language-Specific Writing Tips

### Java
- Reference annotations precisely: `@Transactional(propagation = Propagation.REQUIRED)`
- Include full import paths when disambiguation matters
- Note Java version limitations (e.g., "no var keyword — project is on Java 8")

### PHP
- Note framework version-specific APIs (ThinkPHP 5 vs 5.1 differ significantly)
- Include namespace patterns
- Call out which global functions vs. facade methods are preferred

### Python
- Note Python version (3.6 lacks f-strings with `=` debug, 3.7 lacks walrus operator)
- Include import conventions (absolute vs relative, order)
- Specify type hint usage (if the project uses them vs. doesn't)

### Node.js
- Specify callback vs Promise vs async/await convention
- Note CommonJS (`require`) vs ES modules (`import`)
- Include error-first callback patterns if applicable

### Go
- Note Go version (affects available language features)
- Include error handling pattern (the most critical consistency point)
- Specify package naming and directory conventions

### .NET
- Note .NET version and C# language version
- Specify async/await patterns
- Include dependency injection registration patterns

---

## Quality Checklist

Before finalizing any rule file, verify:

- [ ] Every MUST/NEVER rule has a real code example
- [ ] No rules contradict each other across files
- [ ] Rules are specific enough that two developers would interpret them the same way
- [ ] The AI's "natural instinct" is anticipated and addressed
- [ ] Version numbers are pinned for critical dependencies
- [ ] File paths in examples are relative to the project root
- [ ] Each pitfall includes enough context for the AI to recognize the situation
- [ ] Low-confidence rules are marked with `<!-- LOW_CONFIDENCE: reason -->`
