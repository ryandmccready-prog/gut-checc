# Gut Checc

Gut Checc is a stress test tool for projects built with Claude's CC ecosystem (Claude Code, Cowork, and Skills). It helps non-technical builders find invisible gaps, learn why they matter, and build confidence to ship.

## How It Works

When a user says "start Gut Checc" or anything similar, follow the instructions in `.claude/gut-checc-core.mdc`. That is the core orchestrator and it references the other rule files:

- `.claude/gut-checc-core.mdc` — Start here. Voice, flow, output format, how a run works.
- `.claude/gut-checc-gotchas.mdc` — Specific traps to actively hunt for during stress tests.
- `.claude/gut-checc-scoring.mdc` — Scoring rubrics for all three modes.
- `.claude/gut-checc-knowledge.mdc` — Built-in knowledge base. What good looks like.
- `.claude/gut-checc-research.mdc` — External research protocol. Only when needed.
- `.claude/gut-checc-example-report.md` — Example of a completed report. Read this before generating reports.

## Important

- Every Gut Checc run generates a markdown report file. This is mandatory. The report is the deliverable.
- Chat stays short and conversational. Score, wins, P1 findings, and a pointer to the report.
- Scores are mandatory. Total and all four categories in the report with explanations for each.
- Always check the gotchas file during every stress test. Hunt for those patterns.
- Use the knowledge base for Learn Why sections before going to external research.
- Never give paste-ready fix prompts. Teach the principle. Let the builder learn.
