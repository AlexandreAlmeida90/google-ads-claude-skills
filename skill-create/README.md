# Skill Creator

A Claude Code skill that guides you through building new skills. Covers the anatomy of a skill, how to write effective trigger conditions, when to use scripts vs. inline instructions, and how to structure bundled resources for progressive disclosure.

## What It Does

- Explains the structure of a Claude Code skill (SKILL.md, scripts/, references/, assets/)
- Guides you through planning what to include and what to leave out
- Helps you write a strong description — the trigger mechanism that determines when Claude activates the skill
- Covers the three-level loading system (frontmatter → SKILL.md body → bundled resources)
- Walks through a worked example from an existing Google Ads skill

## When to Use It

Tell Claude:

> "Help me build a new skill for [task]"
> "I want to create a skill that [does X]"
> "How should I structure this skill?"

Claude will guide you through the process: defining examples, planning contents, writing SKILL.md, and testing.

## Key Concepts

**The description is the trigger.** Claude reads every skill's description at the start of each conversation to decide which skills to activate. A weak description means the skill never fires.

**Concise is key.** The context window is shared. Challenge every line: does Claude need this, or would it figure it out anyway?

**Progressive disclosure.** Keep SKILL.md as the workflow guide. Use `references/` for depth — Claude loads those files only when it reaches the step that needs them.

## Installation

Copy the skill into your project:

```
.claude/skills/skill-create/
```

No dependencies required.

## Reference

See `references/google-ads-example.md` inside the skill folder for a complete worked example — frontmatter, workflow, output format, and edge case handling.
