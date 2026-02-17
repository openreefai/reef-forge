# Formation Patterns

Reference guide for topology, SOUL structure, variable conventions, scheduling, knowledge organization, README structure, and binding patterns. Use this when writing formation specs in Phase 3.

---

## Topology Patterns

### Hub-and-Spoke

```
specialist-a ──▶ coordinator ◀── specialist-b
specialist-c ──▶ coordinator ◀── specialist-d
```

**When to use:** One agent orchestrates all others. Specialists do not talk to each other. All information flows through the hub. This is the default pattern for most formations.

**Strengths:** Clear accountability, simple routing, easy to reason about. The coordinator has full context at all times.

**Weaknesses:** The hub is a bottleneck. If the coordinator is slow or overwhelmed, the entire formation stalls.

**Examples:** daily-ops (Chief of Staff coordinates four specialists), most customer-support formations.

**reef.json shape:**
```json
"agentToAgent": {
  "coordinator": ["specialist-a", "specialist-b"],
  "specialist-a": ["coordinator"],
  "specialist-b": ["coordinator"]
}
```

### Mesh

```
agent-a ⇄ agent-b
agent-a ⇄ agent-c
agent-b ⇄ agent-c
```

**When to use:** Agents need to collaborate directly without a central coordinator. Each agent can initiate work with any other agent. Rare in practice -- most formations benefit from a coordinator.

**Strengths:** Low latency between agents. No single point of failure.

**Weaknesses:** Hard to track state. No single agent has full context. Debugging is difficult. Risk of circular delegation.

**Examples:** Peer-review groups where any agent can request review from any other.

**reef.json shape:**
```json
"agentToAgent": {
  "agent-a": ["agent-b", "agent-c"],
  "agent-b": ["agent-a", "agent-c"],
  "agent-c": ["agent-a", "agent-b"]
}
```

### Mixed (Hub with Lateral Edges)

```
coordinator ──▶ specialist-a ──▶ specialist-b ──▶ qa
     ▲                                              │
     └──────────────────────────────────────────────┘
```

**When to use:** Mostly hub-and-spoke, but some specialists need to hand off directly to each other without routing through the hub. reef-forge uses this: Soul Writer passes artifacts to Builder directly, and Builder submits to QA directly.

**Strengths:** Combines coordinator oversight with efficient lateral handoffs. Reduces unnecessary round-trips through the hub.

**Weaknesses:** More edges to reason about. Must be clear about which messages go through the hub and which go laterally.

**Examples:** reef-forge (Architect as hub, but soul-writer passes to builder, builder submits to qa).

**reef.json shape:**
```json
"agentToAgent": {
  "coordinator": ["specialist-a", "specialist-b", "qa"],
  "specialist-a": ["coordinator", "specialist-b"],
  "specialist-b": ["coordinator", "qa"],
  "qa": ["coordinator", "specialist-b"]
}
```

**Rule of thumb:** Start with hub-and-spoke. Add lateral edges only when routing through the hub would create unnecessary latency or lose context.

### Critical Topology Tool Requirements

**Every agent that is a key in `agentToAgent` MUST have `sessions_send` in its `tools.allow`.** Without it, the topology edge is declared but the agent cannot actually send messages at runtime. This is the single most common formation-breaking bug.

**Every agent SHOULD have `sessions_history` in its `tools.allow`.** Sessions reset daily and on idle timeout. Without `sessions_history`, agents lose all context on reset and cannot recover prior conversations or deliverables.

When specifying an agent in Phase 3, always verify:
- Does this agent send messages to other agents? → needs `sessions_send`
- Does this agent need to survive session resets? → needs `sessions_history` (the answer is always yes)

---

## SOUL.md Standard Structure

Every SOUL.md should contain these six sections. Not every agent needs all six to be equally detailed, but the structure should be consistent across a formation.

### 1. Identity and Role Statement
The opening paragraph. First-person voice. States who the agent is, what formation it belongs to (using `{{namespace}}`), and its core responsibility in one to three sentences. Sets the tone for everything that follows.

### 2. Core Behavior
The "Your Role" section. Describes the agent's primary workflow: what it does on each activation, how it processes inputs, what outputs it produces. Use numbered steps for sequential workflows, bullet points for parallel responsibilities.

### 3. Output Format
Templates and schemas for the agent's deliverables. If the agent produces structured output (research briefs, task boards, review findings), define the exact format here. Consistency across activations matters more than flexibility.

### 4. Delegation and Communication
Who this agent talks to, when, and how. Includes message formats for outbound delegation and expectations for inbound messages. References `{{tools}}` for sending and `{{namespace}}` for scoping.

### 5. Quality Standards
What "good" looks like for this agent's work. Specific, measurable criteria. "Cite all sources" is better than "be thorough." "Respond within one heartbeat cycle" is better than "be timely."

### 6. What You Never Do
Hard constraints. Things the agent must not do under any circumstances. Keep this list short and specific. Every item should prevent a real failure mode, not just sound cautious.

### 7. Session History (Required)
Tells the agent it has `sessions_history` and when to use it (session recovery only, not routine).

### 8. State Persistence (Required)
Defines 1-3 files in `knowledge/dynamic/` where the agent writes working state. Every agent needs this to survive session resets. Specify file names, contents, write triggers, and clear conditions.

When speccing a formation, include these two sections in every agent's spec for the Soul Writer. They are as critical as the 6 core sections — without them, the formation works on first boot but breaks on the first session reset.

---

## Variable Conventions

### Naming
- `UPPER_SNAKE_CASE` for all variable names
- Prefix with the scope when ambiguous: `EMAIL_PRIORITY_DOMAINS` not just `DOMAINS`
- Use descriptive names: `INTERACTION_CHANNEL` not `CHAN`

### Types
| Type | Use for | Examples |
|------|---------|---------|
| `string` | Text values, URLs, channel identifiers | `INTERACTION_CHANNEL`, `OPENREEF_REPO_URL` |
| `number` | Counts, limits, thresholds | `MAX_QA_ROUNDS`, `HEARTBEAT_INTERVAL` |
| `boolean` | Feature flags, toggles | `ENABLE_AUTO_BRIEFING` |

### Required vs. Optional
- Mark a variable `required: true` only if the formation cannot function without it
- Always provide a `default` for optional variables
- Document the default value in the description, not just in the schema

### Sensitive Flag
- Set `sensitive: true` for API keys, tokens, passwords, and any value that should not appear in logs or UI
- Never set defaults for sensitive variables
- Sensitive variables must be set via `.env` or secret manager, never committed to source

### .env.example
- List every variable with a comment explaining its purpose
- Show defaults inline for optional variables
- Leave required variables blank with a clear comment

---

## Cron Patterns

### Common Schedules

| Pattern | Meaning | Use for |
|---------|---------|---------|
| `*/5 * * * *` | Every 5 minutes | High-frequency polling (email, alerts) |
| `*/15 * * * *` | Every 15 minutes | Heartbeat checks, task board review |
| `0 * * * *` | Every hour, on the hour | Moderate-frequency aggregation |
| `0 7 * * *` | Daily at 7:00 AM | Morning briefings, daily reports |
| `0 8 * * *` | Daily at 8:00 AM | Content calendar checks, planning |
| `0 8 * * 1` | Mondays at 8:00 AM | Weekly reviews, planning cycles |

### Offset Scheduling
When multiple agents have cron schedules, offset them to avoid resource contention:
- **Bad:** Both agents at `0 7 * * *` -- they compete for resources at the same moment
- **Good:** Agent A at `0 7 * * *`, Agent B at `5 7 * * *` -- five-minute stagger

General rule: stagger by at least 5 minutes. For formations with 4+ cron agents, spread them across a 30-minute window.

### Timezone Handling
- All cron schedules should assume a single timezone per formation
- Document the timezone in the README's cron schedule table
- Use `America/New_York` as the conventional default unless the user specifies otherwise
- The timezone field in reef.json applies to all agents; per-agent timezones are not supported

---

## Knowledge Directory Patterns

### knowledge/static/
- **Committed to source control.** Ships with the formation.
- Contains reference material that does not change at runtime: runbooks, checklists, templates, pattern guides, domain glossaries.
- Files are read-only during execution. Agents reference them but never modify them.
- Use markdown format for all static knowledge files.

### knowledge/dynamic/
- **Generated at runtime.** Lives in the OpenClaw workspace, not in the source tree.
- Contains state that accumulates over time: task boards, context logs, conversation history summaries, learned preferences.
- Agents read and write these files during execution.
- The `.gitkeep` in source ensures the directory structure exists; actual contents are workspace-local.
- **Destroyed on `reef uninstall`.** Warn users to back up dynamic knowledge before teardown.

### Naming Conventions
- Use kebab-case for filenames: `ops-runbook.md`, `task-board.md`, `formation-patterns.md`
- Use descriptive names that indicate content, not function: `api-reference.md` not `data.md`
- One topic per file. Do not combine unrelated reference material into a single file.

---

## README Section Structure

A formation README should follow this order. Every section is required unless marked optional.

1. **Title and tagline** -- Formation name as H1, one-line description as blockquote
2. **What It Does** -- Plain-language explanation of the formation's purpose. Describe the loops or workflows, not the implementation.
3. **Requirements** -- Runtime dependencies: OpenClaw version, CLI tools, API keys, external services
4. **Variables** -- Table with columns: Variable, Type, Default, Required, Description
5. **Quick Start** -- Minimal steps to install and run. Copy-pasteable commands.
6. **Agents** -- One subsection per agent: model, role, cron (if any), and a paragraph describing behavior
7. **Topology** -- ASCII diagram plus explicit edge table (From/To)
8. **Cron Schedule** -- Table with Agent, Schedule, Description columns. Note the timezone.
9. **How It Works** *(optional for solo formations)* -- Detailed walkthrough of each workflow loop
10. **Channel Bindings** -- Table of channel-to-agent mappings. Explain the match object format and channel tokens.
11. **Teardown** -- Uninstall command plus warnings about dynamic data loss. List workspace paths.

---

## Binding Patterns

### Functional Bindings (Hardcoded)
Bindings where the channel is a fixed, known value in the reef.json. Used when the formation always connects to the same channel regardless of who installs it.

```json
{ "channel": "internal:build-notifications", "agent": "builder" }
```

- The channel value is a literal string, not a variable reference.
- Use for internal formation plumbing where the channel is an implementation detail, not a user choice.

### Interaction Bindings (Variable)
Bindings where the channel is set by the user via a variable. Used for the primary user-facing channel.

```json
{ "channel": "{{INTERACTION_CHANNEL}}", "agent": "architect" }
```

- The channel value is a `{{VARIABLE}}` reference resolved at deploy time.
- The variable must be defined in the `variables` section of reef.json.
- Use channel tokens for interaction channels: `slack`, `telegram`, `teams`, `discord`.
- Use match objects with optional peer targeting for specific destinations.

### Binding Format

Bindings use a match object with a required `channel` token and optional fields:

```json
{ "match": { "channel": "slack", "peer": { "kind": "channel", "id": "#ops" } }, "agent": "architect" }
```

- **channel** -- The platform token: `slack`, `telegram`, `teams`, `discord`
- **peer.kind** -- Target type: `direct`, `group`, `channel`
- **peer.id** -- Target identifier: channel name, chat ID, room name
- **accountId**, **guildId**, **teamId**, **roles** -- Additional routing fields

Use `{{VARIABLE}}` interpolation for user-configurable values. Channel token goes in `INTERACTION_CHANNEL`, peer targeting in `INTERACTION_PEER_KIND` and `INTERACTION_PEER_ID`.

This binds to an internal channel named `build-status` that exists only within the formation's runtime. It is not routable from any external platform. Use bare channels only for internal coordination that the user never needs to see.

### Rules
- Every formation must have at least one interaction binding (user-facing).
- Only one agent should be bound to the primary interaction channel. That agent is the single point of contact.
- Additional channels beyond the manifest binding are configured through the bootstrap/reconfigure flow and stored in `knowledge/dynamic/`.
- Never bind multiple agents to the same user-facing channel. Route through a coordinator instead.
