# Formation Quality Checklist

This is the authoritative checklist for QA review. Every item must be checked on every Round 1 review. Items are numbered by category for citation in reports (e.g., A-03, B-07).

---

## A. Manifest (reef.json) -- 14 items

| # | Item | PASS criteria | Evidence to cite |
|---|------|---------------|------------------|
| A-01 | Schema compliance | reef.json validates against `reef-schema.json` with zero errors | Validation output or specific violation |
| A-02 | Required fields present | `reef`, `type`, `name`, `version`, `description`, `namespace`, `agents` all present | Missing field name |
| A-03 | Type correct for agent count | `solo`=1 agent, `shoal`=2-5 agents, `school`=6+ agents | Actual agent count vs. declared type |
| A-04 | Topology valid | Every slug in `agentToAgent` values exists as a key in `agents` | Broken slug reference |
| A-05 | Variable declarations complete | Every `{{VAR}}` used anywhere in the formation files is declared in `variables` | File path, line, and undeclared variable name |
| A-06 | Bindings format correct | Each binding uses a match object with a `channel` token or `{{VARIABLE}}` interpolation | Malformed binding value |
| A-07 | Cron expressions valid | Every `schedule` value is a valid 5-field cron expression | Invalid expression and what is wrong with it |
| A-08 | Dependencies declared | Every external service the formation requires is listed in `dependencies.services` | Undeclared service (e.g., model provider not listed) |
| A-09 | Validation config present | `validation` object exists with at least `agent_exists` and `file_exists` | Missing validation object or fields |
| A-10 | License field set | `license` contains a valid SPDX identifier | Missing or invalid license value |
| A-11 | Author field set | `author` is a non-empty string | Missing or empty author |
| A-12 | Version follows semver | `version` matches pattern `^\d+\.\d+\.\d+(-[a-zA-Z0-9.]+)?(\+[a-zA-Z0-9.]+)?$` | Actual version string |
| A-13 | Description meaningful | `description` is not generic placeholder text (e.g., "A formation", "TODO", "Description here") | The placeholder text found |
| A-14 | Namespace set | `namespace` is present, matches `^[a-z][a-z0-9-]*$`, and aligns with formation name | Missing or non-conforming namespace |

---

## B. SOUL.md Quality -- 10 items

| # | Item | PASS criteria | Evidence to cite |
|---|------|---------------|------------------|
| B-01 | All 6 sections present | Identity/role, core behavior, output format, communication protocol, boundaries, "What You Never Do" | Missing section name |
| B-02 | Identity consistent with reef.json | Agent name and role description in SOUL.md matches `agents[slug].description` in reef.json | Conflicting text from both files |
| B-03 | Behavioral rules specific | Rules are testable ("Check for new emails every 5 minutes") not vague ("Try to be helpful") | Vague rule text |
| B-04 | Output formats structured | At least one output template with field names, not just prose descriptions | Section lacking template |
| B-05 | Communication protocol matches topology | Agents mentioned in SOUL.md communication section match `agentToAgent` in reef.json | Agent referenced in SOUL.md but missing from topology (or vice versa) |
| B-06 | Tool usage matches reef.json | Tools referenced in SOUL.md behavioral rules appear in `agents[slug].tools.allow` | Tool name in SOUL.md not in tools.allow |
| B-07 | Boundaries section comprehensive | Explicit limits on what the agent will not do, covering at minimum: external communication, data modification, scope creep | Missing boundary category |
| B-08 | No AI-obvious language | Does not contain phrases like "I strive to", "I aim to", "delve", "leverage", "It's important to note" | Phrase and line number |
| B-09 | Variable interpolation correct | `{{VAR}}` placeholders match declared variable names exactly (case-sensitive) | Mismatched variable reference |
| B-10 | Personality distinct | Agent voice is distinguishable from other agents in the formation; not a generic template | Comparison showing similarity to another agent's SOUL.md |

---

## C. IDENTITY.md -- 3 items

| # | Item | PASS criteria | Evidence to cite |
|---|------|---------------|------------------|
| C-01 | File exists for every agent | Each agent slug in reef.json has `agents/<slug>/IDENTITY.md` | Missing file path |
| C-02 | All 4 fields present | Name, Role, Formation, Description fields all present | Missing field name |
| C-03 | Formation uses namespace variable | Formation field value is `{{namespace}}` (not hardcoded) | Actual value found |

---

## D. Knowledge/Static Files -- 4 items

| # | Item | PASS criteria | Evidence to cite |
|---|------|---------------|------------------|
| D-01 | Reference material relevant | Static files relate to the agent's declared role and responsibilities | Irrelevant file name and agent role |
| D-02 | No stale or placeholder data | Files contain real content, not "TODO", "placeholder", or sample data that does not apply | Placeholder text found |
| D-03 | Organized by topic | Each file covers one concern; not a single monolithic dump | File that mixes unrelated topics |
| D-04 | Provides needed context | Agent has the reference material it needs to perform its SOUL.md responsibilities | Responsibility from SOUL.md lacking corresponding knowledge file |

---

## E. README.md -- 9 items

| # | Item | PASS criteria | Evidence to cite |
|---|------|---------------|------------------|
| E-01 | Title with tagline | H1 heading with a one-line description of the formation | Missing or generic title |
| E-02 | What it does section | Explains the formation's purpose in 2-4 sentences | Missing section |
| E-03 | Requirements section | Lists OpenClaw version, API keys, and external services needed | Missing section or incomplete list |
| E-04 | Variables table | Table with columns: variable name, type, required, default, description | Missing table or missing column |
| E-05 | Quick-start instructions | Step-by-step install and configure commands | Missing section |
| E-06 | Agents section | Lists every agent with name, role, and one-line description | Missing agent or missing fields |
| E-07 | Topology diagram | Text-based diagram showing agent communication paths | Missing diagram |
| E-08 | How it works narrative | Explains the flow: what happens when the formation runs | Missing section |
| E-09 | Teardown instructions | How to stop, remove, or uninstall the formation | Missing section |

---

## F. File Structure -- 4 items

| # | Item | PASS criteria | Evidence to cite |
|---|------|---------------|------------------|
| F-01 | Agent directories exist | Every agent slug in reef.json has a corresponding `agents/<slug>/` directory | Missing directory |
| F-02 | Knowledge subdirectories exist | Each agent directory contains `knowledge/static/` and `knowledge/dynamic/` | Missing subdirectory path |
| F-03 | .env.example matches variables | `.env.example` lists every variable from reef.json `variables` with comments | Missing variable or file |
| F-04 | .gitignore present | `.gitignore` exists and covers `.env`, `knowledge/dynamic/`, and runtime artifacts | Missing file or missing pattern |

---

## G. Ecosystem Compatibility -- 4 items

| # | Item | PASS criteria | Evidence to cite |
|---|------|---------------|------------------|
| G-01 | Schema validation passes | reef.json validates against the latest `reef.schema.json` from the openreef repo | Validation errors |
| G-02 | Clean install | Formation installs without errors using `reef install` in a clean OpenClaw environment | Install error output |
| G-03 | Variable interpolation resolves | No orphaned `{{VAR}}` references -- every interpolation maps to a declared variable | Orphaned reference, file, and line |
| G-04 | Cron expressions parse | Every cron `schedule` value parses as valid 5-field cron (minute hour day month weekday) | Invalid expression and parse error |

---

## Defect vs. Preference Classification Guide

### DEFECT -- blocks the verdict

A finding is a DEFECT if fixing it is **required for the formation to function correctly or safely**.

| Example | Why it is a DEFECT |
|---------|-------------------|
| reef.json missing `namespace` field | Schema validation fails. Required field. |
| `agentToAgent` references slug `monitor` but no agent `monitor` exists | Runtime will fail to route messages. |
| `type: "shoal"` but 7 agents declared | Schema constraint violated. |
| Variable `{{API_KEY}}` used in SOUL.md but not declared in `variables` | Interpolation will fail at deploy. |
| Agent `builder` SOUL.md says it uses `shell` tool but `tools.allow` lists only `file-read` | Agent will be denied tool access at runtime. |
| IDENTITY.md missing for agent `qa` | Required file missing from agent directory. |
| Cron expression `*/5 * * *` (4 fields) | Invalid cron -- will not parse. |
| `sensitive: false` on a variable named `API_SECRET` that holds credentials | Security issue -- secret stored in plaintext. |

### PREFERENCE -- noted, never blocks

A finding is a PREFERENCE if the formation **functions correctly** but the reviewer would have done it differently.

| Example | Why it is a PREFERENCE |
|---------|----------------------|
| Agent named `content-creator` -- reviewer prefers `copywriter` | Naming style. Both are valid slugs. |
| SOUL.md sections in non-standard order | Ordering choice. All sections present. |
| README uses bullet points instead of table for agents | Formatting preference. Information is present. |
| SOUL.md says "I aim to provide accurate research" | Stylistically weak but not a functional issue. (Cite as B-08 preference.) |
| Description says "A team of agents" -- reviewer prefers more detail | Subjective. Description exists and is not a placeholder. |
| Knowledge files in a single file instead of split by topic | Organization preference. Content is present. |

### Decision Rule

If you cannot decide: **classify as PREFERENCE**. It is worse to block a working formation than to let a style issue through. You can always flag it as a warning.

---

## Review Report Template

Copy this template for every review report:

```
# QA Review -- Round [N]

**Formation:** [name from reef.json]
**Version:** [version from reef.json]
**Verdict:** PASS | FAIL
**Defects:** [count]
**Preferences:** [count]

## Defects

[Numbered list. Each entry includes the checklist item code, title, explanation, and file/line citation.]

1. **[X-##] Title** -- Explanation. File: `path/to/file`, line ##.

[If zero defects, write: "None."]

## Preferences

[Numbered list. Each entry includes the checklist item code, description, and "Non-blocking" label.]

1. **[X-##]** Description. Non-blocking.

[If zero preferences, write: "None."]

## Warnings

[Observations that are not checklist items but worth noting. Techincally valid but suspicious.]

- Warning text here.

[If no warnings, write: "None."]

## Ecosystem Findings

[Results of the ecosystem audit: schema compatibility, CLI version, runtime support, upcoming breaking changes.]

- Finding text here.

[If no findings, write: "Ecosystem audit clean. No issues found."]

## Scorecard

| Category | Result |
|----------|--------|
| A. Manifest | PASS / FAIL |
| B. SOUL.md Quality | PASS / FAIL |
| C. IDENTITY.md | PASS / FAIL |
| D. Knowledge Files | PASS / FAIL |
| E. README.md | PASS / FAIL |
| F. File Structure | PASS / FAIL |
| G. Ecosystem | PASS / FAIL |
```

**Verdict rules:**
- A category is FAIL if it contains one or more DEFECTS.
- A category with only PREFERENCES is PASS.
- The overall verdict is FAIL if any category is FAIL.
- The overall verdict is PASS only if all categories are PASS.

---

## Escalation Report Template

Use this template when defects persist after the maximum number of review rounds:

```
# Escalation Report -- [Formation Name]

**Rounds completed:** [N]
**Unresolved defects:** [count]

## Unresolved Defects

### 1. [Defect title] ([X-##])

- **QA finding (Round 1):** [Original citation with file and line]
- **Builder response (Round 2):** [What they changed and why]
- **QA rebuttal (Round 3):** [Why the fix is insufficient, with updated citation]
- **Current state:** [What the file looks like now -- quote the relevant lines]
- **Recommended resolution:** [QA's recommendation for how to resolve]

### 2. ...

## Resolved Defects (for context)

[List of defects that were successfully fixed, showing the round in which they were resolved.]

- **[X-##] Title** -- Resolved in Round [N].

## QA Recommendation

[One paragraph: Is this formation close to shipping (1-2 minor issues remain) or fundamentally broken (structural problems persist)? What does QA recommend the user decide?]
```

Send the escalation report to the Architect. The Architect presents it to the user for final decision.
