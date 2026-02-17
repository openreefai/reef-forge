# reef-forge

> A 5-agent shoal that designs, builds, and adversarially reviews OpenReef formations.

You describe the agent system you want. reef-forge researches the ecosystem, writes SOUL files, scaffolds the complete formation, runs adversarial QA across vendor models, and delivers a production-ready formation back to you. If QA finds ecosystem bugs along the way, those findings are included in the delivery summary for your awareness.

## What It Does

reef-forge runs a six-phase build pipeline:

1. **Understand** -- Architect gathers your requirements through conversation: what the formation does, how many agents, what services it connects to, and how you want to interact with it.
2. **Research** -- Researcher clones the openreef, tide, and openclaw repos, browses ClawHub for existing skills, and produces a structured brief covering prior art, ecosystem state, and available components.
3. **Specify** -- Architect writes a formation spec capturing agents, topology, variables, bindings, and acceptance criteria. You approve before building begins.
4. **Build** -- Soul Writer authors SOUL.md, IDENTITY.md, and knowledge/static files for every agent. Builder scaffolds reef.json, README, .env.example, .gitignore, and the full directory tree.
5. **Review** -- QA (running on a different vendor model) performs adversarial review against a 48-item quality checklist and an ecosystem audit. Builder and QA iterate for up to `MAX_QA_ROUNDS` rounds until all defects are resolved or disputes are escalated.
6. **Deliver** -- Architect presents the finished formation with a summary of what was built, how QA findings were resolved, and any ecosystem issues surfaced during QA.

All output is reviewed by a human before anything is published. The formation uses a multi-vendor model strategy: four agents run on Anthropic models while the QA agent runs on OpenAI gpt-5.2, ensuring the reviewer brings genuinely independent judgment rather than shared model biases.

## Requirements

- [OpenClaw](https://github.com/openclaw/openclaw) >= 0.5.0 installed and running
- [OpenReef CLI](https://github.com/openreefai/openreef) >= 0.9.0 installed
- Anthropic API key (for Architect, Researcher, Soul Writer, Builder)
- OpenAI API key (for QA agent -- gpt-5.2)
- GitHub access token (optional -- for private repo cloning or to avoid rate limits)

## Variables

| Variable | Type | Default | Required | Description |
|----------|------|---------|----------|-------------|
| `INTERACTION_CHANNEL` | string | - | yes | Channel token for the primary contact channel (e.g. `slack`, `telegram`, `discord`, `teams`) |
| `OPENREEF_REPO_URL` | string | `https://github.com/openreefai/openreef` | no | URL of the openreef repository for Researcher and QA to clone |
| `TIDE_REPO_URL` | string | `https://github.com/openreefai/tide` | no | URL of the tide registry repository for Researcher and QA to clone |
| `OPENCLAW_REPO_URL` | string | `https://github.com/openclaw/openclaw` | no | URL of the openclaw runtime repository for Researcher and QA to clone |
| `MAX_QA_ROUNDS` | number | `4` | no | Maximum adversarial QA rounds before escalating unresolved disputes to user |

## Quick Start

```bash
# 1. Copy and fill in your environment variables
cp .env.example .env
# Edit .env: set INTERACTION_CHANNEL at minimum

# 2. Deploy the formation
reef install .
```

Once installed, talk to the Architect via your configured `INTERACTION_CHANNEL`. Describe the formation you want to build and the Architect takes it from there.

The repo URL variables default to the official openreef, tide, and openclaw repositories. Override them only if you are working with private mirrors.

## Agents

### Architect

**Model:** `anthropic/claude-opus-4-6` | **Role:** Coordinator

Single point of contact for the entire formation. The Architect decomposes formation requests into structured specs, defines acceptance criteria, routes work to the four specialist agents, and mediates QA disputes. It orchestrates the six-phase workflow end to end: gathering requirements, delegating research, writing the formation spec, coordinating the build, monitoring adversarial review, and delivering the final result. The Architect never writes SOULs, never scaffolds files, and never skips QA.

### Researcher

**Model:** `anthropic/claude-opus-4-6` | **Role:** Researcher

Deep domain research before building begins. The Researcher clones the openreef, tide, and openclaw repos read-only, browses ClawHub for existing skills, and produces structured briefs with an executive summary, key findings, sources, and recommended actions. It activates on demand when the Architect or QA needs ecosystem context, prior-art review, or clarification on spec ambiguities. All repo access is strictly read-only -- the Researcher never modifies cloned code.

### Soul Writer

**Model:** `anthropic/claude-opus-4-6` | **Role:** Writer

Writes the behavioral core of every agent in the target formation. The Soul Writer produces SOUL.md files with all six required sections (identity and role, behavioral rules, output formats, communication protocol, tool usage, and boundaries), IDENTITY.md identity cards, and knowledge/static reference material such as checklists, format specs, and domain glossaries. Each SOUL gets a distinct voice matched to the agent's role type -- coordinators sound different from researchers, reviewers different from writers. Hands off completed files to the Builder with a structured manifest including line counts and reef.json implications.

### Builder

**Model:** `anthropic/claude-opus-4-6` | **Role:** Builder

Scaffolds complete formation file trees from the Architect's spec. The Builder authors reef.json to schema, writes README.md and .env.example, creates agent directory structures with knowledge subdirectories, integrates SOUL.md files from the Soul Writer, wires ClawHub skills into dependencies, and runs `reef validate` before every QA submission. During the review phase, the Builder responds to QA defect findings with fixes and can dispute preference findings with evidence citing the spec, schema, or precedent formations.

### QA

**Model:** `openai/gpt-5.2` | **Role:** Reviewer

Adversarial reviewer running on a different vendor model (OpenAI instead of Anthropic) to provide cognitive diversity and independent judgment. QA reviews every formation against a 48-item quality checklist spanning seven categories: manifest compliance (14 items), SOUL.md quality (10 items), IDENTITY.md completeness (3 items), knowledge files (4 items), README structure (9 items), file structure (4 items), and ecosystem compatibility (4 items). Every finding is classified as either a blocking defect or a non-blocking preference. QA also clones ecosystem repos read-only to verify schema compatibility, runtime support, and upcoming breaking changes.

## Topology

```
                          +------------+
                          |  Architect |
                          +-----+------+
                         /  |   |   \
                        /   |   |    \
                       v    v   v     v
            researcher   soul   builder   qa
                         writer    |      |
                           |       |      |
                           +------>+      |
                                   |      |
                                   +----->+
                                          |
                              +-----------+-----------+
                              |           |           |
                              v           v           v
                           builder   soul-writer  researcher
```

**Pattern:** Mixed hub with lateral edges.

The Architect sits at the center, connected bidirectionally to all four specialists. Three sets of lateral edges allow direct collaboration during the Build and Review phases without routing every message through the hub.

**Explicit edge list:**

| From | To |
|------|----|
| architect | researcher, soul-writer, builder, qa |
| researcher | architect |
| soul-writer | architect, builder |
| builder | architect, qa |
| qa | architect, builder, soul-writer, researcher |

The lateral edges serve specific purposes:

- **soul-writer to builder** -- Soul Writer hands off completed SOUL.md and IDENTITY.md files directly to Builder for integration into the file tree.
- **builder to qa** -- Builder submits completed formations directly to QA for adversarial review.
- **qa to builder, soul-writer, researcher** -- QA sends defect findings back to Builder for fixes, flags SOUL quality patterns to Soul Writer, and requests ecosystem clarification from Researcher.

All user-facing communication flows exclusively through the Architect.

## How It Works

### Phase 1 -- Understand

You describe the formation you want to build through the Architect. The Architect asks clarifying questions until it has clear answers on every dimension:

- What problem does the formation solve?
- Who is the user (technical level, domain, how they will interact)?
- How many agents, and what does each one do?
- What external services or APIs are involved?
- What is the interaction model (single point of contact, multi-agent, autonomous)?
- Are there scheduled tasks (cron)?
- What is the interaction channel?

No building begins until requirements are unambiguous. Ambiguity at this stage becomes bugs downstream.

### Phase 2 -- Research

The Architect delegates to the Researcher with a structured research request covering four areas:

- **Ecosystem review** -- what exists in openreef, tide, and openclaw that is relevant to this formation?
- **ClawHub skill scan** -- are there published skills that cover part of the need, avoiding reinvention?
- **Domain research** -- what external APIs, protocols, or patterns will the formation depend on?
- **Prior art** -- are there existing formations in the Tide registry that solve a similar problem?

The Researcher clones relevant repos, investigates, and delivers a structured brief with an executive summary, key findings with file-path citations, sources, and recommended actions. The Architect waits for the complete brief before proceeding to specification.

### Phase 3 -- Specify

The Architect writes a formation spec that captures every decision in a structured format: formation type, namespace, description, agent definitions with roles and models, topology with edge purposes, variable declarations, channel bindings, dependency list, and acceptance criteria. This spec is the primary artifact -- downstream agents execute against it without ambiguity. The spec is shared with you for approval. No building begins until you sign off.

### Phase 4 -- Build

Two sequential delegations. First, the Soul Writer receives the approved spec and produces SOUL.md (with all six required sections), IDENTITY.md, and knowledge/static files for every agent in the target formation. Each agent gets a distinct voice matched to its role type. Second, the Builder receives the spec plus all SOUL artifacts and scaffolds the complete file tree: reef.json authored to schema, README.md, .env.example with all variables, .gitignore, reef.lock.json, and the full agents/ directory structure with knowledge subdirectories. The Builder integrates the Soul Writer's files, reconciles any variable or tool mismatches, runs `reef validate`, and submits to QA when validation passes clean.

### Phase 5 -- Review

QA runs the adversarial review protocol (see [Adversarial QA Protocol](#adversarial-qa-protocol) below). Rounds alternate between QA and Builder until all defects are resolved or `MAX_QA_ROUNDS` rounds elapse. The Architect monitors the exchange and mediates impasses. Ecosystem issues discovered during review are included in the QA report for user awareness.

### Phase 6 -- Deliver

The Architect presents the finished formation with four deliverables:

1. A summary of what was built and why
2. A record of how QA findings were resolved
3. Any ecosystem issues surfaced during QA (for your awareness)
4. Installation instructions for deploying the new formation

You review the deliverables and deploy the new formation with `reef install`.

## Adversarial QA Protocol

QA runs on a different vendor model (OpenAI gpt-5.2) for cognitive diversity -- a second opinion from a fundamentally different system.

The review uses a 48-item quality checklist across seven categories:

| Category | Items | Covers |
|----------|-------|--------|
| A. Manifest | 14 | Schema compliance, type/agent-count match, topology validity, variable declarations |
| B. SOUL.md Quality | 10 | Six required sections, testable rules, no AI-obvious language, distinct personality |
| C. IDENTITY.md | 3 | File exists per agent, all four fields present, namespace interpolation |
| D. Knowledge Files | 4 | Relevant material, no stale data, organized by topic |
| E. README.md | 9 | Title, variables table, topology diagram, quick start, teardown |
| F. File Structure | 4 | Agent directories, knowledge subdirectories, .env.example, .gitignore |
| G. Ecosystem | 4 | Schema validation, CLI compatibility, variable interpolation, cron parsing |

Every finding is classified as either a **defect** (blocking -- schema violations, broken references, missing files, security issues) or a **preference** (non-blocking -- wording choices, naming style, section ordering). Only defects block the verdict.

Rounds alternate between QA and Builder up to `MAX_QA_ROUNDS` (default: 4). If defects persist after the final round, QA produces an escalation report and the Architect presents it to you with a recommendation.

## Channel Bindings

| Channel | Agent |
|---------|-------|
| `{{INTERACTION_CHANNEL}}` | architect |

The binding is user-configurable via the `INTERACTION_CHANNEL` variable, which is the channel token (e.g. `slack`, `telegram`, `discord`, `teams`). Optional peer targeting can be configured with `INTERACTION_PEER_KIND` and `INTERACTION_PEER_ID` variables.

This single binding routes to the Architect, the only agent the user interacts with directly. All delegation and coordination happens behind the scenes. The four specialist agents have no external channel bindings -- they communicate exclusively through the inter-agent topology.

## Teardown

```bash
reef uninstall reef-forge/reef-forge
```

**WARNING:** `reef uninstall` destroys agent workspaces, including all `knowledge/dynamic/` contents. Research briefs, formation specs, QA review reports, and build artifacts are permanently deleted.

Runtime data lives in OpenClaw workspaces, **not** in the source tree:

```
$OPENCLAW_STATE_DIR/workspace-reef-forge-architect/knowledge/dynamic/
$OPENCLAW_STATE_DIR/workspace-reef-forge-researcher/knowledge/dynamic/
$OPENCLAW_STATE_DIR/workspace-reef-forge-soul-writer/knowledge/dynamic/
$OPENCLAW_STATE_DIR/workspace-reef-forge-builder/knowledge/dynamic/
$OPENCLAW_STATE_DIR/workspace-reef-forge-qa/knowledge/dynamic/
```

`$OPENCLAW_STATE_DIR` defaults to `~/.openclaw/` if not set.

**Before uninstalling,** copy any data you want to keep. Key data to preserve:

- **Builder workspace** -- completed formation file trees, reef.json drafts, validation logs
- **Researcher workspace** -- ecosystem research briefs
- **QA workspace** -- review reports and escalation reports
- **Architect workspace** -- formation specs, task history, escalation records

The source `formations/reef-forge/` directory is unaffected by uninstall.
