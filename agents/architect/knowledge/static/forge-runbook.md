# Forge Runbook

Quick-reference card for the Architect agent.

---

## Formation Topology

```
                         ┌─────────────┐
              user ────▶ │  architect  │ ◀──── {{INTERACTION_CHANNEL}}
                         └──┬──┬──┬──┬┘
                            │  │  │  │
          ┌─────────────────┘  │  │
          ▼                    │  │
  ┌──────────────┐             │  │
  │  researcher  │             │  │
  │ (research)   │             │  │
  └──────┬───────┘             │  │
         │ brief               │  │
         └──────────┐          │  │
                    ▼          │  │
                ┌──────────────┘  └──────────────┐
                ▼                                ▼
        ┌──────────────┐                ┌──────────────┐
        │ soul-writer  │ ──artifacts──▶ │   builder    │
        │  (SOULs)     │                │ (scaffold)   │
        └──────────────┘                └──────┬───────┘
                                               │ submits
                ┌──────────────────────────────┘
                ▼
        ┌──────────────┐
        │      qa      │ ──findings──▶ builder (round-trip)
        │ (adversarial)│ ──ecosystem──▶ architect
        └──────────────┘
```

**Edges (agentToAgent):**
- `architect → researcher` -- research requests with scope and constraints
- `architect → soul-writer` -- approved spec for SOUL/IDENTITY/knowledge authoring
- `architect → builder` -- approved spec + SOUL artifacts for scaffolding
- `architect → qa` -- (indirect: builder submits to QA after scaffolding)
- `researcher → architect` -- completed research briefs
- `soul-writer → architect` -- completed SOUL artifacts
- `soul-writer → builder` -- SOUL artifacts for integration into file tree
- `builder → architect` -- scaffolding status and completion notice
- `builder → qa` -- completed formation for adversarial review
- `qa → architect` -- escalation after max rounds; ecosystem issue findings
- `qa → builder` -- findings reports for each review round
- `qa → soul-writer` -- SOUL-specific revision requests
- `qa → researcher` -- requests for clarification on ecosystem constraints
No agent other than Architect communicates with the user.

---

## Phase Checklist

Verify each gate before advancing to the next phase.

### Phase 1: Understand
- [ ] Problem statement is clear and specific
- [ ] User's technical level and domain are known
- [ ] Agent count and responsibilities are defined
- [ ] External service dependencies are identified
- [ ] Interaction model is decided (hub-and-spoke, mesh, etc.)
- [ ] Cron requirements are captured (or confirmed as none)
- [ ] Interaction channel token is confirmed (e.g. slack, telegram, discord)

### Phase 2: Research
- [ ] Research request sent to Researcher with clear scope
- [ ] Researcher's brief received and reviewed
- [ ] Relevant ClawHub skills identified (or confirmed none exist)
- [ ] Ecosystem constraints documented (openreef/tide/openclaw gaps)
- [ ] Domain-specific findings incorporated into planning

### Phase 3: Specify
- [ ] Formation spec written using the standard template
- [ ] All agents listed with roles, models, and cron schedules
- [ ] Topology defined with explicit edge list and purpose for each edge
- [ ] Variables defined with types, defaults, required flags, and sensitivity
- [ ] Bindings defined with channel format
- [ ] Acceptance criteria written (measurable, specific)
- [ ] Spec shared with user and approved

### Phase 4: Build
- [ ] Soul Writer received spec and delivered all SOUL.md files
- [ ] Soul Writer delivered all IDENTITY.md files
- [ ] Soul Writer delivered knowledge/static files where needed
- [ ] Builder received spec + SOUL artifacts
- [ ] Builder produced reef.json, README.md, .env.example, .gitignore, reef.lock.json
- [ ] Builder produced complete directory tree
- [ ] Builder submitted formation to QA

### Phase 5: Review
- [ ] QA produced initial findings report
- [ ] Builder responded to all findings
- [ ] Rounds tracked (current round / {{MAX_QA_ROUNDS}})
- [ ] Ecosystem issues (if any) noted for delivery summary
- [ ] All issues resolved, OR escalation report produced

### Phase 6: Deliver
- [ ] Final formation presented to user
- [ ] QA resolution summary included
- [ ] Ecosystem issues included (for user awareness)
- [ ] Installation instructions provided
- [ ] User confirmed receipt

---

## Delegation Message Templates

### To Researcher

```
## Research Request

- **Phase:** 2 - Research
- **Formation:** {name}
- **Priority:** {P0-P3}

### Scope
{What to investigate: ecosystem review, ClawHub scan, domain research, prior art}

### Context
{Problem statement and user requirements from Phase 1}

### Deliverable
Structured research brief covering: ecosystem state, available skills, domain constraints, prior art.

### Constraints
{Any scope limits, focus areas, or time constraints}
```

### To Soul Writer

```
## SOUL Writing Request

- **Phase:** 4 - Build
- **Formation:** {name}

### Formation Spec
{Full formation spec from Phase 3}

### Research Brief
{Relevant excerpts from Researcher's brief}

### Deliverable
For each agent in the spec:
1. SOUL.md (personality, behavior, tools, communication, constraints)
2. IDENTITY.md (name, role, formation, description)
3. knowledge/static/ files (runbooks, reference material as needed)
```

### To Builder

```
## Scaffolding Request

- **Phase:** 4 - Build
- **Formation:** {name}

### Formation Spec
{Full formation spec from Phase 3}

### SOUL Artifacts
{List of all files delivered by Soul Writer}

### Deliverable
Complete formation file tree:
- reef.json (validated with `reef validate`)
- README.md
- .env.example
- .gitignore
- reef.lock.json
- agents/{name}/ directories with all artifacts placed

Submit to QA when complete.
```

---

## Escalation Report Template

```
## Escalation Report: {formation name}

**Rounds completed:** {n} / {{MAX_QA_ROUNDS}}
**Resolved issues:** {count}
**Unresolved issues:** {count}

### Unresolved Issues

#### Issue {n}: {title}
- **QA's position:** {finding and rationale}
- **Builder's position:** {response and rationale}
- **Architect's recommendation:** {your judgment}

### Ecosystem Issues (if any)
| Repo | Issue Title | Status |
|------|-------------|--------|
| {repo} | {title} | drafted / duplicate found |

### Recommendation
{Ship as-is / fix specific items / rework section / restart phase}
```

---

## Formation Type Quick Reference

| Type | Agents | Use When |
|------|--------|----------|
| **solo** | 1 | Single-purpose agent, no coordination needed. Examples: a standalone research bot, a single-channel notifier. |
| **shoal** | 2-5 | Small to mid-size collaboration. Agents share tasks with moderate topology. Examples: a writer + editor pair, daily-ops (5 agents), reef-forge (5 agents). |
| **school** | 6+ | Complex coordination. Many specialists with a coordinator hub. Requires explicit topology design. |

**Choosing a type:** Start with the simplest type that solves the problem. If the user describes a single task, it is probably solo. If they describe two complementary roles, it is probably shoal. If they describe a workflow with delegation, review, or multiple autonomous loops, it is school.
