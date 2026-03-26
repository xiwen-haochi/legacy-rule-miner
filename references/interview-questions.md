# Interview Questions

Question bank for Phase 2 — Interactive Refinement (L8). Use these to uncover rules that code analysis cannot detect.

Read this file during Step 5 of the workflow. Select questions based on what the automated scan revealed, focusing on gaps and low-confidence items.

---

## How to Use

1. **Don't ask all questions** — pick the most relevant ones based on scan results
2. **Group related questions** — use the askQuestions tool with batches of 3–5 questions
3. **Start with high-impact questions** — oral conventions and known pitfalls first
4. **Skip questions already answered by code analysis** — if the scan already determined the answer with high confidence, don't waste the user's time
5. **Use discovered patterns as context** — "I noticed your services use `@Transactional`. Are there cases where this should NOT be used?"

---

## Round 1: Oral Conventions & Team Practices

These are the highest-value questions — they reveal rules invisible to code analysis.

### Team Conventions
- Are there any coding rules your team follows that aren't written down anywhere? (e.g., "we always put new APIs in a specific controller", "we never use feature X")
- Are there any "everyone knows but nobody documented" rules about this project?
- When a new team member joins, what do you tell them about this codebase that they wouldn't figure out on their own?

### Code Modification Norms
- Are there files or modules that should never be modified, or require special approval?
- Are there areas that are known to be fragile — where a small change can cause unexpected breakage?
- Is there code that looks wrong or outdated but actually works correctly and should not be "fixed"?

### Review Standards
- What are the most common reasons code gets rejected in review?
- Are there any patterns you explicitly tell new developers NOT to use, even if they seem modern/better?

---

## Round 2: Business Domain Constraints

### Data Constraints
- Are there business rules about data that aren't enforced in code? (e.g., "orders older than 30 days cannot be cancelled", "user IDs must never be reused")
- Are there fields in the database that have special meanings not obvious from the name? (e.g., `status=5` means "archived but billable")
- Are there data migration artifacts — old-format records mixed with new-format ones?

### Business Logic
- Are there business processes with strict ordering requirements? (e.g., "payment must complete before inventory is decremented")
- Are there seasonal or time-dependent behaviors? (e.g., "year-end settlement runs differently")
- Are there customer-specific or region-specific logic branches?

### External Dependencies
- Are there third-party APIs with known quirks? (e.g., "the payment gateway returns 200 on some error cases")
- Are there rate limits, timeouts, or retry policies that must be respected?
- Are there external systems that are only available during certain hours?

---

## Round 3: Known Pitfalls & War Stories

### Production Incidents
- Has the project had any production incidents caused by code changes? What happened?
- Are there any "war stories" about things that went wrong that should inform how code is written?

### Performance Traps
- Are there known performance bottlenecks that should not be made worse?
- Are there queries that are intentionally slow (e.g., batch jobs) and should not be "optimized" in a way that changes behavior?
- Are there caching strategies that must be preserved?

### Concurrency Issues
- Has the project experienced race conditions? Where?
- Are there operations that must be idempotent?
- Are there distributed lock patterns that must be followed?

---

## Round 4: Scan Results Review

Present automated scan findings and ask for corrections/additions.

### Patterns Review
- I found that this project uses [pattern X]. Is that correct, or are there exceptions?
- I noticed [inconsistency Y] between modules/files. Is this intentional?
- I marked [rule Z] as LOW_CONFIDENCE. Can you confirm or correct it?

### Completeness Check
- Are there any conventions I missed about:
  - Error handling?
  - Logging?
  - Database operations?
  - API design?
  - Authentication/Authorization?
  - Import/alias conventions?
- Are there any modules with special rules I haven't covered?

### Improvement Boundaries
- Which of these patterns are OK to gradually improve (use safer/modern syntax without changing architecture)?
- Which must stay exactly as they are, even if the current approach has known issues?
- If I need to introduce a new dependency, what's the approval process?
- Do certain database fields need to be updated together across tables? (e.g., renaming a user also updates order records)

---

## Round 5: Module-Specific Deep Dive

Ask these only when analyzing specific modules or when the scan detected modules with potential special rules.

### Module Context
- What does this module do and how critical is it?
- Who typically works on this module?
- Is this module planned for rewrite/deprecation?

### Module-Specific Rules
- Does this module have any conventions that differ from the rest of the project?
- Are there any known issues specific to this module?
- Are there any external system integrations unique to this module?

---

## Question Format for askQuestions Tool

When using the askQuestions tool, structure questions as selections where possible for faster response:

```
Question: "I found that your Service classes use field injection (@Autowired). Should AI-generated code continue this pattern?"
Options:
- "Yes, keep using field injection" (recommended based on scan)
- "No, use constructor injection for new code"
- "Depends on the specific case"
Free text: "Any additional context?"
```

For open-ended questions, use free-text format:

```
Question: "What coding rules does your team follow that aren't written down anywhere?"
Free text only, with a helpful prompt like "e.g., 'we never modify the billing module without approval'"
```

---

## Adaptive Questioning Strategy

### If the user seems busy/terse:
- Reduce to 5–8 essential questions
- Focus on Round 1 (oral conventions) and Round 3 (known pitfalls)
- Use selection-based questions for faster answers

### If the user is engaged and detailed:
- Go through all rounds
- Ask follow-up questions on interesting answers
- Dive deep into module-specific rules

### If the user says "I don't know" frequently:
- They may not be the right person for these questions
- Suggest they consult another team member
- Proceed with code-analysis-based rules and mark more items as LOW_CONFIDENCE
