# Gut Checc Report

**Project:** Mega Plan Review (plan-ceo-review)
**Mode:** Skill
**Score:** 45 / 100
**Date:** 2026-03-12

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Instructions | 19/25 | Deeply specific review structure with strong constraints and mode commitment rules. Loses points for vague adjectives in Expansion mode ("beautiful," "elegant," "joy to operate") and Rails-specific assumptions throughout. |
| Examples | 8/25 | Template formats for tables, ASCII diagrams, and the completion summary are well-shown — but zero complete input/output examples of an actual plan review in action. |
| Edge Cases | 5/25 | Assumes a substantial Rails implementation plan every time. No handling for non-Rails projects, tiny plans, strategy-only plans, missing git repos, or sections that don't apply. |
| Usability | 13/25 | Clear 10-section workflow with stops. But ~480 lines risks inconsistent execution at the tail end, and Rails-specific commands limit the audience to one tech stack. |

---

## What's Solid

- **Mode commitment is brilliantly enforced:** "Once the user selects a mode, COMMIT to it. Do not silently drift." This is exactly how you prevent Claude from hedging. Most skills let Claude waffle — yours locks it in. The mode comparison table at the bottom is a great reference anchor.
- **The one-issue-per-question rule is smart design:** "One issue = one AskUserQuestion call. Never combine multiple issues into one question." This prevents Claude from dumping a wall of questions the user ignores. It forces Claude to prioritize and the user to engage with each decision.
- **The Error & Rescue Map is the standout section:** Showing the exact table format with specific exception classes, rescue actions, and user-visible outcomes is how you show format without describing it. This section alone would make most plan reviews better.
- **Priority hierarchy is explicit:** "Step 0 > System audit > Error/rescue map > Test diagram > Failure modes > Opinionated recommendations > Everything else. Never skip Step 0..." Under context pressure, Claude knows exactly what to cut and what to protect.

---

## Findings

### 🔴 P1 — Fix First

#### Zero Complete Review Examples

**What happens:** Claude has 10 detailed section descriptions and table templates, but has never "seen" what a completed section review looks like in practice. It knows the structure but not the depth, tone, or judgment quality you expect. Each run, it improvises how thorough to be, how long each section should be, and what "opinionated recommendation" actually sounds like in your voice.

**Why it matters:** You've built a 480-line skill with incredible structural detail — but examples are where consistency lives. The table templates anchor FORMAT beautifully (that's why your Instructions score is high). But the narrative portions — the architecture evaluation, the security assessment, the "lead with your recommendation" style — have no anchor. Claude knows the checklist but not the standard.

**Scoring impact:** Examples

**What good looks like:** You don't need to example every section — that would double the skill's length. Instead, show one complete section review (pick Section 2: Error & Rescue Map since it's your strongest section). Show a real plan input snippet, then show the full review output including the table, the narrative evaluation, and the AskUserQuestion call. This single example would anchor Claude's judgment quality across all 10 sections. Consider putting it in a separate bundled resource file rather than in SKILL.md to manage length.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Rails/Ruby Assumptions Baked Into the Skill

**What happens:** The system audit runs `grep -r "TODO\|FIXME" --include="*.rb" --include="*.js"` and `find . -name "*.rb" -newer Gemfile.lock`. Section 2 examples reference `ActiveRecord::ConnectionTimeoutError`, `Faraday::TimeoutError`, and `Rails.logger.error`. Section 7 checks for "N+1 queries" and "ActiveRecord association traversal." Section 9 asks about "rake tasks." The entire skill assumes a Rails application.

**Why it matters:** If someone uses this skill on a Python project, a Node.js project, or a Go service, the system audit commands will either fail or return nothing useful. The example exception classes won't match their stack. The performance checks reference Rails-specific patterns. The skill will still run — Claude is smart enough to adapt — but it'll be fighting against its own instructions the whole time, producing a less rigorous review.

**Scoring impact:** Edge Cases, Usability

**What good looks like:** Two approaches. The straightforward one: declare upfront that this skill is designed for Rails projects and name that in the description. The better one: make the system audit commands and example references tech-stack-aware. Instead of hardcoding `*.rb`, the skill could say "Identify the primary language and framework, then run equivalent commands for that stack." The table examples could use generic placeholders alongside the Rails-specific ones so Claude maps the pattern to any stack.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Skill Length Risks Inconsistent Execution

**What happens:** At ~480 lines, this skill is at the edge of what performs reliably in a single SKILL.md. Claude loads the full file when the skill activates, and by the time it reaches Section 8, 9, and 10, earlier details compete for attention. The sections that suffer most are the ones at the end — Long-Term Trajectory, Deployment, and Observability — which are also the ones most likely to get rushed or surface-level treatment.

**Why it matters:** The progressive disclosure system works in three levels: metadata (~100 tokens), full SKILL.md (ideally under 5k tokens / ~500 lines), and bundled resources. At 480 lines of dense instructional content, you're using almost all of Level 2's budget. The "STOP after each section" pattern helps because Claude processes sequentially, but the sheer instruction volume means later sections get less faithful execution than earlier ones.

**Scoring impact:** Instructions, Usability

**What good looks like:** Move reference material to bundled resource files. The mode comparison table, the section-by-section table templates (Error & Rescue format, Data Flow format, Failure Modes format), and the Completion Summary template are all reference material that Claude only needs when it reaches that section. If they lived in a `references/` folder inside the skill, SKILL.md could focus on the core logic — the philosophy, the prime directives, the section sequence, and the stop-and-ask rules — while the heavy templates load on demand. This could cut SKILL.md by 30-40%.

**Read more:** Anthropic Skills documentation — support.claude.com/en/articles/12512198-how-to-create-custom-skills

---

### 🟡 P2 — Strengthen

#### Aggressive Prompt Language Throughout

**What happens:** The skill uses "CRITICAL," "STOP," "Do NOT," "NEVER," "ALWAYS" in all-caps extensively. The "STOP. AskUserQuestion once per issue. Do NOT batch." pattern repeats 10 times. "CRITICAL rule" and "CRITICAL GAP" appear multiple times.

**Why it matters:** On Claude 4.6, this level of aggressive language can cause overtriggering — Claude over-indexes on the capitalized instructions at the expense of the surrounding nuance. The "STOP" pattern is important enough to emphasize, but when everything is screaming, nothing stands out. Normal-weight language like "Pause here and ask one question per issue" works just as well on newer models and lets the truly critical rules (like mode commitment) stand out when they ARE emphasized.

**Scoring impact:** Instructions

**What good looks like:** Reserve all-caps emphasis for the 2-3 truly critical rules (mode commitment, no code changes, no batching). Rewrite the repeated section-end instruction in normal weight: "Pause here. Ask one question per issue. Don't batch multiple issues. Wait for the user before moving on." Same instruction, less noise.

**Read more:** Anthropic Claude 4.6 best practices — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices

---

#### No Edge Case Handling for Input Variations

**What happens:** The skill assumes every input is a substantial, multi-file implementation plan for a codebase with git history, existing architecture, and enough substance to fill 10 review sections. What happens with a plan that's just three bullet points? A plan for a brand new project with no git history? A plan that's strategy-level (no code files mentioned)? A plan that's already well-reviewed and has no real issues?

**Why it matters:** A plan review skill that only handles substantial, well-structured plans is like a writing assistant that only handles polished drafts. The messy cases are where people need the most help. Without handling, Claude either forces all 10 sections onto a plan that doesn't warrant them (producing filler) or awkwardly skips sections without explaining why.

**Scoring impact:** Edge Cases

**What good looks like:** Add a few lines after Step 0 that scale the review to the plan's scope. Something like: "If the plan is under 20 lines, skip Sections 7-10 and note them as 'not applicable at this scope.' If there's no git history, skip the system audit and note it. If the plan is strategy-level with no code references, adapt the technical sections to focus on architecture and decision quality rather than implementation details."

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Vague Adjectives in Expansion Mode

**What happens:** Expansion mode asks Claude to evaluate whether architecture is "beautiful," "elegant," and would create "joy to operate." These are taste judgments with no definition. Claude interprets "beautiful architecture" differently every run — sometimes it means simple, sometimes it means well-abstracted, sometimes it means clever.

**Why it matters:** Every other instruction in this skill is impressively specific. The Prime Directives are concrete. The section checklists are concrete. But the Expansion mode additions rely on adjectives that tell Claude nothing actionable. This creates inconsistency specifically in the mode where you're pushing for the highest ambition.

**Scoring impact:** Instructions

**What good looks like:** Define what "beautiful" means in your engineering worldview. Based on your Engineering Preferences, it probably means: "explicit over clever, minimal diff, DRY, well-tested, and obvious to a new engineer in 6 months." Replace "What would make this architecture beautiful?" with "What would make a new engineer joining in 6 months say 'this is exactly how I'd build it'?" — which you actually already do in one place. Make that the consistent standard.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### System Audit Commands Will Fail Outside Git/Rails

**What happens:** The pre-review system audit hardcodes five shell commands including `git log`, `git diff main`, `git stash list`, `grep --include="*.rb"`, and `find . -name "*.rb" -newer Gemfile.lock`. If there's no git repo, no main branch, no Ruby files, or no Gemfile.lock, these commands fail and the audit produces nothing useful.

**Why it matters:** The system audit is in your Priority Hierarchy as the second-highest priority item ("Step 0 > System audit > ..."). If it fails, Claude either skips it (violating the priority rules) or wastes time on errors. Neither outcome serves the user.

**Scoring impact:** Edge Cases

**What good looks like:** Make the audit adaptive: "Detect the project's primary language and framework. Run the appropriate equivalents of these commands for that stack. If no git repo exists, note it and skip git-specific commands. If the project is greenfield with no history, note it and proceed to Step 0."

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

### 🟢 P3 — Polish

- **Description could be a stronger trigger:** "CEO/founder-mode plan review" isn't how someone naturally asks for this. "Review my implementation plan with maximum rigor" or "stress test this plan before I build it" matches how people would actually invoke it. *(Usability)*
- **Empty globs are appropriate:** Manual-only activation makes sense for a plan review skill. No action needed, just noting it's intentional. *(Usability)*
- **The "For Each Issue You Find" section duplicates the STOP instructions:** The formatting rules section near the end restates the same patterns from the section-end instructions. This adds length without adding clarity. *(Instructions)*
- **Allowed tools don't include Write or Edit:** If the skill is supposed to update TODOS.md (mentioned in Required Outputs), it needs Write or Edit in allowed-tools. Currently it can read and search but can't actually write the TODO updates it recommends. *(Edge Cases)*

---

## Bottom Line

This is one of the most ambitious and structurally rigorous skills I've seen. The 10-section review framework, the mode commitment rule, the one-question-per-issue pattern, and the Error & Rescue Map are genuinely excellent design. You clearly understand what makes a thorough code review and you've encoded that knowledge with impressive specificity.

The gaps are about adaptability and anchoring. The skill assumes one specific context (Rails, substantial plan, git history) and provides zero examples of what a great review actually looks like in practice. It's like having a world-class recipe with every step listed but no photo of the finished dish. Add one complete section example, make the audit adaptive to different tech stacks, and move the heavy reference templates to bundled resources — and this becomes a genuinely powerful review tool.

**Start with:** Add one complete example review of a single section (Section 2: Error & Rescue Map is ideal). Show a plan snippet going in, the review coming out, and a well-formed AskUserQuestion. This anchors Claude's judgment quality across all 10 sections and is the single highest-leverage change.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
