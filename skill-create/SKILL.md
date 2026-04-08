---
name: skill-create
description: Guide for creating Claude Code skills. Use when the user wants to build a new skill, update an existing skill, or understand how to extend Claude with reusable specialized workflows, domain knowledge, or tool integrations.
---

# B6 Skill Creator

A skill is a folder in `.claude/skills/` that gives Claude reusable, specialized context for a specific domain or task. Instead of re-explaining the same workflow in every conversation, you encode it once — and Claude activates it automatically when it's relevant.

Think of a skill as a standing brief for a specific job. The richer the brief, the better Claude performs that job every time.

## What Skills Provide

1. **Specialized workflows** — multi-step procedures Claude follows automatically
2. **Domain expertise** — business-specific knowledge, schemas, and logic Claude can't infer
3. **Tool integrations** — instructions for working with APIs, file formats, or external services
4. **Bundled resources** — scripts, reference docs, and assets for complex or repeated tasks

## Anatomy of a Skill

```
.claude/skills/
└── skill-name/
    ├── SKILL.md          ← required
    ├── scripts/          ← optional: executable code
    ├── references/       ← optional: docs loaded on demand
    └── assets/           ← optional: files used in output
```

### SKILL.md (required)

Every SKILL.md has two parts:

**Frontmatter (YAML):**
```yaml
---
name: skill-name
description: What this skill does and when to use it.
---
```

**Body (Markdown):** Instructions for executing the skill. Only loaded after the skill triggers — so it can be as detailed as needed without affecting every conversation.

Only `name` and `description` go in frontmatter. Nothing else.

### Bundled Resources (optional)

**`scripts/`** — Executable code for tasks that require deterministic reliability or would otherwise be rewritten each session.
- *Example: a Python script that pulls Google Ads data and exports an Excel report*
- Benefits: token-efficient, consistent, can run without loading into context

**`references/`** — Documentation and reference material loaded into context only when Claude needs it. Keeps SKILL.md lean.
- *Example: a swipe file of approved ad copy Claude reads before writing new headlines*
- Best for: API docs, schemas, domain knowledge, detailed workflow guides

**`assets/`** — Files used in the output but not read into context.
- *Example: an Excel template with pre-built formatting that gets populated with data*

## Core Principles

### Concise is Key

The context window is shared. Skills compete for space with conversation history, other skill metadata, and the user's request. Only include what Claude doesn't already know.

**Default assumption: Claude is already very capable.** Challenge every paragraph: "Does Claude need this, or would it figure it out anyway?"

Prefer concrete examples over verbose explanations.

### Set Appropriate Degrees of Freedom

Match specificity to the task's fragility:

- **High freedom (loose text guidance):** Multiple valid approaches exist; judgment calls are needed
- **Medium freedom (pseudocode or parameterized patterns):** A preferred approach exists but some variation is fine
- **Low freedom (exact scripts, precise sequences):** Operations are fragile; consistency is critical

A narrow bridge with cliffs needs guardrails. An open field doesn't.

### The Description is the Triggering Mechanism

Claude reads every skill's `description` at the start of each conversation to decide which skills to activate. The body never loads unless the skill triggers — so the description is the most important thing you'll write.

A strong description includes:
- **What** the skill produces
- **What inputs** it expects
- **Specific phrases** that should trigger it

**Weak:** `"Helps with Google Ads."`

**Strong:** `"Audits a Google Ads search term report to identify wasted spend, classify query intent, and recommend negative keywords. Use when the user shares a search term CSV, asks to find irrelevant queries, or wants to reduce wasted ad spend."`

## Progressive Disclosure

Skills use a three-level loading system:

1. **Frontmatter only** — always in context (~100 words per skill)
2. **SKILL.md body** — loaded when the skill triggers
3. **Bundled resources** — loaded only when Claude determines they're needed

This means you can have a detailed skill without it weighing on every conversation. Use `references/` aggressively to keep SKILL.md under 500 lines — and always tell Claude when to read each reference file.

**Pattern: keep SKILL.md as the workflow guide, references/ as the depth.**

```markdown
## Copy Generation

Read `references/swipe-file.md` before writing any headlines.
See `references/approved-copy.md` for client-approved examples (if it exists).
```

Claude loads those files only when it reaches that step.

## Skill Creation Process

### Step 1: Understand with Concrete Examples

Before writing anything, define 2–3 specific examples of how the skill will be used:
- What would a user say to trigger it?
- What does Claude produce?
- What inputs does it need?

Validate with real usage before finalizing anything.

### Step 2: Plan the Reusable Contents

For each example, ask: what would need to be rewritten from scratch without this skill?

- Code that runs repeatedly → `scripts/`
- Documentation Claude would have to rediscover → `references/`
- Templates or files used in output → `assets/`

### Step 3: Create the Folder

```
.claude/skills/your-skill-name/
```

### Step 4: Write SKILL.md

Start with the frontmatter. Write the description last — after you've written the body, you'll know exactly what the skill does and when it should fire.

**Writing style:** Use imperative form. Lead with actions. Skip preamble. Write for another instance of Claude, not a human reading docs.

### Step 5: Test and Iterate

1. Trigger the skill in a real conversation
2. Note where Claude struggles or goes off-track
3. Update SKILL.md or bundled resources accordingly
4. Repeat until the skill runs cleanly without correction

## What Not to Include

Only include files that directly support execution. Do not create:
- README.md, CHANGELOG.md, INSTALLATION_GUIDE.md
- Setup or testing documentation
- User-facing explainers

The skill exists for Claude, not for humans reading the folder.

## Worked Example

See `references/google-ads-example.md` for a complete Google Ads skill built using this pattern — frontmatter, workflow, output format, and edge case handling included.
