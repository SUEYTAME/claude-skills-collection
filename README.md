# My Skills Collection

A personal Claude Code / Cowork **plugin marketplace** bundling 140 curated AI skills and 14 development agents into 10 installable plugins.

## Install

**In the Claude desktop app (Cowork):**
1. Open **Customize** (bottom-left) → **Skills** or **Personal plugins**
2. **Personal plugins → +** → **Add marketplace from GitHub**
3. Enter: `SUEYTAME/claude-skills-collection`
4. Install the plugins you want — all their skills appear in the Skills panel.

**In Claude Code (CLI):**
```
/plugin marketplace add SUEYTAME/claude-skills-collection
/plugin install product-management@my-skills-collection
```

## What's inside

| Plugin | Contents | Source |
|---|---|---|
| **engineering-lifecycle** | 24 skills | [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) |
| **product-management** | 68 skills | [phuryn/pm-skills](https://github.com/phuryn/pm-skills) |
| **frontend-design** | 13 skills | [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) |
| **sparc-and-extras** | 12 skills | [ruvnet/ruflo](https://github.com/ruvnet/ruflo) |
| **superpowers** | 14 skills | [obra/superpowers](https://github.com/obra/superpowers) |
| **decision-tools** | 1 skill | [aiwithremy/claude-skills-llm-council](https://github.com/aiwithremy/claude-skills-llm-council) |
| **openhuman-shipping** | 1 skill | [tinyhumansai/openhuman](https://github.com/tinyhumansai/openhuman) |
| **browser-automation** | 1 skill | [vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser) |
| **code-minimalism** | 6 skills | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) |
| **dev-agents** | 14 agents | SUEYTAME (personal subagents) |

### dev-agents

Installable subagents that Claude Code can dispatch automatically or on request:

- **Pipeline:** `architectobot` (planning), `codecrusher` (implementation), `qualityqueen` (QA), `taskmaster` (orchestration)
- **Platform:** `build-agent`, `deploy-agent`, `dev-agent`, `mobile-agent`, `test-agent`
- **PR workflow:** `pr-manager`, `pr-manager-lite`, `pr-reviewer`
- **Other:** `designguru` (UI/UX), `memory-keeper` (session learnings)

```
/plugin install dev-agents@my-skills-collection
```

## Credits & licenses

All skills are the work of their original authors (linked above), redistributed here for personal use under their respective open-source (mostly MIT) licenses. Full credit to the original creators. This repo is an aggregation for convenience, not original authorship.
