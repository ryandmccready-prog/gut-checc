# Gut Checc Report

**Project:** Weekly Engineering Retrospective (retro)
**Mode:** Skill
**Score:** 68 / 100
**Date:** 2026-03-12

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Instructions | 23/25 | Exceptionally specific. Every step has exact commands, exact metrics, exact output formats. Tone rules are concrete, not vibes. One of the most well-instructed skills I've reviewed. |
| Examples | 12/25 | Multiple format examples (histogram, trends table, JSON schema, tweetable summary) anchor individual pieces well. Missing a complete end-to-end output example showing what a full retro looks like. |
| Edge Cases | 14/25 | Handles zero commits, first-run, argument validation, and Pacific time. Missing handling for repos without conventional commits, very few commits, failed git commands, and team vs solo repos. |
| Usability | 19/25 | Clean invocation with intuitive arguments. Description nails the trigger. Compare mode is a smart feature. ~340 lines is moderate but justified by the command specificity. Self-contained by design. |

---

## What's Solid

- **The specificity is exceptional:** Every metric has an exact computation method. Every output format has a visual template. Every git command is written out. There's almost zero room for Claude to improvise the wrong thing. This is how you write instructions.
- **Persistent history with trend tracking is a killer feature:** Saving JSON snapshots to `.context/retros/` and auto-loading the most recent one for comparison turns a one-off report into a living feedback loop. Most retro skills are fire-and-forget. This one builds context over time.
- **The tone rules are concrete, not vibes:** "Skip generic praise ('great job!') — say exactly what was good and why" is the opposite of "be professional." It tells Claude exactly what the tone IS and IS NOT, with specific examples of what to avoid.
- **Argument validation with usage message:** Invalid input gets a clean help message and stops. No guessing, no weird partial runs.
- **Compare mode is well-designed:** Using `--since` and `--until` to avoid overlap between windows shows attention to the git log edge cases that most builders miss.

---

## Findings

### 🔴 P1 — Fix First

#### No Complete Output Example

**What happens:** The skill shows individual format pieces — the metrics table, the histogram, the trends comparison, the JSON schema — but never shows what a complete retro looks like assembled together. Claude knows each piece but has to improvise how they flow together, how long the narrative sections should be, and how the tone plays out across 2500-3500 words.

**Why it matters:** The individual format examples anchor the structural pieces well (that's why your Instructions score is so high). But the narrative sections — "Time & Session Patterns," "Shipping Velocity," "Top 3 Wins," "3 Things to Improve" — have descriptions but no examples. These are the sections where Claude's voice, judgment, and depth matter most, and they're the sections most likely to drift between runs. One run might give you a punchy 3-sentence wins section. The next might give you a full paragraph per win.

**Scoring impact:** Examples

**What good looks like:** Add one complete retro output as a bundled reference file (e.g., `references/example-retro.md`). It doesn't need to be from a real repo — mock data is fine. The value is showing Claude the assembled final product: the tweetable summary, the metrics table, the narrative sections, the wins, the improvements, the habits. When Claude can see the whole thing, it nails the proportions and tone every time. Keep it out of SKILL.md to manage length — reference it with "Read `references/example-retro.md` before generating output."

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

### 🟡 P2 — Strengthen

#### No Handling for Repos Without Conventional Commits

**What happens:** Step 5 (Commit Type Breakdown) categorizes by conventional commit prefix: feat/fix/refactor/test/chore/docs. If the repo uses freeform commit messages ("updated login page," "fixed the thing," "wip"), every commit falls into an uncategorized bucket and the percentage bar is meaningless.

**Why it matters:** Conventional commits are common in disciplined repos but not universal. A builder using this skill on a project with freeform messages gets a broken section that either shows 100% "other" or skips the breakdown entirely without explanation. The fix ratio flag ("fix ratio exceeds 50%") also depends on accurate categorization.

**Scoring impact:** Edge Cases

**What good looks like:** Add a fallback: "If fewer than 50% of commits match conventional prefixes, note that the repo doesn't use conventional commits. Instead, categorize by keyword heuristics (commits containing 'fix', 'bug', 'patch' → fix; commits containing 'add', 'new', 'create' → feat; etc.) and note the categorization is approximate." This keeps the section useful without requiring a specific commit convention.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### No Error Handling for Git Commands

**What happens:** Step 1 runs five git commands in parallel. If any of them fail — no remote named origin, no main branch, permission error, shallow clone without full history — the skill has no guidance on what to do. Claude either crashes the run, skips the failed data silently, or improvises.

**Why it matters:** The git fetch at the top (`git fetch origin main --quiet`) is the most likely failure point — not every repo has an `origin` remote or a `main` branch (some use `master`, some have custom names). A single failed command shouldn't torpedo the entire retro.

**Scoring impact:** Edge Cases

**What good looks like:** Add a brief error handling note: "If `git fetch origin main` fails, try `git fetch origin` and detect the default branch name with `git remote show origin | grep 'HEAD branch'`. If no remote exists, fall back to the local branch. Note any data limitations in the output."

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Very Few Commits Produces Thin Output

**What happens:** If the window has 1-3 commits, most sections produce degenerate output. Session detection finds one session. The histogram has one bar. Hotspot analysis shows 3 files. PR size distribution has one bucket. The "Top 3 Wins" section struggles to find 3 things. But the skill still runs all 13 steps.

**Why it matters:** A retro with 2 commits stretched across 13 sections feels padded and unhelpful. The metrics table shows numbers but the narratives have nothing meaningful to say. The builder walks away thinking the tool isn't useful when really the input just didn't have enough substance for a full analysis.

**Scoring impact:** Edge Cases

**What good looks like:** Add a threshold check after Step 1: "If the window has fewer than 5 commits, produce a shortened retro: metrics table, commit list, streak count, and a note suggesting a longer window. Skip session detection, histograms, and narrative sections that need more data to be meaningful."

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### AskUserQuestion Not in Allowed Tools

**What happens:** The allowed-tools list includes Bash, Read, Write, and Glob — but not AskUserQuestion. The skill is designed to be non-interactive (run and output), so this is likely intentional. But if Claude hits an ambiguous situation (unclear default branch, unexpected repo structure), it has no way to ask the user for clarification.

**Why it matters:** For a fully-automated workflow this is mostly fine. But edge cases — like detecting the wrong branch, or a repo with unusual structure — would benefit from one clarifying question rather than a wrong assumption. Without AskUserQuestion in the allowed list, Claude must either guess or fail.

**Scoring impact:** Edge Cases

**What good looks like:** If the skill is intentionally non-interactive, add a note: "This skill runs without user interaction. If git commands fail or data is ambiguous, note assumptions in the output rather than stopping." If you want the option to clarify, add AskUserQuestion to allowed-tools.

**Read more:** Anthropic Skills documentation — support.claude.com/en/articles/12512198-how-to-create-custom-skills

---

### 🟢 P3 — Polish

- **Grep missing from allowed-tools:** The eval-related patterns in some of the git commands could benefit from Grep for file searching. Minor since Bash covers it, but Grep is more targeted. *(Usability)*
- **"Round LOC/hour to nearest 50" is a nice touch:** This kind of small formatting rule prevents false precision. More skills should do this. No action needed — just a win worth noting. *(Instructions)*
- **The 2500-3500 word target is smart:** Gives Claude a lane without being rigid. On thin-data windows (few commits), the lower bound might force padding — consider making the target conditional on commit count. *(Instructions)*
- **`.context/retros/` directory choice is clean:** Keeps retro data out of the main project tree. Consider adding `.context/` to `.gitignore` guidance so retro data doesn't accidentally get committed. *(Edge Cases)*

---

## Bottom Line

This is the highest-scoring skill I've reviewed in this stack, and it earned it. The instruction specificity is genuinely excellent — exact commands, exact formats, exact metrics, exact tone rules. The persistent history feature turns a one-off tool into a feedback loop that gets more valuable over time. The compare mode is thoughtful design. This skill was clearly built by someone who actually uses retros and thought about what makes them useful versus what makes them generic.

The main gap is the missing complete output example. You've shown Claude every individual piece of the puzzle but never the assembled picture. One example retro (even with mock data) in a bundled reference file would lock in the narrative quality, section proportions, and tone consistency that currently drift between runs. After that, the edge case handling for unusual repos would round this out nicely.

**Start with:** Create a `references/example-retro.md` with one complete mock retro output. Add a line in SKILL.md: "Read `references/example-retro.md` before generating output." This anchors the narrative sections that currently have the most drift.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
