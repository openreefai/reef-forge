# Issue Templates and Classification Guide

Reference material for drafting ecosystem issues against openreef, tide, and openclaw.

---

## Issue Draft Template

Use this template for all repos. Adapt the Evidence section to the repo context.

```
**Target Repo:** openreef | tide | openclaw
**Severity:** critical | moderate | minor
**Duplicate Check:** None found | Potential duplicate: #NNN — [reason]

**Title:** [Area prefix]: [concise, actionable description]

## Description
[What is the problem. Factual, concise.]

## Evidence
- **File:** [path/to/file.ext, lines N-M]
- **Observed behavior:** [what actually happens]
- **Reproduction steps:**
  1. [Step]
  2. [Step]
  3. Observe: [result]

## Expected Behavior
[What should happen instead, per docs or convention.]

## Found During
[Formation name, version, and review type, e.g., "reef-forge QA review of weather-alerts v0.1.0"]

## Severity Assessment
[One sentence justifying the classification.]
```

**Repo-specific notes:**

- **openreef** -- Prefix titles with the area (e.g., "Schema validation:", "Agent lifecycle:"). Common categories: schema validation bugs, config parsing failures, documentation drift.
- **tide** -- Include registry state in Evidence when relevant (available package versions, constraint expressions). Common categories: resolution errors, version constraint handling, search/discovery bugs.
- **openclaw** -- Include agent config (sandbox mode, tool permissions) in Evidence. Common categories: sandbox escapes, tool permission violations, skill execution failures, message routing errors.

---

## Severity Classification

### Critical
Breaks core functionality or creates a security risk. Users are blocked or data is at risk.
- Valid reef.json rejected by `reef validate`, preventing deployment
- Sandbox restrictions bypassed, allowing unintended access
- Messages routed outside the declared `agentToAgent` graph
- Credentials logged in plaintext

If you classify more than one finding as critical per review, double-check that each truly meets the bar.

### Moderate
Incorrect behavior or materially wrong documentation, but workarounds exist. This is the default when something is clearly wrong but not dangerous.
- Configuration option documented but silently ignored at runtime
- Version resolution returns a valid but incorrect version
- Error messages reference outdated configuration keys
- CLI command exits 0 on failure

### Minor
Improvement suggestions, cosmetic problems, or trivial documentation errors. Do not inflate to moderate.
- Typo in a non-critical error message
- CLI help string slightly misleading but command works correctly
- README example uses deprecated but still-functional syntax

---

## Duplicate Detection

**Keyword search:** Search the target repo's issues using 2-3 keywords from the finding. Try the error message text (exact match), the affected component name, and the symptom description. Search both open and recently closed issues (last 90 days).

**Label filtering:** Filter by `bug` + component area label, `documentation` for doc drift, `security` for sandbox/permission issues.

**Closed issues and linked PRs:** If a fix was merged but the problem persists, reference the closed issue and note the regression. If a fix PR is open, skip drafting unless the finding adds new information.

**When uncertain:** Draft the issue anyway and flag the potential duplicate. Let the user decide.

---

## Examples

### Good Draft

```
**Target Repo:** openreef
**Severity:** moderate
**Duplicate Check:** None found

**Title:** Schema validation: agentToAgent allows references to undeclared agents

## Description
reef.json schema validation does not verify that agent names in `agentToAgent`
correspond to agents declared in `agents`. A formation passes `reef validate`
while containing routing rules to nonexistent agents.

## Evidence
- **File:** src/schema/validate.ts, lines 142-158
- **Observed behavior:** `reef validate` exits 0 when `agentToAgent.architect`
  includes `"nonexistent-agent"`, which is not in `agents`.
- **Reproduction steps:**
  1. Create reef.json with `"agents": { "architect": { ... } }`
  2. Add `"agentToAgent": { "architect": ["nonexistent-agent"] }`
  3. Run `reef validate`
  4. Observe: exits 0, no warnings

## Expected Behavior
`reef validate` should fail, indicating `nonexistent-agent` is not declared.

## Found During
reef-forge QA review of weather-alerts v0.1.0

## Severity Assessment
Moderate. Passes validation but fails at runtime. Not a security issue, but
defeats the purpose of validation.
```

### Bad Draft (with corrections)

```
**Title:** Validation is broken
**Severity:** critical

## Description
The validation doesn't work properly. It lets bad configurations through.

## Evidence
I tried some invalid configs and they passed.

## Expected Behavior
It should catch errors.
```

**Problems:**
- **Vague title.** Which validation? What is broken? Prefix with area and describe the specific failure.
- **No evidence.** Which configs? What did they contain? What file is responsible? Include paths and line numbers.
- **No reproduction steps.** A maintainer cannot act without follow-up questions.
- **Inflated severity.** "Critical because validation is important" is circular. State what actually breaks downstream.
- **Missing "Found During."** Which formation, which version? Maintainers need this to prioritize.

Every field exists so the maintainer never has to ask "what do you mean?" If they would need to ask, the draft is not ready.
