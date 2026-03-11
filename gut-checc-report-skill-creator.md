# Gut Checc Report

**Project:** skill-creator
**Mode:** Skill
**Score:** 50 / 100
**Date:** 2026-03-09

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Instructions | 17/25 | Clear 6-step process with specific writing style guidance, but limited negative constraints to prevent drift. |
| Examples | 9/25 | Partial illustrative examples in Step 2 but no complete input/output demonstration of a finished skill. |
| Edge Cases | 9/25 | Some conditional skip logic, but most failure scenarios (script errors, conflicting requirements, bad input) are unaddressed. |
| Usability | 15/25 | Well-structured process and a solid description, but missing a complete worked reference and undocumented frontmatter options. |

---

## What's Solid

- **The 6-step process is genuinely well-structured:** Each step builds on the last, with clear entry/exit criteria and "skip this step" conditions. A builder can follow this without getting lost.
- **The progressive disclosure teaching is excellent:** Explaining the three-level loading system (metadata → SKILL.md → bundled resources) gives builders a mental model they can apply to every skill they build.
- **The Step 2 analysis pattern is smart:** Walking through "how would I do this from scratch → what's reusable" is a great framework for identifying what belongs in a skill vs. what doesn't.

---

## Findings

### P1 — Fix First

#### No Complete Example of a Finished Skill

**What happens:** A builder follows all six steps but has never seen what "done" looks like, so the skill they produce may miss the mark structurally or qualitatively.

**Why it matters:** The skill teaches the process of creating skills but never shows the finished product. It's like teaching someone to cook a recipe by explaining technique but never showing the plated dish.

**Scoring impact:** Examples

**What good looks like:** The single most powerful thing a skill can include is a complete input/output example. For a skill-creator skill, that means showing a short but complete SKILL.md — frontmatter, instructions, constraints, examples, and edge case handling — for a simple, concrete use case. Even one 30-line example of a well-built skill would anchor everything the rest of the instructions describe. Builders learn format by seeing format, not by reading about it.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### Missing User-Facing Input/Output Examples

**What happens:** The skill has no examples showing "user says X → skill-creator does Y." The Step 2 examples show analysis reasoning (pdf-editor, frontend-webapp-builder) but not the actual interaction or output artifacts.

**Why it matters:** Without input/output pairs, Claude interprets the process differently each run. One run might front-load questions, another might jump straight to creating files. The examples that exist are illustrative fragments, not behavioral anchors.

**Scoring impact:** Examples

**What good looks like:** Two to three complete examples covering different scenarios: a user asking to build a skill from scratch, a user wanting to iterate on an existing skill, and a user with a vague request that needs Step 1 clarification. Each example should show the user prompt, Claude's response, and a snippet of what gets produced. These don't need to be long — even abbreviated examples dramatically reduce drift.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

### P2 — Strengthen

#### Limited Constraints (No "Don'ts")

**What happens:** The skill tells Claude what TO do but rarely what NOT to do. Without negative constraints, Claude falls back on default behaviors — adding verbose explanations, overengineering skill structures, or producing skills that are too complex for the use case.

**Why it matters:** Constraints are often more powerful than instructions. "Never create a skill with more than 3 reference files unless the user explicitly requests it" is the kind of boundary that prevents the most common drift pattern: overbuilding.

**Scoring impact:** Instructions

**What good looks like:** Strong skills include explicit "don'ts" alongside their instructions. Things like: "Do not create reference files for information that fits in SKILL.md." "Do not add example directories the skill won't use." "Never produce a SKILL.md longer than 300 lines without checking if content should move to references." These hard boundaries prevent Claude from drifting toward its default pattern of thoroughness-at-all-costs.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### Happy Path Process Only

**What happens:** The 6-step process assumes everything goes smoothly — user has clear requirements, init script works, packaging validates. No guidance on what to do when things go sideways.

**Why it matters:** Real usage is messy. A builder might provide conflicting requirements ("I want it simple but also handle 15 edge cases"). The init script might fail because of permissions or missing Python. An existing skill might be so poorly structured that iteration is harder than starting over. These are the moments where Claude needs guidance most.

**Scoring impact:** Edge Cases

**What good looks like:** Even basic edge case handling goes a long way. A few lines addressing the most likely failure points: "If the init script fails, create the directory structure manually with these files..." or "If the user's requirements conflict, surface the tension and help them prioritize before proceeding." The goal isn't covering every scenario — it's covering the three or four that happen most often.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### Globs Not Documented as a Frontmatter Option

**What happens:** The skill teaches builders to create skills but never mentions `globs` as a frontmatter option. Builders don't know their skill can auto-activate on specific file types.

**Why it matters:** Globs determine whether a skill triggers automatically when Claude encounters certain files. A builder creating an image-editor skill would benefit from `globs: **/*.png, **/*.jpg` but has no way to know this exists from this skill's instructions alone.

**Scoring impact:** Usability

**What good looks like:** The "Anatomy of a Skill" section documents `name` and `description` as required frontmatter but should also mention `globs` as an optional field that controls automatic activation. Even one sentence explaining what globs do and when to set them would close this gap.

**Read more:** Anthropic Claude Code docs — docs.anthropic.com/en/docs/claude-code

---

#### No Guidance on When to Keep vs. Split Content

**What happens:** The skill mentions that references files keep SKILL.md lean and that files over 10k words should include grep patterns, but there's no practical guidance on when content should live in SKILL.md vs. references vs. assets.

**Why it matters:** Builders either stuff everything into SKILL.md (making it bloated and slow to load) or split everything into references (making the skill fragmented and hard to follow). Clear thresholds help builders make this decision consistently.

**Scoring impact:** Edge Cases

**What good looks like:** Concrete rules of thumb: "If a section is under 500 words and essential to every run, keep it in SKILL.md. If it's reference material that's only needed sometimes, move it to references/. If it's over 2,000 words, it almost certainly belongs in references/." These aren't rigid rules — they're decision aids that prevent the most common mistakes.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

### P3 — Polish

- **Instructions not front-loaded:** The About Skills section (anatomy, progressive disclosure) comes before the actionable 6-step creation process. Builders who already know what skills are have to scroll past educational content to reach the workflow. *(Instructions)*
- **No version or changelog:** The skill will evolve but there's no version tracking. Builders and collaborators can't tell what changed between iterations. *(Usability)*
- **License reference without context:** Frontmatter says "license: Complete terms in LICENSE.txt" but the skill doesn't explain what this means for builders using or distributing it. *(Usability)*

---

## Bottom Line

This is a well-thought-out skill with a genuinely useful 6-step framework. The progressive disclosure concept is taught and practiced well, and the Step 2 analysis pattern is a smart way to help builders think about what belongs in a skill. The gap that holds it back is the gap it teaches others to avoid: no complete examples. The skill knows that examples are the most powerful tool for consistency (it literally says so in its own content) but doesn't include any for its own workflow. Fixing that one thing would move the needle more than anything else.

**Start with:** Add one complete example of a finished SKILL.md — even a short, simple one — to anchor what "done" looks like. This directly addresses both P1 findings and is the single highest-impact improvement.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
