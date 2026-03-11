# Gut Checc Report

**Project:** Tailored Resume Generator
**Mode:** Skill
**Score:** 54 / 100
**Date:** 2026-03-09

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Instructions | 16/25 | Detailed and mostly specific steps, but some vague adjectives and critical rules buried near the bottom. |
| Examples | 8/25 | One full example exists, but it actively teaches Claude to fabricate metrics and personal details — undermining the skill's own truthfulness rule. |
| Edge Cases | 13/25 | Handles career changers and seniority levels, but no guidance for inadequate input, mismatched experience, or conflicting info. |
| Usability | 17/25 | Good description, clear how-to section, and helpful tips — but the skill's density (346 lines, 10 instruction steps) creates a lot to absorb. |

---

## What's Solid

- **Thorough instruction steps:** The 10-step process from gathering info through strategic recommendations gives Claude a clear sequence to follow. The action verb formula (Action + What + How/Why + Result) is exactly the kind of specific formatting rule that produces consistent output.
- **Real constraints exist:** The "Don't" section under Best Practices sets hard boundaries — no first-person pronouns, no fabricating experience, no exceeding 2 pages. These are the kinds of negative constraints that prevent drift better than positive instructions alone.
- **Multiple user scenarios:** Career changers, recent graduates, senior executives, technical roles, and creative roles each get specific handling guidance. That's more scenario coverage than most Skills include.

---

## Findings

### P1 — Fix First

#### Example Teaches Claude to Fabricate

**What happens:** The example output adds metrics ("reduced processing time by 60%", "improved ROI by 35%", "50+ SQL queries"), job details ("for executive team"), and full contact info (name, email, phone, LinkedIn) that were never in the user's input. Claude learns from this example to invent numbers and embellish experience on every run.

**Why it matters:** Examples are the single most powerful behavioral anchor in a Skill. They outweigh written instructions. The skill says "Be truthful and accurate — never fabricate experience" but the example directly contradicts this by fabricating metrics and personal details. Claude will follow what you SHOW over what you TELL, and right now you're showing it to make things up.

**Scoring impact:** Examples

**What good looks like:** An example should only include information that was explicitly provided in the input. If the user said "Built Python scripts for data automation," the example output should say something like "Built Python automation scripts for data processing workflows" — rephrased and strengthened, but not fabricated. Metrics should only appear if the user provided them. If contact info isn't in the input, the example should use placeholders like "[Your Name]" and "[Your Email]" to show Claude that it should prompt the user for this info rather than invent it. The golden rule: every claim in the output should trace back to something in the input.

**Read more:** docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/multishot-prompting

---

### P2 — Strengthen

#### Overengineered Scope

**What happens:** The skill tries to be a resume generator, gap analyzer, interview prep coach, and cover letter advisor all in one. Ten instruction steps, five user-type specializations, three format options, and 346 lines total. Claude has to hold all of this in working memory on every run.

**Why it matters:** More instructions doesn't mean better instructions. When a skill tries to do everything, consistency drops because Claude can't reliably follow 10 complex steps the same way every time. A focused resume generator that nails the core task would outperform this broader version. The interview tips, cover letter hooks, and strategic recommendations could each be their own skill.

**Scoring impact:** Instructions

**What good looks like:** A strong V1 skill does one thing extremely well. For a resume generator, that's: analyze the job description, map the user's experience to it, and produce a tailored resume in a consistent format. The "extras" (gap analysis, interview prep, cover letter hooks) are great ideas — for separate skills or as opt-in additions, not mandatory steps. Think about what the user actually needs every single time they use this, and make that the core.

**Read more:** docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### Critical Constraints Buried at the Bottom

**What happens:** The "Don't" list and the most important behavioral rule ("Be truthful and accurate — never fabricate experience") are buried under Step 9, near the bottom of a 346-line document. Claude reads top to bottom and gives more weight to earlier instructions.

**Why it matters:** If the truthfulness rule were at the top — before the instructions, before the example — it would act as a hard constraint on everything that follows. Buried at line 289, it's more of an afterthought that Claude may not weigh as heavily. Front-loading critical rules is one of the simplest, highest-impact changes you can make to any skill.

**Scoring impact:** Instructions

**What good looks like:** Put your most critical constraints right at the top of the Instructions section, before the step-by-step process. Something like: "CRITICAL RULES — apply to every resume generated: Never fabricate metrics, achievements, or experience not provided by the user. Never invent contact details. Never use first-person pronouns." Then the 10-step process follows underneath, operating within those hard boundaries. Critical rules first, process details second.

**Read more:** docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### Happy Path Gaps

**What happens:** The skill handles different user types well but doesn't address what Claude should do when input is seriously inadequate — like a user who provides three bullet points and a job title, or when the user's experience genuinely doesn't match the target role at all.

**Why it matters:** Real users don't always provide clean, complete input. The skill says "Request the full job description if not provided" but doesn't specify how hard to push, what minimum information Claude needs to proceed, or what to do when experience and requirements have almost zero overlap. By the third or fourth use, someone will hit these gaps.

**Scoring impact:** Edge Cases

**What good looks like:** Add a "Minimum Requirements" section that tells Claude: "You need at least X to generate a useful resume. If the user provides less, ask for specifically Y and Z before proceeding." Also address the mismatch scenario: "If the user's experience has less than 30% overlap with the job requirements, be upfront about the gap and focus on transferable skills rather than trying to force-fit unrelated experience." These don't need to be elaborate — a few sentences each prevents confusion.

**Read more:** docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

#### Output Length Undefined in Concrete Terms

**What happens:** The skill says "Keep to 1 page for <10 years experience, 2 pages for 10+ years" but never defines what one page means in markdown. How many bullet points per role? How long is the summary? Without concrete limits, output length will drift between runs.

**Why it matters:** "One page" is a print concept that doesn't translate directly to markdown. Claude has no way to enforce page length. Some runs will produce 15 bullet points per role, others will produce 5. The user ends up manually trimming or padding, which eats the time savings.

**Scoring impact:** Edge Cases

**What good looks like:** Replace the page-length concept with concrete limits: "Professional Summary: 3-4 sentences maximum. Per role: 4-6 bullet points for relevant roles, 2-3 for less relevant ones. Skills section: no more than 4 categories with 4-5 items each. Total resume: under 500 words for early career, under 800 words for 10+ years." Now Claude has measurable constraints it can actually follow.

**Read more:** docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---

### P3 — Polish

- **Usage examples show input but not output:** The Career Transition and With Existing Resume usage patterns show what to provide but not what Claude would produce for those scenarios. Adding even a partial output example for each would anchor behavior across use cases. *(Examples)*
- **Multiple formats mentioned, only one shown:** Markdown, Plain Text, and Word/PDF formats are listed as options, but only the Markdown format appears in the example. Claude may improvise the others. *(Examples)*
- **No version or changelog:** The skill will evolve but there's no way to track what changed between versions. Even a simple "v1.0 — initial release" note helps. *(Usability)*
- **Globs intentionally empty (just confirm):** Fine for a manual-trigger skill like this. Just make sure that's a choice, not an oversight. *(Usability)*

---

## Bottom Line

This skill shows real thought and ambition — the 10-step process, the special handling for different career stages, and the ATS optimization guidance all show a builder who understands the problem space deeply. The honest read: the example is actively working against you right now by teaching Claude to fabricate, and the skill is trying to do too many things at once which makes consistent output harder to achieve. Both are very fixable.

**Start with:** Fix the example so every metric, detail, and piece of contact info traces directly back to the user's input. This is the highest-impact change because examples are the strongest behavioral anchor — fix this one thing and the entire skill gets more honest and consistent overnight.

---

## Next Steps

1. Address the P1 finding — clean the example of fabricated data
2. Consider front-loading critical constraints (truthfulness rule at the top)
3. Run Gut Checc again after changes
4. Compare your new score to this report

*Save this report so you can track your progress across runs.*
