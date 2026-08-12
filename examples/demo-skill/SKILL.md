---
name: demo-skill
description: >
  A minimal example skill that demonstrates conformance to the Agent Skills
  Unified Skills Directory standard. Use this as a template when packaging a
  skill for cross-agent sharing.
version: 1.0.0
author: Agent Skills Standard Contributors
homepage: https://github.com/<your-org>/agent-skills-standard/tree/main/examples/demo-skill
license: CC0-1.0
tags:
  - example
  - template
---

# Demo Skill

This skill exists only to illustrate the **minimum compliant package layout**
defined in `SPEC.md` §3–§4. It is a spec anchor, not a working tool.

## What it shows

- A `SKILL.md` with the **required** `name` and `description` frontmatter fields.
- Optional `version`, `author`, `homepage`, `license`, and `tags` fields.
- No `scripts/`, `references/`, or `assets/` — those are optional and omitted here.

## How agents use it

An agent that reads the unified skills directory loads this skill's
`name` + `description` into its trigger context. When the user's intent
matches, the agent loads the body on demand (progressive disclosure).

That's the whole point: a skill is just a folder with a `SKILL.md`, and any
agent conforming to this standard can pick it up from the shared directory.
