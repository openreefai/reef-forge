# Soul Writing Guide

Reference material for writing SOUL.md files that define agent behavior, personality, and boundaries. This guide covers each of the six required sections in depth, variable interpolation, anti-patterns to avoid, and tone calibration by role type.

---

## Section 1: Identity & Role

**Purpose:** Establish who the agent is in one paragraph. The agent reads this first and it frames everything that follows.

**Requirements:**
- Second person. Always "You are..." not "The agent is..."
- Name the formation using `{{namespace}}`
- Name the human user or role the agent serves (if applicable)
- Name specific sibling agents by their role, not generic references
- State what the agent owns outright vs. what it defers to others
- One paragraph. If it takes more than that, you're including behavioral rules here that belong in Section 2

**Good example:**
```
You are the research arm for the {{namespace}} formation. You activate when
the Architect delegates an investigation task. You produce structured briefs
with sourced findings and recommended actions. You don't make decisions about
what to research; the Architect scopes every request. You don't act on your
findings; you deliver them and the Architect routes the next steps.
```

**Bad example:**
```
You are a helpful research assistant. Your goal is to provide accurate and
thorough research on various topics. You work with other agents to accomplish
tasks and deliver value to the user.
```

The bad example could describe any research tool ever built. It says nothing about this agent's specific place in this formation.

---

## Section 2: Behavioral Rules

**Purpose:** Define how the agent operates. This is the longest section for most agents and the one that matters most for day-to-day behavior.

**Requirements:**
- Cover the agent's workflow end to end: what triggers it, what it does first, what decisions it makes, what it produces, what it does after producing
- Use numbered steps for sequential workflows, bullets for parallel options
- Specify trigger conditions: heartbeat interval, cron schedule, on-demand via delegation, or event-driven
- Include decision trees where the agent must choose between paths
- Reference other agents by name when describing handoffs

**Structure patterns:**

For **on-demand agents** (researcher, writer): describe the request-to-delivery cycle.
```
When you receive a research request from the Architect:
1. Confirm you understand the question. If ambiguous, ask before starting.
2. Gather sources using web-search and file-read.
3. Structure findings into the brief format (see Output Formats).
4. Run the quality checklist.
5. Deliver the brief to the Architect with a one-line summary.
```

For **scheduled agents** (briefing compiler, inbox manager): describe the cycle behavior.
```
Every 5 minutes on your heartbeat:
1. Check for new emails since last cycle.
2. Categorize each into one of four buckets.
3. Draft responses for routine items.
4. Flag urgent items to the Chief of Staff immediately.
5. Update the running inbox summary.
```

For **coordinator agents** (architect, chief of staff): describe routing logic and escalation.
```
When a new formation request arrives:
1. Decompose into agent specs: name, role, responsibilities, tools.
2. Delegate research to the Researcher with a scoped question.
3. Wait for the research brief. Review it. Pass relevant findings to the Soul Writer.
4. Route Soul Writer output to the Builder for scaffolding.
5. Send the built formation to QA for adversarial review.
6. If QA passes, deliver. If QA fails, route fixes to the responsible agent.
7. After {{MAX_QA_ROUNDS}} failed rounds, escalate to the user.
```

---

## Section 3: Output Formats

**Purpose:** Define the exact structure of everything the agent produces. Consistency is a feature.

**Requirements:**
- Use markdown code blocks to show templates
- Name every field in the template
- Specify length constraints where relevant ("3-5 sentences", "under 200 words")
- If the agent produces multiple output types, define each separately
- Include at least one realistic example or clearly annotated template

**Template pattern:**
```markdown
# [Output Title]

**Requested by:** [source agent or user]
**Date:** [date]

## Summary
[2-4 sentences. The recipient should understand the key point from this alone.]

## Details
### [Detail Area 1]
[specifics]

### [Detail Area 2]
[specifics]

## Recommended Actions
1. [First action]
2. [Second action]
```

**Key principle:** If two instances of the same output type look structurally different, the format spec is too loose. Tighten it.

---

## Section 4: Communication Protocol

**Purpose:** Define how the agent communicates, both in content and in style.

**Requirements:**
- Specify default tone and register
- Define message length defaults (one-liner for status, paragraph for analysis)
- Specify formatting rules: when to use bullets, when to use prose, when to use headers
- Define escalation language: how the agent signals urgency or blockers
- Describe inter-agent message conventions if applicable

**Tone should match role type.** See the Tone Calibration section below for specifics.

**Inter-agent message convention:**
```
**To:** [Agent Name]
**Re:** [Topic]
**Priority:** [normal / high / blocker]

[Body: what you need, what you're providing, what you expect back]
```

Agents should not bury the ask. Lead with what they need, then provide context.

---

## Section 5: Tool Usage

**Purpose:** Define which tools the agent uses and the rules governing that usage.

**Requirements:**
- List each tool the agent has access to
- For each tool, describe: when to use it, how to use it responsibly, what to avoid
- If the agent has filesystem access, specify read vs. write permissions and target directories
- If the agent has network access, specify what domains or APIs it calls
- Reference `{{tools}}` for the tool list when the formation spec provides one

**Example for a researcher agent:**
```
## Tool Usage

You have access to: {{tools}}

**web-search** - Use for current information, pricing data, and public documentation.
Do not search for information that should come from the codebase repos. Search
first, then verify against the cloned source when possible.

**file-read** - Use to read cloned repository files. You have read-only access
to the openreef, tide, and openclaw repos. Never attempt to modify these files.

**git-clone** - Clone ecosystem repos at the start of a research task if they're
not already present in your workspace. Clone once per task, not once per question.
```

---

## Section 6: Boundaries ("What You Never Do")

**Purpose:** Define the hard limits. These are the bright lines the agent must not cross, stated without ambiguity.

**Requirements:**
- 5-8 items per agent. Fewer means you haven't thought hard enough. More means you're padding
- Each starts with **Never** in bold
- Each includes the prohibition AND the reason or consequence
- Be specific. A boundary should be testable: you should be able to look at the agent's behavior and determine whether it violated the rule
- Cover at minimum: scope boundaries (what's not your job), authority boundaries (what decisions you can't make), safety boundaries (what actions could cause harm)

**Good boundaries:**
```
- **Never modify cloned repositories.** You have read-only access for a reason.
  Your job is analysis, not contribution.
- **Never present unverified claims as findings.** If you couldn't confirm it,
  say "unverified" explicitly. The Architect makes decisions based on your
  briefs; inaccurate data leads to bad formations.
- **Never skip the quality checklist.** Every deliverable runs through the
  checklist before handoff. Skipping it means shipping defects.
```

**Bad boundaries:**
```
- Be careful with sensitive information
- Try to be accurate
- Respect other agents' work
```

These are not boundaries. They are hopes. A boundary is a specific action the agent must never take, stated as a rule that can be followed or broken.

---

## Variable Interpolation Rules

SOUL.md files are templates. The runtime interpolates variables at boot. Use template variables wherever a value might change between deployments.

**Standard variables:**
- `{{namespace}}` - The formation name. Use this instead of hardcoding the formation name anywhere
- `{{tools}}` - The agent's allowed tool list from reef.json. Use in the Tool Usage section
- Custom variables defined in reef.json's `variables` block (e.g., `{{INTERACTION_CHANNEL}}`, `{{MAX_QA_ROUNDS}}`)

**Rules:**
1. Always use double curly braces: `{{variable_name}}`
2. Variable names match exactly what's defined in reef.json. Case-sensitive
3. Never invent variables that aren't defined in the formation spec. If you need a new variable, flag it in your handoff manifest so the Builder can add it to reef.json
4. Use variables for anything deployment-specific: channel names, usernames, repo URLs, thresholds, model names
5. Do not use variables for static content that won't change between deployments. "You are a researcher" doesn't need a variable

**Anti-pattern:** Hardcoding a value that's already a reef.json variable.
```
# Wrong
You report to the #ops-room Slack channel.

# Right
You report to {{INTERACTION_CHANNEL}}.
```

---

## Anti-Patterns

Patterns to avoid in every SOUL.md you write. If you catch yourself doing any of these, rewrite.

### AI-Obvious Language
Phrases that immediately signal "a language model wrote this":
- "Let's dive in" / "Let's explore"
- "I'd be happy to help"
- "Here's the thing"
- "In today's fast-paced world"
- "It goes without saying"
- "At the end of the day"
- "This is a great question"
- "To be more specific" / "More specifically"

These phrases add no information. Delete them. Write direct, active sentences instead.

### Vague Rules
Rules that sound good but provide no actionable guidance:
- "Be thorough and accurate" - How? What does thorough mean for this agent?
- "Handle requests efficiently" - What does efficient look like? What's the baseline?
- "Maintain high quality" - Against what standard? Measured how?

Replace with specifics: "Run every brief through the 8-point checklist before delivery." "Respond to delegated tasks within the same heartbeat cycle." "Flag research requests that will take more than one cycle to complete."

### Missing Boundaries
A SOUL without a "What You Never Do" section is a SOUL that lets the agent do anything. Agents need constraints the same way employees need job descriptions. Without explicit prohibitions, the agent will drift into other agents' territory, make decisions it shouldn't, or produce outputs nobody asked for.

### Copy-Paste Souls
Reusing the same SOUL structure across agents with only names swapped. Every agent has a distinct role. A Reviewer's behavioral rules look nothing like a Writer's. If two SOULs are more than 30% structurally similar, one of them is wrong.

### Aspirational Language
"Strive for excellence." "Aim for clarity." "Be the best researcher you can be." These are motivational posters, not behavioral rules. Agents don't have motivation. They have instructions. Write instructions.

### Over-Specifying Common Sense
"Always respond in English." "Use proper grammar." "Be professional." Unless there's a specific reason to call these out (multilingual formation, deliberately casual voice), these waste space. Focus rules on behaviors specific to this agent in this formation.

### Under-Specifying Critical Paths
Skimming over the workflows that matter most. If the agent's primary job is reviewing formations against a quality checklist, that checklist should be in the SOUL or in a knowledge/static file, not hand-waved as "review for quality." Spell out the criteria.

---

## Tone Calibration by Role Type

The voice of a SOUL.md should match the role it's writing for. A coordinator's SOUL reads differently from a researcher's because they operate differently.

### Coordinator
- **Register:** Authoritative. Commands, not suggestions
- **Sentence length:** Short. Direct. One idea per sentence
- **Verbs:** Route, delegate, escalate, decide, prioritize, flag
- **Pattern:** "If X, do Y. If Z, escalate." Decision trees, not essays
- **Avoid:** Hedging, qualifiers ("perhaps", "might want to"), open-ended options

### Researcher
- **Register:** Precise. Measured. Evidence-first
- **Sentence length:** Medium. Complex enough for nuance, short enough for clarity
- **Verbs:** Investigate, verify, source, distinguish, flag, assess
- **Pattern:** "Find X. Verify against Y. If conflicting, note both and flag confidence level."
- **Avoid:** Unsourced assertions, confident language about uncertain findings

### Writer
- **Register:** Crafted. The voice is the product
- **Sentence length:** Varied deliberately. Short for impact. Longer for rhythm
- **Verbs:** Draft, shape, cut, sharpen, match, voice
- **Pattern:** Show, don't tell. The Writer's SOUL should itself demonstrate good writing
- **Avoid:** Generic writing advice ("be clear and concise"), AI cliches, filler

### Reviewer
- **Register:** Rigorous. Clinical. Specific
- **Sentence length:** Short to medium. Findings stated as facts
- **Verbs:** Identify, cite, fail, pass, flag, reject, verify
- **Pattern:** "[File]:[Line] - [What's wrong] - [What the spec requires]"
- **Avoid:** Softening language ("could be improved"), vague praise ("looks good overall"), unsupported opinions

### Reporter
- **Register:** Factual. Structured. Numbers first
- **Sentence length:** Short. Often fragments for data points
- **Verbs:** File, draft, check, deduplicate, summarize, format
- **Pattern:** "[Count] items [action]. [Count] items [status]." Structured data, not narrative
- **Avoid:** Editorializing, narrative flourishes, opinions about the data

---

## SOUL.md Quality Checklist

Run through this before every handoff. Each item is pass/fail.

### Structure
- [ ] All 6 required sections present
- [ ] Sections in correct order: Identity, Behavioral Rules, Output Formats, Communication Protocol, Tool Usage, Boundaries
- [ ] Each section has substantive content (not placeholder or stub text)
- [ ] Total length is 80-150 lines (shorter for simple agents, longer for complex ones)

### Voice & Tone
- [ ] Second person throughout
- [ ] Voice matches role type (see Tone Calibration above)
- [ ] No AI-obvious language
- [ ] No filler sentences (every sentence does work)
- [ ] No aspirational language masquerading as rules

### Specificity
- [ ] Behavioral rules are testable (you could determine if the agent followed them)
- [ ] Output formats include concrete templates with field names
- [ ] Boundaries are specific prohibitions, not vague guidance
- [ ] Tool usage describes when and how, not just what

### Variables & References
- [ ] Formation namespace uses `{{namespace}}`
- [ ] Tool list uses `{{tools}}` or specific tool names from the spec
- [ ] No hardcoded values that should be variables
- [ ] No references to variables not defined in reef.json
- [ ] Sibling agents referenced by correct names from the formation spec

### Consistency
- [ ] IDENTITY.md fields match reef.json agent entry
- [ ] SOUL.md responsibilities match the agent description in reef.json
- [ ] Tool Usage section matches the tools.allow list in reef.json
- [ ] Knowledge/static files referenced in SOUL.md actually exist in the handoff

### Boundaries
- [ ] 5-8 boundary items present
- [ ] Each starts with "Never"
- [ ] Each includes the prohibition and the rationale
- [ ] Boundaries cover scope, authority, and safety at minimum
- [ ] No boundary is so obvious it wastes the agent's context window
