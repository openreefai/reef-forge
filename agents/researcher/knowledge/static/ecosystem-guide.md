# Ecosystem Guide

Reference material for investigating the OpenReef ecosystem. Use this guide to know where to look, what to look for, and how the pieces connect.

---

## OpenReef Repository

**Clone URL:** `{{OPENREEF_REPO_URL}}`
**What it is:** The core framework that defines what a formation is, how it validates, and how the CLI operates.

### Key Files and Directories

| Path | Purpose |
|------|---------|
| `schema/reef.schema.json` | JSON Schema for reef.json. This is the source of truth for what fields are valid, required, and typed. Check here first when investigating whether a formation config option exists. |
| `schema/reef.lock.schema.json` | JSON Schema for reef.lock.json. Defines the lockfile contract for deterministic resolution. |
| `spec/SPEC.md` | The OpenReef specification. Defines formation structure, agent contracts, communication rules, variable interpolation, and validation requirements. |
| `spec/agent-contract.md` | Agent-level spec: required files (IDENTITY.md, SOUL.md), knowledge directory structure, tool declarations, and sandbox rules. |
| `spec/variables.md` | Variable interpolation spec. Defines `{{VAR}}` syntax, resolution order, sensitive vs. non-sensitive, and default handling. |
| `cli/` | CLI source code. Look here for the implementation of `reef validate`, `reef init`, `reef deploy`, and other commands. |
| `cli/commands/validate.js` | The validation command. Read this to understand exactly which checks run during `reef validate` and what error codes mean. |
| `cli/commands/init.js` | The scaffolding command. Useful for understanding the default file tree a new formation starts with. |
| `docs/` | Internal documentation. May contain ADRs (Architecture Decision Records) that explain why certain design choices were made. |
| `CHANGELOG.md` | Release history. Check this to understand what changed between versions and whether recent changes affect the formation being built. |
| `package.json` or `pyproject.toml` | Dependency manifest. Check the version number here -- it is the authoritative framework version. |

### What to Investigate

- Current spec version and any recent breaking changes
- Supported agent roles and whether custom roles are allowed
- Variable interpolation rules -- what syntax is supported, what happens with missing variables
- Validation rules -- what `reef validate` checks and what it ignores
- agentToAgent communication constraints -- are there topology restrictions?
- Sandbox model definitions -- what `filesystem` and `network` options exist

---

## Tide Repository

**Clone URL:** `{{TIDE_REPO_URL}}`
**What it is:** The public registry of published formations. Think of it as npm for formations.

### Key Files and Directories

| Path | Purpose |
|------|---------|
| `registry/` | Published formation metadata. Each formation has a directory or manifest entry. |
| `registry/index.json` | Registry index. Lists all published formations with name, version, author, and category. Start here for any registry-wide query. |
| `schema/registry-entry.schema.json` | Schema for registry entries. Defines required metadata fields for publishing. |
| `categories/` | Category definitions. Lists recognized formation categories and their descriptions. |
| `docs/publishing.md` | Publishing guide. Documents the workflow for submitting a formation to Tide. |
| `docs/naming.md` | Naming conventions. Rules for formation names, namespaces, and versioning. |
| `CHANGELOG.md` | Registry changes. Useful for spotting recently added formations or category changes. |
| `scripts/validate-entry.js` | Entry validation. The checks that run when a formation is submitted to Tide. |

### What to Investigate

- How many formations exist in each category -- is the target category saturated?
- Naming conventions -- what patterns do successful formations follow?
- Version ranges -- what semver patterns are standard in the registry?
- Formation sizes -- how many agents do published formations typically have?
- Recent publishing activity -- is the registry actively growing?
- Required metadata for publishing -- what fields does the formation need before it can be listed?

---

## OpenClaw Repository

**Clone URL:** `{{OPENCLAW_REPO_URL}}`
**What it is:** The agent runtime. This is what actually executes formations -- the engine that runs agents, manages sandboxes, routes messages, and enforces tool permissions.

### Key Files and Directories

| Path | Purpose |
|------|---------|
| `runtime/` | Core runtime source. Agent lifecycle, message routing, tool execution. |
| `runtime/sandbox/` | Sandbox implementation. How filesystem and network restrictions are enforced. |
| `runtime/tools/` | Built-in tool definitions. Each tool has a manifest describing its capabilities, parameters, and permissions. |
| `runtime/tools/registry.json` | Tool registry. Authoritative list of all built-in tools and their identifiers. |
| `runtime/models/` | Model provider integrations. Which LLM providers are supported and how model strings are resolved. |
| `runtime/sessions/` | Session management. Implementation of `sessions_send` and agent-to-agent communication. |
| `docs/tool-authoring.md` | Tool authoring guide. How custom tools and ClawHub skills are loaded into the runtime. |
| `docs/sandbox-model.md` | Sandbox documentation. What each sandbox level permits and restricts. |
| `docs/model-compatibility.md` | Model compatibility matrix. Which models work with which features. |
| `CHANGELOG.md` | Runtime release history. Critical for spotting recent capability changes. |
| `package.json` or `pyproject.toml` | Runtime version. Cross-reference with reef.json `compatibility.openclaw` field. |

### What to Investigate

- Supported tool types and their exact identifiers (must match reef.json `tools.allow` values)
- Sandbox levels -- what does `filesystem: "restricted"` actually restrict?
- `sessions_send` contract -- message size limits, delivery guarantees, error handling
- Model string format -- how `anthropic/claude-opus-4-6` and `openai/gpt-5.3-codex` are resolved
- Runtime version compatibility -- does the current runtime satisfy `>=0.5.0`?
- Known limitations or open bugs that affect formation execution

---

## ClawHub Skill Discovery

**URL:** https://clawhub.dev
**What it is:** The skill marketplace. Published skills that can be wired into formations as tool dependencies.

### Discovery Workflow

1. **Search by keyword.** Start at `https://clawhub.dev/search?q={keyword}` or use the web-search tool to find skills by capability.
2. **Check the skill page.** Each skill page shows: name, identifier, version, description, required permissions, compatibility range, install count, and last updated date.
3. **Verify compatibility.** The skill must be compatible with the OpenClaw version specified in the formation's `compatibility.openclaw` field. Look for the `openclaw` compatibility range on the skill page.
4. **Check maintenance status.** A skill last updated more than 6 months ago is a risk. Note this in your brief.
5. **Record the exact identifier.** Builder needs the precise skill identifier and version range to wire it into reef.json dependencies. Use the format shown on the skill page (e.g., `@clawhub/skill-name@^1.2.0`).
6. **Look for alternatives.** If the first skill you find has low install counts or compatibility concerns, search for alternatives before recommending it.

### What to Report About Skills

For each relevant skill, include in your brief:
- Skill identifier and recommended version range
- What it does (one sentence)
- Compatibility range
- Install count and last update date
- Any permissions it requires
- Whether alternatives exist

---

## Spec Compliance Checklist

When assessing whether a formation design will pass `reef validate`, check these requirements against the current SPEC:

1. **reef.json structure** -- all required top-level fields present and correctly typed per `reef.schema.json`
2. **Agent source paths** -- every `agents.*.source` path points to a directory that exists and contains IDENTITY.md
3. **IDENTITY.md presence** -- every agent directory has one. It must contain Name, Role, Formation, and Description.
4. **SOUL.md presence** -- every agent directory should have one (check SPEC for whether this is required or recommended)
5. **Variable declarations** -- every `{{VAR}}` used in agent files has a corresponding entry in `reef.json.variables`
6. **Tool permissions** -- every tool in `tools.allow` is a recognized tool identifier in the runtime registry
7. **agentToAgent topology** -- communication edges are declared and do not reference nonexistent agents
8. **Binding validity** -- at least one binding exists and references a valid agent
9. **Compatibility range** -- `compatibility.openclaw` is a valid semver range that the current runtime satisfies
10. **Knowledge directory structure** -- if present, follows the `knowledge/static/` and `knowledge/dynamic/` convention

---

## Ecosystem Integration Points

These are the critical seams where the three repos and ClawHub connect:

| Integration Point | What Connects | Where to Verify |
|---|---|---|
| reef.json schema | OpenReef schema <-> formation config | `openreef/schema/reef.schema.json` |
| Tool identifiers | OpenClaw tool registry <-> reef.json `tools.allow` | `openclaw/runtime/tools/registry.json` |
| Model strings | OpenClaw model providers <-> reef.json agent `model` | `openclaw/runtime/models/` |
| Sandbox levels | OpenClaw sandbox impl <-> reef.json agent `sandbox` | `openclaw/runtime/sandbox/` |
| ClawHub skills | ClawHub registry <-> reef.json `dependencies` | ClawHub skill page + `openclaw/docs/tool-authoring.md` |
| Tide publishing | Tide registry schema <-> reef.json metadata | `tide/schema/registry-entry.schema.json` |
| Variable interpolation | OpenReef spec <-> `{{VAR}}` usage in agent files | `openreef/spec/variables.md` |
| Agent contract | OpenReef spec <-> agent directory contents | `openreef/spec/agent-contract.md` |
| Version compatibility | reef.json `compatibility` <-> OpenClaw runtime version | `openclaw/package.json` or `pyproject.toml` |

---

## Common Patterns Across Published Formations

These patterns appear repeatedly in high-quality formations on Tide. Use them as baselines:

### Coordinator Pattern
One agent acts as the single point of contact and routes work to specialists. Most formations with 3+ agents use this. The coordinator is the only agent bound to an external channel.

### Research-Before-Build Pattern
A dedicated research agent investigates before the builder starts. Prevents wasted work by surfacing constraints early. Common in formations that interact with external APIs or ecosystems.

### Adversarial QA Pattern
A reviewer agent (often on a different model vendor) reviews output before it reaches the user. The reviewer and builder iterate for a bounded number of rounds before escalating.

### Knowledge Split Pattern
Static knowledge (reference material, API docs, schemas) goes in `knowledge/static/`. Dynamic knowledge (state, poll results, cached findings) goes in `knowledge/dynamic/`. Agents never write to static; they always write to dynamic.

### Minimal Tool Permissions
Agents only declare the tools they actually need. A research agent does not get `file-write`. A builder does not get `git-clone`. This follows the principle of least privilege and matches how the OpenClaw sandbox enforces restrictions.

### Variable Isolation
Sensitive variables (API keys, tokens) are marked `sensitive: true` in reef.json. Non-sensitive variables (usernames, channel names, URLs) are marked `sensitive: false`. This affects how the runtime handles interpolation and logging.

### Single External Binding
Most formations bind exactly one agent to the external channel. All user interaction flows through that single agent, which coordinates internally. This simplifies the user experience and prevents conflicting responses.
