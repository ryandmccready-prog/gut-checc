# Gut Checc Report

**Project:** Artifacts Builder Skill
**Mode:** Skill
**Score:** 27 / 100
**Date:** 2026-03-09

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Instructions | 9/25 | The build pipeline steps are clear and specific, but the actual creative work (Step 2: developing the artifact) has virtually no guidance — that's where the real skill lives. |
| Examples | 2/25 | Zero input/output examples anywhere in the Skill. The design anti-patterns ("no purple gradients") hint at what bad looks like but nothing shows what good looks like. |
| Edge Cases | 5/25 | One strong boundary (complex vs. simple artifacts) and one technical safeguard (Node 18+). Nothing for script failures, ambiguous requests, or dependency issues. |
| Usability | 11/25 | Good description with a clear trigger distinction. Logical step-by-step flow. But references a "Common Development Tasks" section that doesn't exist, and no globs are set. |

---

## What's Solid

- **The description nails the "when to use / when not to use" decision:** "Use for complex artifacts requiring state management, routing, or shadcn/ui components — not for simple single-file HTML/JSX artifacts" is exactly the kind of boundary that prevents a Skill from firing when it shouldn't. Most builders skip this entirely.
- **The anti-slop design guideline is specific and grounded:** "Avoid excessive centered layouts, purple gradients, uniform rounded corners, and Inter font" names the exact patterns to avoid. That's a real constraint that changes output, not a vague "make it look good."
- **The init script documentation tells Claude exactly what it gets:** The checklist of what the project includes (React + TS, Tailwind, 40+ shadcn components, path aliases, Parcel config) means Claude knows its toolkit from the start. No guessing.

---

## Findings

### 🔴 P1 — Fix First

#### Zero Examples

**What happens:** Claude has rules and a process but has never seen a complete run in action. Every time it builds an artifact, it's interpreting the instructions fresh — so the quality, structure, and design approach drift between uses.

**Why it matters:** Examples are the strongest anchoring tool in any Skill. One clear example of "user asked for X, here's the component structure and final artifact" would do more for consistency than all the procedural steps combined. Without any, Claude improvises the most important part (the actual building) every single time.

**Scoring impact:** Examples

**What good looks like:** Include at least two examples. One where a user asks for something straightforward (say, a dashboard with a few cards) and one where the request is more complex (a multi-page app with state). Show the key decisions: how the components were structured, which shadcn components were chosen and why, and what the final artifact looked like. These examples become the reference point Claude mirrors on every future run.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/multishot-prompting

---

#### Step 2 Is a Blank Check

**What happens:** The five-step process goes: init project (detailed), develop your artifact (one sentence), bundle (detailed), share (clear), test (optional). Step 2 — the part where the actual artifact gets built — says "edit the generated files. See Common Development Tasks below for guidance." But those Common Development Tasks don't exist in this file. The hardest, most important step is the one with no instructions.

**Why it matters:** This is like giving someone a fully equipped kitchen, detailed instructions for turning on the oven, a clear explanation of how to plate and serve — but nothing about how to actually cook. The build pipeline is solid. The design and development guidance is almost entirely missing. Claude knows how to set up the project and how to bundle it, but has no guidance on how to make good decisions in between.

**Scoring impact:** Instructions

**What good looks like:** Step 2 needs to be the longest section, not the shortest. Cover the principles that make artifacts good: how to break a request into components, when to reach for which shadcn components, how to structure state, how to handle layout for different content types. You don't need to cover everything — even a short section on "how to think about component structure" and "common patterns for different artifact types" would transform this Skill's usefulness.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### Design Guidance Is Anti-Patterns Only

**What happens:** The design section tells Claude what NOT to do (no purple gradients, no centered layouts, no rounded corners, no Inter font). But it never shows what TO do. Claude knows four things to avoid but has no positive design direction — no preferred color approaches, no layout philosophies, no typography guidance beyond "not Inter."

**Why it matters:** Constraints prevent bad output, but they don't produce good output. Telling Claude to avoid purple gradients doesn't tell it what color palette to use instead. Avoiding centered layouts doesn't tell it what layout approach to prefer. Without positive design direction, Claude avoids four specific traps but still has infinite choices for everything else — and it'll make different choices every time.

**Scoring impact:** Instructions, Examples

**What good looks like:** Pair the anti-patterns with positive guidance. Even something like "prefer left-aligned layouts with clear visual hierarchy" or "use muted, professional color palettes unless the user's request calls for something bold" gives Claude a direction, not just a list of walls. Better yet, show a before/after: "this is AI slop, this is what we're going for instead."

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

### 🟡 P2 — Strengthen

#### Happy Path Only — No Error Recovery

**What happens:** The Skill assumes every script runs perfectly, every dependency installs, and every build succeeds. What does Claude do when `init-artifact.sh` fails? When `npm install` errors out? When Parcel can't bundle because of a TypeScript error? When the user's Node version is too old despite the auto-detection?

**Why it matters:** Build tools fail. Dependencies conflict. Scripts hit edge cases. When something breaks mid-process, Claude currently has zero guidance on how to diagnose, recover, or communicate the problem to the user. It'll either guess at a fix, retry blindly, or just report the error without context.

**Scoring impact:** Edge Cases

**What good looks like:** Add a short troubleshooting section covering the most common failure points. "If init fails, check Node version. If bundle fails, check for TypeScript errors in the build output. If dependencies conflict, try clearing node_modules and reinstalling." A few lines of recovery guidance prevent a lot of confusion.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### No Guardrails for When This Stack Is Overkill

**What happens:** The description says "not for simple single-file HTML/JSX artifacts" — but where's the line? A user asks for an interactive chart. Is that "complex" enough for React + TypeScript + Vite + Parcel + Tailwind + shadcn/ui? Or would a single HTML file with a script tag handle it fine? The Skill has no guidance for the gray area between "clearly simple" and "clearly complex."

**Why it matters:** React 18 + TypeScript + Vite + Parcel + Tailwind + 40 shadcn components is a heavy stack. Spinning all that up for something that could be a single HTML file wastes time and adds complexity. Without guidance on when this level of tooling is actually warranted, Claude defaults to using the full stack every time the Skill triggers — even when a simpler approach would produce a better result faster.

**Scoring impact:** Edge Cases, Instructions

**What good looks like:** Define the boundary more specifically. Something like: "Use this stack when the artifact needs multiple interactive components, client-side routing, shared state between components, or heavy use of shadcn/ui. If it's a single view with basic interactivity, a simple HTML artifact is probably better." Give Claude permission to say "you don't need the full stack for this one."

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### Missing "Common Development Tasks" Section

**What happens:** Step 2 says "See Common Development Tasks below for guidance" but no such section exists in the Skill. Claude follows that reference and finds nothing.

**Why it matters:** This is either a section that was planned and never written, or it exists in a companion file that isn't referenced. Either way, Claude is being pointed to guidance that isn't there. For a builder reading this Skill, it's confusing. For Claude following it, it's a dead end at the exact moment it needs the most help.

**Scoring impact:** Usability, Instructions

**What good looks like:** Either add the Common Development Tasks section covering things like adding new components, structuring pages, managing state, and working with shadcn/ui — or remove the reference so Claude isn't looking for something that doesn't exist.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### No Constraints Beyond Design

**What happens:** The Skill tells Claude what to DO (run scripts, follow steps) but the only "don't" is the design anti-slop guideline. There are no behavioral constraints: don't over-engineer the component tree, don't add features the user didn't ask for, don't use external API calls in artifacts (since they run sandboxed), don't include dependencies beyond what's already installed.

**Why it matters:** Without constraints, Claude fills in the gaps with its own defaults. It might add a backend call that won't work in an artifact sandbox. It might create a deeply nested component tree when a flat structure would work fine. Telling Claude what NOT to do creates hard boundaries that prevent drift into patterns that cause real problems.

**Scoring impact:** Instructions

**What good looks like:** Add constraints specific to artifact building. Things like: "Don't add npm packages beyond what's already installed — work with the existing stack." "Don't create more than one level of nested routing unless the user specifically asks for it." "Don't make external API calls — artifacts run sandboxed and can't reach outside services." These prevent the most common artifact-building mistakes.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

### 🟢 P3 — Polish

- **Testing discouraged upfront for complex artifacts:** The Skill says "avoid testing the artifact upfront as it adds latency." For simple artifacts that's fine, but for complex multi-component builds with state management and routing, shipping without testing is risky. A note about when testing IS worth the latency would help Claude judge. *(Edge Cases)*
- **Reference section is a single URL:** Only links to shadcn/ui docs. No links to React patterns, Tailwind docs, Vite config, or Parcel troubleshooting. Claude can access these on its own, but explicit references help it prioritize the right sources. *(Usability)*
- **No version or changelog:** As this Skill evolves, there's no way to track what changed. A simple version line in the frontmatter helps future you know which iteration you're working with. *(Usability)*
- **Empty globs with no explanation:** No globs field set. For a build tool Skill this is likely intentional (manual trigger only), but being explicit about that choice helps anyone reading the Skill understand it was deliberate. *(Usability)*

---

## Bottom Line

The build pipeline is genuinely well-constructed — the init script, the bundle process, and the toolchain documentation are clear and specific. The description does a great job defining when to use this Skill vs. when not to. And the anti-slop design rule shows you're thinking about output quality, not just output existence. That's the right instinct.

The gap is that this Skill is 90% "how to run the tools" and 10% "how to build something good with them." It's like having detailed instructions for setting up a woodworking shop but nothing about how to actually make furniture. Step 2 — the part where the real work happens — is the shortest section when it should be the longest. Add development guidance, positive design direction, and two solid examples, and this becomes a fundamentally different Skill.

**Start with:** Write the missing Step 2 content. Cover how Claude should approach component structure, which shadcn components to reach for in common scenarios, and how to think about layout. Then add two complete examples showing a user request turned into a finished artifact. These two changes address the three P1 findings at once.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
