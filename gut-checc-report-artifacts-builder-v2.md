# Gut Checc Report (Re-Run)

**Project:** Artifacts Builder Skill
**Mode:** Skill
**Score:** 27 → 60 (+33)
**Date:** 2026-03-09

---

## Score Breakdown

| Category | Score | Delta | Notes |
|----------|-------|-------|-------|
| Instructions | 9 → 18 | +9 | The 4-step approach, component selection table, and 5 hard constraints transformed Step 2 from empty to the strongest section. Some room left on state patterns and responsive guidance. |
| Examples | 2 → 17 | +15 | Two solid examples with different complexity levels. Both show the request, approach, component tree, and key decisions. Missing an edge case example. |
| Edge Cases | 5 → 9 | +4 | Constraints implicitly handle several edge cases (sandbox limitations, scope creep, routing overuse). But no explicit handling for script failures, ambiguous requests, or requests too simple for this stack. |
| Usability | 11 → 16 | +5 | Dead "Common Development Tasks" reference replaced with real content. Component table makes mapping easy. Still no globs, no version, minimal reference section. |

---

## What's Solid

- **Step 2 went from weakest to strongest.** The 4-step approach (views → components → state → build outside-in) gives Claude a clear thinking process, not just a list of things to do. This is exactly the kind of structural guidance that produces consistent results across different requests.
- **The constraints are precise and explained.** "Don't make external API calls — artifacts run sandboxed" tells Claude both the rule AND why. Claude follows "don't" rules more reliably when it understands the reason. Every constraint here has a reason attached.
- **The component table is a cheat sheet.** Six artifact types mapped to specific components. This eliminates the "which shadcn component should I use?" guessing game and keeps component selection consistent across runs.
- **Examples show decisions, not just outputs.** "Drag-and-drop skipped intentionally — adds major complexity, user didn't ask for it" teaches Claude how to think about scope, not just what to build. That's more valuable than showing finished code.

---

## Previous P1s — Status

| Finding | Status |
|---------|--------|
| Zero Examples | ✅ Resolved — two complete examples with different complexity levels |
| Step 2 Is a Blank Check | ✅ Resolved — now the most detailed section with approach, component guide, and constraints |
| Design Guidance Is Anti-Patterns Only | ✅ Resolved — 5 positive guidelines paired with the anti-patterns |

---

## Findings

### P2 — Strengthen

#### No Error Recovery or Troubleshooting

**What happens:** The Skill assumes every script runs, every install succeeds, and every build completes. When `init-artifact.sh` fails because of a Node version issue, or `bundle-artifact.sh` hits a TypeScript error, Claude has no playbook for diagnosing or recovering.

**Why it matters:** Build tools fail regularly. Dependency conflicts, TypeScript strictness errors, Parcel config issues — these aren't rare. They're Tuesday. Without guidance, Claude either retries blindly, dumps a raw error at the user, or improvises a fix that might make things worse. A short troubleshooting section covering the 3-4 most common failure points would save a lot of confusion.

**Scoring impact:** Edge Cases

**What good looks like:** Add a brief "If things break" section after Step 3. Cover the obvious ones: init script fails (check Node version, check that scripts directory exists), bundle fails (check for TypeScript errors in build output, verify index.html exists in root), dependency conflicts (clear node_modules and reinstall). You don't need to cover every failure — just the ones a user will hit in the first week.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### No Guidance for "Wrong Tool" Requests

**What happens:** The description says "not for simple single-file HTML/JSX artifacts" but Claude has no guidance for the gray area. User asks for an interactive chart with two dropdowns. Is that "complex" enough for React + TypeScript + Vite + Parcel + Tailwind + shadcn? Or would a single HTML file handle it in a fraction of the time? The Skill activates, Claude spins up the full stack, and nobody questions whether it was necessary.

**Why it matters:** The full stack is powerful but heavy. Using it for something that could be a 50-line HTML file wastes time and adds complexity. Worse, it teaches Claude to always reach for the heavy tooling. A few lines defining the actual threshold — "if it's one view with fewer than 3 interactive elements, suggest a simple HTML artifact instead" — keeps the Skill honest about its own scope.

**Scoring impact:** Edge Cases, Instructions

**What good looks like:** Add a short decision gate near the top. Something like: "Before initializing, assess whether the request genuinely needs this stack. If the artifact is a single view with basic interactivity (one or two inputs, no shared state, no component reuse), a simple HTML artifact is faster and simpler. Use this Skill when there are multiple views, shared state between components, or heavy use of shadcn/ui."

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### No Handling for Vague or Ambiguous Requests

**What happens:** Both examples show clear, detailed user requests. Real usage isn't always that clean. "Build me something cool for my startup" or "make a dashboard" with no other details. The Skill doesn't tell Claude what to do when the input is too vague to make good component and design decisions.

**Why it matters:** Without guidance, Claude either guesses (building something the user didn't want) or asks a generic "what would you like?" question that doesn't move things forward. A specific instruction — "if the request is too vague to identify components, ask about the data they want to display and the primary action they need to take" — gives Claude a concrete next step instead of a shrug.

**Scoring impact:** Edge Cases

**What good looks like:** Add a short "If the request is unclear" block in Step 2. Two or three lines: "If you can't identify at least two specific components from the request, ask the user what data or content the artifact should display and what the user should be able to DO with it. These two answers are enough to start building."

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

### P3 — Polish

- **Examples show structure but not visual output.** Both examples describe the approach, component tree, and key decisions — all valuable. But neither shows what the finished artifact looks like to the user. Even a brief description of the visual result ("the dashboard renders as a 2x2 card grid above a full-width transaction table, light gray background, no sidebar") would give Claude a visual target, not just a structural one. *(Examples)*
- **Testing advice could be more nuanced.** "Avoid testing upfront" makes sense for simple artifacts but is risky for complex multi-component builds. A note like "for artifacts with shared state or form submission logic, a quick test before presenting catches the bugs the user would see immediately" balances speed with quality. *(Instructions)*
- **No version or changelog.** Same as the first run — as this Skill evolves, a version line helps track iterations. *(Usability)*
- **Empty globs, no explanation.** Manual-only activation probably makes sense here. Adding a comment in the frontmatter ("globs intentionally empty — this Skill triggers on conversational requests, not file patterns") makes that decision explicit. *(Usability)*
- **Reference section is still one link.** Adding links to React patterns, Tailwind docs, and Vite troubleshooting gives Claude better sources when it needs to look something up mid-build. *(Usability)*

---

## Bottom Line

This is a different Skill than it was an hour ago. The three P1s are all resolved. Step 2 went from one dead-end sentence to the most detailed section with a clear thinking process, a component mapping table, and hard constraints that prevent real problems. The two examples anchor the approach concretely. The design section now tells Claude where to go, not just where to avoid.

The remaining gaps are about resilience — what happens when things don't go perfectly. Vague requests, build failures, requests that don't need this stack. These are P2s because they'll come up in real usage but they won't break the Skill on the happy path.

**Next move:** Add the "wrong tool" decision gate near the top and a short troubleshooting section after Step 3. Those two additions cover the biggest remaining edge case gaps without adding much length.

---

## Next Steps

1. Address the P2 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
