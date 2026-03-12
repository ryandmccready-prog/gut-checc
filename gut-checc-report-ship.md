# Gut Checc Report

**Project:** Ship Workflow (ship)
**Mode:** Skill
**Score:** 67 / 100
**Date:** 2026-03-12

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Instructions | 24/25 | Near-perfect specificity. Every step has exact commands, exact stop/continue rules, exact commit ordering. The "only stop for / never stop for" list is brilliant constraint design. |
| Examples | 10/25 | PR body template, commit message format, and CHANGELOG format are shown. Missing a complete end-to-end example showing what the user actually sees when they run `/ship`. |
| Edge Cases | 16/25 | Handles merge conflicts, test failures, main-branch abort, eval skipping, and pre-landing review failures. Missing handling for no VERSION file, no CHANGELOG, no gh CLI, and zero-commit branches. |
| Usability | 17/25 | Clean single-command invocation. Description nails the trigger perfectly. Project-specific references (bin/test-lane, specific eval runners) mean this only works for one codebase. |

---

## What's Solid

- **The "only stop for / never stop for" design is brilliant:** Explicitly listing what IS and ISN'T a blocking condition eliminates the most common automation failure — Claude stopping to ask about things it should just handle. "Never stop for: uncommitted changes (always include them)" and "never stop for: commit message approval (auto-commit)" are exactly right.
- **Bisectable commit splitting is thoughtful engineering:** The commit ordering rules (infrastructure → models → controllers → VERSION+CHANGELOG) with the "each commit must be independently valid" constraint shows real understanding of how git bisect works and why it matters. Most ship workflows just do one big commit.
- **The eval suite integration is thorough:** Step 3.25 checks if prompt-related files changed, maps affected eval suites, runs at the right tier, and stops on failure. The tier reference table with cost estimates is a practical touch that helps the builder understand the tradeoff.
- **Pre-landing review is a strong safety net:** Running a structured review against a checklist after tests pass but before shipping catches the structural issues that automated tests miss. The three-option response (fix/acknowledge/false-positive) per critical finding is well-designed.

---

## Findings

### 🔴 P1 — Fix First

#### Project-Specific References Limit Transferability

**What happens:** The skill references `bin/test-lane`, specific eval runner paths (`test/evals/*_eval_runner.rb`), a 4-digit VERSION format, a specific CHANGELOG structure, `.claude/skills/review/checklist.md`, and Rails + Vitest test commands. These are all specific to one codebase. Anyone else using this skill would need to rewrite nearly every step.

**Why it matters:** This is a judgment call — if this skill is meant for one project only, this is fine and you can treat it as a P3. But if you ever want to share this workflow or use it across projects, the hard-coded paths and commands make that impossible without a rewrite. The DESIGN of this skill (the stop/continue rules, bisectable commits, version bumping logic) is excellent and transferable. The IMPLEMENTATION is locked to one repo.

**Scoring impact:** Usability, Edge Cases

**What good looks like:** If this is single-project only, add a note in the description: "Configured for [project name]. Requires bin/test-lane, 4-digit VERSION file, and CHANGELOG.md." If you want it transferable, extract the project-specific commands into a config section at the top of the skill that a new user would customize, and make the rest of the workflow reference those config values.

**Read more:** Anthropic Skills documentation — support.claude.com/en/articles/12512198-how-to-create-custom-skills

---

#### No Complete End-to-End Example

**What happens:** The skill shows the PR body template, the commit message format, and the CHANGELOG format as individual pieces. But the user has never seen what a complete `/ship` run looks like from start to finish — what Claude outputs at each step, what the console experience feels like, and what the final PR looks like.

**Why it matters:** For a fully automated workflow, the user experience IS the output. If Claude is too verbose at each step, the run feels slow. If it's too silent, the user doesn't know what's happening. Without an example of the ideal run, Claude improvises the verbosity level — sometimes printing every git command output, sometimes just the PR URL. The "next thing they see is the review + PR URL" design goal is stated but never shown.

**Scoring impact:** Examples

**What good looks like:** Add a reference file showing an ideal `/ship` run transcript — the key outputs at each step, what gets printed vs what runs silently, and what the final PR URL output looks like. This anchors the pacing and verbosity that makes the difference between a smooth automation and a noisy one.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

### 🟡 P2 — Strengthen

#### Missing Edge Cases for Required Files

**What happens:** The skill assumes VERSION, CHANGELOG.md, and `.claude/skills/review/checklist.md` all exist. For VERSION and CHANGELOG, it reads them without checking. If VERSION doesn't exist, the version bump step crashes. If CHANGELOG doesn't exist, the entry generation fails.

**Why it matters:** The pre-landing review DOES handle this for the checklist ("If the file cannot be read, STOP and report the error") — which is the right pattern. But VERSION and CHANGELOG don't get the same treatment. A new branch on a project that hasn't set up these files yet would hit a confusing error mid-ship.

**Scoring impact:** Edge Cases

**What good looks like:** Add existence checks: "If VERSION doesn't exist, create it with 0.0.1.0 and note it in the commit. If CHANGELOG.md doesn't exist, create it with a header and the first entry." Alternatively, stop and tell the user to create them. Either way, handle it explicitly rather than crashing.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### No Handling for Zero-Commit Branch

**What happens:** If someone runs `/ship` on a branch with uncommitted changes but zero commits ahead of main, `git log main..HEAD --oneline` returns nothing. The CHANGELOG has nothing to generate from. The commit splitting has nothing to split. The PR has no meaningful diff.

**Why it matters:** This is an edge case but a realistic one — someone might stash work, switch branches, and accidentally run `/ship` on a branch that's even with main. The skill should catch this early rather than failing mid-workflow.

**Scoring impact:** Edge Cases

**What good looks like:** Add a check in Step 1 after the git log: "If there are no commits ahead of main and no uncommitted changes, abort: 'Nothing to ship — this branch is even with main and has no uncommitted changes.'"

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### gh CLI Dependency Not Checked

**What happens:** Step 8 uses `gh pr create` to create the pull request. If the GitHub CLI isn't installed or isn't authenticated, this step fails after all the previous work (tests, review, version bump, commits, push) has already been done.

**Why it matters:** Failing at the very last step after doing everything else is the worst place to fail. The user has committed, pushed, and bumped the version — but has no PR. They'd need to manually create it or figure out why `gh` failed.

**Scoring impact:** Edge Cases

**What good looks like:** Add a preflight check in Step 1: "Verify `gh` is installed and authenticated (`gh auth status`). If not, abort early: 'GitHub CLI not configured. Run `gh auth login` first.'"

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

### 🟢 P3 — Polish

- **No dry-run mode:** A `--dry-run` flag that shows what would happen without committing, pushing, or creating the PR would build trust during first use. Not critical since the skill is designed for speed, but it's a natural feature to add later. *(Usability)*
- **Parallel test execution is smart:** Running Rails and Vitest tests simultaneously with `tee` and `wait` is a real time saver. No action needed — just a win. *(Instructions)*
- **The version bump auto-decision is elegant:** MICRO for < 50 lines, PATCH for 50+, ask for MINOR/MAJOR. Simple, clear, saves a question 90% of the time. *(Instructions)*
- **"Never force push" is the right constraint:** Simple, clear, prevents the most common destructive git mistake. *(Instructions)*

---

## Bottom Line

This is an impressively well-designed automation workflow. The stop/continue rules, bisectable commit splitting, eval suite integration, and pre-landing review are all genuinely sophisticated features that show deep understanding of what makes a shipping workflow safe and fast. The instruction specificity is near-perfect — almost every step has exact commands and exact decision criteria.

The main question is audience. If this is a personal tool for one project, it's excellent and the project-specific references are just context, not a problem. If you want it to be shareable, the hard-coded paths need to become configurable. Either way, add a few existence checks for required files (VERSION, CHANGELOG) and a gh CLI preflight check so the workflow fails fast and early rather than late and messy.

**Start with:** Add preflight checks in Step 1 for VERSION, CHANGELOG.md, and `gh auth status`. Failing at the last step after doing all the work is the worst user experience in an automation workflow. Failing immediately with a clear message is the best.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
