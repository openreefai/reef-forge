# reef.json Field Reference

Complete reference for the OpenReef formation manifest. Every formation is defined by a single `reef.json` at the repository root.

Schema version: `1.0`
Canonical schema: `reef-schema.json` (bundled in this knowledge directory)

---

## Top-Level Required Fields

Every reef.json must include all of these. Validation fails if any are missing.

| Field | Type | Description |
|-------|------|-------------|
| `reef` | string | Specification version. Must be `"1.0"`. |
| `type` | string | Formation category. One of: `solo`, `shoal`, `school`. |
| `name` | string | Formation name. Lowercase alphanumeric with hyphens. Pattern: `^[a-z][a-z0-9-]*$`. Max 128 chars. |
| `version` | string | Semantic version. Pattern: `^\d+\.\d+\.\d+(-[a-zA-Z0-9.]+)?(\+[a-zA-Z0-9.]+)?$`. |
| `description` | string | Brief description of what the formation does. Max 500 chars. |
| `namespace` | string | Prefix for all agent IDs. Lowercase alphanumeric with hyphens. Pattern: `^[a-z][a-z0-9-]*$`. Max 64 chars. |
| `agents` | object | Agent definitions keyed by slug. Must have at least one agent. |

## Top-Level Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `author` | string | Formation author or organization. |
| `license` | string | SPDX license identifier (e.g. `"MIT"`, `"Apache-2.0"`). |
| `repository` | string | URL of the formation's source repository. Must be a valid URI. |
| `compatibility` | object | Platform compatibility constraints. |
| `variables` | object | User-supplied configuration values. |
| `agentToAgent` | object | Communication topology as directed adjacency list. |
| `bindings` | array | Channel routing rules. |
| `cron` | array | Scheduled jobs with agent assignment. |
| `dependencies` | object | Required skills and services. |
| `validation` | object | Post-deploy health check configuration. |

---

## Formation Type Thresholds

The `type` field must match the number of agents defined in the `agents` object.

| Type | Agent Count | Description |
|------|-------------|-------------|
| `solo` | Exactly 1 | Single-agent formation. One agent handles everything. |
| `shoal` | 2 to 5 | Small multi-agent team. Typical hub-and-spoke or small pipeline. |
| `school` | 6 or more | Large multi-agent formation. Complex topologies, specialized roles. |

Validation will fail if the type does not match the agent count. Count the agents in your `agents` object and set the type accordingly.

---

## Compatibility

```json
"compatibility": {
  "openclaw": ">=0.5.0"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `openclaw` | string | Required OpenClaw version range. Uses semver range syntax. |

---

## Variables

User-supplied configuration values. Each key is the variable name (conventionally `UPPER_SNAKE_CASE`). Variables are referenced in bindings and SOUL.md templates using `{{VARIABLE_NAME}}` syntax.

```json
"variables": {
  "USER_NAME": {
    "type": "string",
    "description": "Your name, used in drafts and briefings",
    "required": true
  },
  "WRITING_VOICE": {
    "type": "string",
    "description": "Description of your writing voice/tone",
    "default": "Direct, conversational, no jargon.",
    "required": false,
    "sensitive": false
  }
}
```

### Variable Properties

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `type` | string | yes | -- | Value type. One of: `string`, `number`, `boolean`. |
| `description` | string | no | -- | Human-readable description shown to the user at install time. |
| `default` | any | no | -- | Default value if the user does not provide one. Type must match the `type` field. |
| `required` | boolean | no | `false` | If `true`, the user must supply a value. Required variables should not have a default. |
| `sensitive` | boolean | no | `false` | If `true`, the value is never stored in plaintext. Use for API keys, tokens, passwords. |

### Variable Declaration Rules

- Every `{{VARIABLE}}` referenced in bindings, SOUL.md templates, or cron prompts must be declared in `variables`.
- Required variables (`"required": true`) must not have a `"default"` value. The user must provide them.
- Optional variables should have a sensible `"default"`.
- Sensitive variables (`"sensitive": true`) are never written to `.env.example` with real values. Use a placeholder like `your-api-key-here`.
- Variable names follow `UPPER_SNAKE_CASE` convention.
- The `type` field constrains what values are accepted at install time.

---

## Agent Definition Schema

Each agent is keyed by its slug (lowercase, hyphenated). The slug is used in `agentToAgent`, `bindings`, `cron`, and file paths.

```json
"agents": {
  "chief-of-staff": {
    "source": "agents/chief-of-staff",
    "description": "Single point of contact for the formation.",
    "role": "coordinator",
    "model": "anthropic/claude-opus-4-6",
    "tools": {
      "allow": ["web-search", "file-read", "file-write"]
    },
    "sandbox": {
      "network": true,
      "filesystem": "restricted"
    }
  }
}
```

### Agent Properties

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `source` | string | yes | Relative path to the agent's directory from the formation root. Always `agents/<slug>`. |
| `description` | string | yes | What this agent does. Be specific and concise. |
| `role` | string | no | Agent's role label (e.g. `coordinator`, `researcher`, `writer`, `builder`, `reviewer`). |
| `model` | string | no | LLM model identifier in `provider/model-name` format (e.g. `anthropic/claude-opus-4-6`, `google/gemini-3-flash-preview`). |
| `tools.allow` | array of strings | no | List of tool/skill names the agent is permitted to use. |
| `sandbox.network` | boolean | no | Whether the agent can access the network. |
| `sandbox.filesystem` | string | no | Filesystem access level. One of: `full`, `restricted`, `none`. |

### Agent Slug Rules

- Lowercase alphanumeric with hyphens only.
- The slug is used everywhere: `agentToAgent` keys and values, `bindings[].agent`, `cron[].agent`, and the `source` directory name.
- The `source` path must always be `agents/<slug>`.
- The agent directory must exist and contain at minimum a `SOUL.md` file.

---

## Topology (agentToAgent)

Directed adjacency list defining which agents can send messages to which other agents.

```json
"agentToAgent": {
  "chief-of-staff": ["inbox-manager", "research-analyst", "content-writer"],
  "inbox-manager": ["chief-of-staff"],
  "research-analyst": ["chief-of-staff"],
  "content-writer": ["chief-of-staff"]
}
```

### Topology Rules

- Keys are source agent slugs. Values are arrays of target agent slugs.
- Every slug referenced (as key or in a value array) must exist in the `agents` object.
- No self-loops. An agent cannot list itself as a target.
- Edges are directional. If A can message B, that does not mean B can message A. Add both directions explicitly if needed.
- Agents not listed as keys cannot initiate messages to other agents (they can still receive).
- Common patterns:
  - **Hub-and-spoke:** One coordinator connects to all specialists. Each specialist connects only to the coordinator.
  - **Pipeline:** A -> B -> C -> D. Each agent connects to the next in sequence.
  - **Mesh:** Every agent can message every other agent (use sparingly; most formations benefit from structure).

---

## Bindings

Channel routing rules that connect external communication channels to agents.

```json
"bindings": [
  { "channel": "{{INTERACTION_CHANNEL}}", "agent": "architect" },
  { "channel": "email:inbox", "agent": "inbox-manager" }
]
```

### Binding Properties

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `channel` | string | yes | Channel identifier. Either a hardcoded functional channel or a `{{VARIABLE}}` reference. |
| `agent` | string | yes | Agent slug to bind. Must exist in the `agents` object. |

### Channel Format Convention

Bindings use match objects with a required `channel` token and optional peer targeting:

- **channel** -- The platform token: `slack`, `telegram`, `teams`, `discord`
- **peer.kind** -- Target type: `direct`, `group`, `channel`
- **peer.id** -- Target identifier: channel name, chat ID, room name
- **accountId**, **guildId**, **teamId**, **roles** -- Additional routing fields

Examples:
- `{ "match": { "channel": "slack" }, "agent": "triage" }` -- Any Slack message
- `{ "match": { "channel": "slack", "peer": { "kind": "channel", "id": "#general" } }, "agent": "triage" }` -- Slack #general
- `{ "match": { "channel": "telegram", "peer": { "kind": "direct", "id": "12345" } }, "agent": "support" }` -- Telegram DM

### Interaction vs. Functional Channels

**Interaction channels** are user-configurable. They use `{{VARIABLE}}` syntax and the variable must be declared in `variables`. The user sets the value at install time.

**Functional channels** are hardcoded strings. They represent fixed system integrations that do not change per deployment (e.g. `email:inbox`, `webhook:/deploy`).

Rule: Never hardcode an interaction channel. If the user should be able to choose the platform or target, use a variable.

---

## Cron

Scheduled jobs that send a prompt to an agent on a recurring schedule.

```json
"cron": [
  {
    "schedule": "*/5 * * * *",
    "agent": "inbox-manager",
    "prompt": "Check for new emails. Categorize and triage.",
    "timezone": "America/New_York"
  }
]
```

### Cron Properties

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `schedule` | string | yes | -- | Standard cron expression (5-field: minute, hour, day-of-month, month, day-of-week). |
| `agent` | string | yes | -- | Agent slug to trigger. Must exist in the `agents` object. |
| `prompt` | string | yes | -- | Message sent to the agent when the cron fires. Be specific about what the agent should do. |
| `timezone` | string | no | `"UTC"` | IANA timezone identifier (e.g. `America/New_York`, `Europe/London`, `Asia/Tokyo`). |

### Cron Format Reference

```
* * * * *
| | | | |
| | | | +-- Day of week (0-7, where 0 and 7 are Sunday)
| | | +---- Month (1-12)
| | +------ Day of month (1-31)
| +-------- Hour (0-23)
+---------- Minute (0-59)
```

Common patterns:
- `*/5 * * * *` -- Every 5 minutes
- `*/15 * * * *` -- Every 15 minutes
- `0 * * * *` -- Every hour on the hour
- `0 8 * * *` -- Daily at 8:00 AM
- `0 7 * * 1-5` -- Weekdays at 7:00 AM
- `0 0 * * 0` -- Weekly on Sunday at midnight

### Cron Prompt Guidelines

- Be explicit about what the agent should do. Do not write vague prompts like "do your thing."
- Include the expected outputs: what should the agent produce, who should it notify.
- If the cron triggers a multi-step process, list the steps.
- Keep prompts under 500 characters. Detailed behavior belongs in the agent's SOUL.md, not in the cron prompt.

---

## Dependencies

Required skills and external services.

```json
"dependencies": {
  "skills": {
    "web-search": "^1.0.0",
    "file-read": "^1.0.0"
  },
  "services": [
    {
      "name": "Anthropic API",
      "url": "https://console.anthropic.com",
      "required": true,
      "description": "LLM provider for coordinator and research agents"
    }
  ]
}
```

### Skills

ClawHub skills with semver version ranges. Keys are skill names, values are version range strings.

| Field | Type | Description |
|-------|------|-------------|
| `<skill-name>` | string | Semver version range (e.g. `^1.0.0`, `>=2.0.0`, `~1.2.0`). |

### Services

External services the formation depends on. Informational for the user at install time.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | string | yes | -- | Service name (e.g. `Anthropic API`, `GitHub API`). |
| `url` | string | no | -- | Service URL for the user to sign up or configure. |
| `required` | boolean | no | `true` | Whether the service is required for the formation to function. |
| `description` | string | no | -- | What the service is used for in this formation. |

---

## Validation

Post-deploy health check configuration. Controls what `reef validate` checks after deployment.

```json
"validation": {
  "agent_exists": true,
  "file_exists": true,
  "binding_active": true,
  "cron_exists": true,
  "agent_responds": {
    "enabled": false,
    "timeout": 30
  }
}
```

### Validation Properties

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `agent_exists` | boolean | `true` | Verify all agents declared in `agents` were created in the runtime. |
| `file_exists` | boolean | `true` | Verify workspace files (SOUL.md, IDENTITY.md, knowledge dirs) were written. |
| `binding_active` | boolean | `true` | Verify all bindings declared in `bindings` are active and routable. |
| `cron_exists` | boolean | `true` | Verify cron jobs declared in `cron` were created in the scheduler. |
| `agent_responds.enabled` | boolean | `false` | Functional connectivity check. Sends a test message and waits for a response. Opt-in only; requires `--verify-connectivity` flag at deploy time. |
| `agent_responds.timeout` | number | `30` | Timeout in seconds for agent response check. Minimum 1, maximum 300. |

### Validation Guidelines

- Always set `agent_exists`, `file_exists`, and `binding_active` to `true`. These are the baseline checks.
- Set `cron_exists` to `true` if the formation has any cron jobs. Set to `false` if there are no cron entries.
- Leave `agent_responds.enabled` as `false` for most formations. It is a heavyweight check that requires all agents to be fully operational. Enable it only when requested or for production-critical formations.

---

## Complete reef.json Structure

For reference, here is the full skeleton with every possible field:

```json
{
  "reef": "1.0",
  "type": "shoal",
  "name": "my-formation",
  "version": "0.1.0",
  "description": "What this formation does.",
  "author": "Author Name",
  "license": "MIT",
  "repository": "https://github.com/org/repo",
  "namespace": "my-ns",
  "compatibility": {
    "openclaw": ">=0.5.0"
  },
  "variables": {},
  "agents": {},
  "agentToAgent": {},
  "bindings": [],
  "cron": [],
  "dependencies": {
    "skills": {},
    "services": []
  },
  "validation": {
    "agent_exists": true,
    "file_exists": true,
    "binding_active": true,
    "cron_exists": true,
    "agent_responds": {
      "enabled": false,
      "timeout": 30
    }
  }
}
```

No additional properties are allowed at any level. The schema uses `"additionalProperties": false` throughout. If you add a field not listed here, validation will fail.
