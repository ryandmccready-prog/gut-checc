# Gut Checc Report

**Project:** Plan Review Mode (plan-eng-review)
**Mode:** Skill
**Score:** 50 / 100
**Date:** 2026-03-12

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Instructions | 20/25 | Extremely specific review structure, strong mode commitment rule, clear AskUserQuestion protocol. Loses points for Rails-specific assumptions and one tools gap. |
| Examples | 5/25 | Zero complete input/output examples. The TODO format spec is well-structured but no one has "seen" what a completed review looks like. |
| Edge Cases | 9/25 | SMALL CHANGE mode is smart scope handling. But no handling for non-Rails stacks, tiny plans, missing git history, or sections with nothing to review. |
| Usability | 16/25 | At ~163 lines, this is well within SKILL.md performance range. Clear 3-option scope selection. Description works as a trigger. One allowed-tools gap. |

---

## What's Solid

- **The three-mode scope selection is excellent design:** SCOPE REDUCTION / BIG CHANGE / SMALL CHANGE gives the user real control over review depth. The SMALL CHANGE mode is especially smart — it forces Claude to pick the single most important issue per section, which prevents the review from ballooning on minor diffs.
- **Mode commitment is enforced clearly:** "If I do not select SCOPE REDUCTION, respect that decision fully. Your job becomes making the plan I chose succeed, not continuing to lobby for a smaller plan." This prevents the most common failure mode in plan reviews — Claude endlessly pushing for less work.
- **The "escape hatch" rule saves time:** "If a section has no issues, say so and move on. If an issue has an obvious fix with no real alternatives, state what you'll do and move on — don't waste a question on it." This prevents Claude from manufacturing fake issues just to fill sections.
- **Unresolved decisions get tracked:** "If the user does not respond to an AskUserQuestion or interrupts to move on, note which decisions were left unresolved." This prevents silent defaults that create surprises during implementation.

---

## Findings

### 🔴 P1 — Fix First

#### Zero Complete Review Examples

**What happens:** Claude has detailed section descriptions and a strong questioning protocol but has never seen what a well-executed review actually looks like. It knows the rules but not the standard. Each run, it improvises how thorough each section should be, what "opinionated recommendation" sounds like in practice, and how much context to include per issue.

**Why it matters:** Examples do more than instructions ever can for locking in consistency. You've written clear rules for HOW to ask questions (numbered issues, lettered options, recommendation first), but without seeing a real example of that pattern executed, Claude interprets "lead with your recommendation" differently every time. Sometimes it's a decisive one-liner. Sometimes it's a paragraph of hedging with a preference at the end.

**Scoring impact:** Examples

**What good looks like:** Add one example showing a complete Architecture Review issue — the problem description with file references, the 2-3 lettered options, the "We recommend B:" opener, and the one-sentence WHY mapped to an engineering preference. This single example would anchor Claude's tone and judgment depth across all four sections. Put it right after the "For each issue you find" section so it's adjacent to the rules it demonstrates.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Bash Missing from Allowed Tools

**What happens:** The `allowed-tools` list includes Read, Grep, Glob, and AskUserQuestion — but not Bash. The Retrospective Learning section says "Check the git log for this branch" which requires running git commands. Without Bash, Claude can't execute the git log check that powers the retrospective analysis.

**Why it matters:** The retrospective check is one of the most valuable features in the skill — it catches areas that were previously problematic and flags them for more aggressive review. But if Claude can't actually run git commands, this entire section becomes a dead instruction. Claude either skips it silently or tries to ask the user to run the commands manually, breaking the review flow.

**Scoring impact:** Edge Cases, Usability

**What good looks like:** Add `Bash` to the allowed-tools list in frontmatter. If the intent is to keep the review purely reading-based (no command execution), then rewrite the Retrospective Learning section to say "Ask the user to provide the git log for this branch" or remove the git dependency entirely.

**Read more:** Anthropic Skills documentation — support.claude.com/en/articles/12512198-how-to-create-custom-skills

---

### 🟡 P2 — Strengthen

#### Rails/Ruby Assumptions in Review Criteria

**What happens:** The Performance Review section checks for "N+1 queries and database access patterns" — a Rails-specific concept. The Test Review references "JS or Rails test." The Code Quality section mentions checking for stale "ASCII diagrams in touched files" which assumes a specific documentation pattern. While less embedded than the CEO-version skill, these references still assume a Rails + JS stack.

**Why it matters:** Someone using this skill on a Python, Go, or Node-only project will hit review criteria that don't apply. Claude will either skip the Rails-specific checks (reducing review depth) or awkwardly try to map them (checking for "N+1 queries" in a language without ActiveRecord). Neither outcome produces the best review for that stack.

**Scoring impact:** Edge Cases

**What good looks like:** Two options. Simple: declare in the description that this skill is optimized for Rails + JS projects. Better: rewrite the stack-specific checks as patterns. Instead of "N+1 queries" say "inefficient data access patterns (N+1 queries in ORMs, unbatched API calls, unnecessary round-trips)." The concept transfers to any stack; the Rails terminology doesn't.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Aggressive Prompt Language

**What happens:** "CRITICAL RULE," "STOP," "Do NOT batch," "Never combine" appear throughout, along with bold-and-caps combinations. The stop-and-ask instruction repeats after every section in all-caps style.

**Why it matters:** On Claude 4.6, aggressive language can cause over-indexing — Claude focuses so hard on the capitalized rules that it under-weights the surrounding nuance. When everything is shouting, nothing stands out. The most important rules (mode commitment, one-question-per-issue) get the same emphasis level as formatting preferences.

**Scoring impact:** Instructions

**What good looks like:** Reserve heavy emphasis for the 2-3 genuinely critical rules. The section-end stop instruction could be: "Pause here. One issue per question. Wait for the user before continuing." Same behavior, less noise, and the truly critical rules (like mode commitment) stand out more when they ARE emphasized.

**Read more:** Anthropic Claude 4.6 best practices — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices

---

#### No Handling for Tiny or Non-Code Plans

**What happens:** The skill assumes every input is a multi-file implementation plan with enough substance for 4 full review sections. What happens when the plan is 5 lines? When it's a design doc with no code references? When it's a migration-only change?

**Why it matters:** The SMALL CHANGE mode handles small diffs well, but there's a gap between "small code change" and "plan that isn't about code at all." A design doc, an API contract change, or a pure data migration would force Claude to stretch the Architecture and Performance sections to fit content that doesn't apply.

**Scoring impact:** Edge Cases

**What good looks like:** Add a brief triage step after Step 0: "If the plan touches fewer than 3 files and has no architectural decisions, default to SMALL CHANGE mode and skip sections that don't apply. Note which sections were skipped and why."

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

### 🟢 P3 — Polish

- **Description could be a stronger trigger:** "Eng manager-mode plan review" doesn't match how someone naturally asks for this. "Review my implementation plan" or "stress test this plan before I build it" matches more natural invocations. *(Usability)*
- **Write/Edit missing from allowed-tools for TODOS.md:** The skill asks Claude to propose TODOS.md updates and present them via AskUserQuestion, but if the user says "add it," Claude can't actually write to the file without Write or Edit in allowed-tools. *(Usability)*
- **Completion summary could be more scannable:** The current text-based format works but a table (like the CEO-version's box format) would let the user see all findings at a glance. *(Usability)*

---

## Bottom Line

This is a well-scoped, focused plan review skill that avoids the length trap its CEO sibling falls into. At 163 lines, it's comfortably within SKILL.md's performance range, and the three-mode scope selection is genuinely smart design. The mode commitment rule and unresolved decisions tracking show real thought about how plan reviews actually go wrong.

The gap is the same one that hits most instructional skills: Claude knows the rules but has never seen the game played. One example of a completed issue review — with the numbered issue, lettered options, recommendation-first format, and engineering-preference mapping — would anchor the quality bar across every section. Fix the Bash gap in allowed-tools so the retrospective check actually works, and this becomes a solid everyday review tool.

**Start with:** Add one complete example of an Architecture Review issue showing the full AskUserQuestion format. Then add Bash to allowed-tools so the retrospective learning section actually works.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
