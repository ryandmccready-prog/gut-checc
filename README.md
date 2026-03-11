# Gut Checc

A stress test for projects built with Claude Code, Skills, and Cowork.

You drop in your project and Gut Checc scores it, finds every gap, ranks what to fix first, and teaches you why each finding matters so you actually learn something. It generates a markdown report you keep and reference. Run it again after improving and watch the score climb.

## What it grades

**Skills** — Are the instructions specific enough? Are there examples? Does it handle edge cases? Could someone new use it?

**Claude Code projects** — Does it work? Could someone else run it? Are secrets handled? Is it worth building?

**Cowork setups** — Is the task defined clearly? Is output consistent? Does it save time? Could someone else use it?

Four categories per mode. 25 points each. 100 total.

## Install

1. Download or clone this repo
2. Copy the `.claude/` folder and `CLAUDE.md` into your project root:

```
your-project/
  .claude/
    gut-checc-core.mdc
    gut-checc-scoring.mdc
    gut-checc-knowledge.mdc
    gut-checc-gotchas.mdc
    gut-checc-research.mdc
    settings.local.json
  CLAUDE.md
```

3. If on Mac, make sure the files are readable:

```
chmod 644 .claude/*.mdc
chmod 644 CLAUDE.md
```

## Quick start

Open Claude Code in your project folder and type:

```
start Gut Checc
```

Drop in the project you want to test. Gut Checc detects whether it's a Skill, Claude Code project, or Cowork setup and runs the right rubric.

You get a short chat summary with your score and P1 findings, plus a full markdown report saved to your project folder.

## What a report looks like

Gut Checc generates a `gut-checc-report-[project-name].md` file in your project root. Here's what to expect:

- **Score breakdown** with four categories, each scored out of 25 with a one-sentence explanation
- **What's solid** — specific things you did well
- **P1 findings** — things that will break. Fix these first
- **P2 findings** — things that make your build weaker. Strengthen after P1s
- **P3 findings** — polish items. Nice to have
- **Learn Why** — every P1 and P2 finding includes the principle behind it with a real source, so you understand WHY it matters
- **Bottom line** — honest read and where to start

## Re-runs

Run Gut Checc again after making changes. It finds your previous report automatically and shows a progress comparison with score deltas and resolved/open/new findings.

## Files

| File | What it does |
|------|-------------|
| `gut-checc-core.mdc` | The orchestrator. Voice, flow, output format, how a run works. |
| `gut-checc-scoring.mdc` | Scoring rubrics for all three modes. Point ranges for each category. |
| `gut-checc-knowledge.mdc` | Built-in knowledge base. What good looks like for Skills, CC projects, and Cowork. |
| `gut-checc-gotchas.mdc` | Specific traps new builders fall into. Gut Checc hunts for these on every run. |
| `gut-checc-research.mdc` | When and how to pull external docs. Only kicks in when the project uses third-party tools. |
| `settings.local.json` | Pre-configures web search and Anthropic docs permissions. |
| `CLAUDE.md` | Tells Claude Code what this project is and where to find the rules. |

## What Gut Checc does NOT do

- Does not rewrite your project
- Does not give you prompts to paste
- Does not compare your project to others
- Does not shame you for a low score. A 20 is a starting point, not a failure.

## Who this is for

New builders shipping with Claude's CC ecosystem who want honest feedback before they share their work. The kind of feedback an experienced builder would give you if they were looking over your shoulder.

## Built by

Ryan McCready / ThirdHat

v1.0
