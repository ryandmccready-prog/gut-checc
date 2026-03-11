# Gut Checc Report

**Project:** OmniReply Pro v3.2.1
**Mode:** Skill
**Score:** 40 / 100
**Date:** 2026-03-09

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Instructions | 12/25 | Has strong specific constraints (forbidden words, word limits) but the tone matrix is all undefined adjectives that drift every run. |
| Examples | 7/25 | One example covering one scenario out of eight template categories — seven templates are completely unanchored. |
| Edge Cases | 12/25 | Five edge cases listed and an escalation protocol, but common real-world scenarios like unknown tier, non-English input, or no clear question are missing. |
| Usability | 9/25 | Globs are set well, but the description is buzzword soup that won't trigger naturally, and there's no guidance for the user on what to provide. |

---

## What's Solid

- **Real constraints exist:** The forbidden words list, emoji policy, word count limits, and signature rules are exactly the kind of specific boundaries that prevent drift. Most builders don't think to add these.
- **Edge cases got attention:** Having a dedicated edge cases section with five real scenarios (reply-all, out of office, internal forwards) shows the builder is thinking beyond the happy path. That instinct is valuable.
- **Templates provide structure:** Eight response templates across departments give Claude a structural skeleton to follow. The categories are well-chosen and cover real B2B SaaS communication needs.

---

## Findings

### 🔴 P1 — Fix First

#### Adjective-Heavy Tone Matrix

**What happens:** The tone calibration matrix uses undefined adjectives ("Ultra-formal," "Warm," "Conversational," "empathy-first") that Claude interprets differently every run.

**Why it matters:** Two emails with identical sentiment and tier will get responses in noticeably different tones because Claude has no anchor for what these words actually mean in practice.

**Scoring impact:** Instructions

**What good looks like:** Instead of saying "Ultra-formal, empathy-first," show Claude what that looks like. One sentence demonstrating ultra-formal tone and one demonstrating warm tone does more than the entire matrix. Specific beats vague every time. "Use sentences under 12 words, no contractions, acknowledge the impact before the solution" is actionable. "Ultra-formal" is a vibe.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### One Example for Eight Templates

**What happens:** There's one example at the bottom showing a support escalation response. The other seven template categories (feature request, how-to, inbound sales, post-demo follow-up, quarterly review, churn risk, billing) have zero demonstrated outputs.

**Why it matters:** Examples are the single most powerful consistency tool. One example cuts drift in half. Zero examples means Claude improvises every time. Seven undemonstrated templates means seven places where output quality is a coin flip.

**Scoring impact:** Examples

**What good looks like:** Each template category should have at least one fully realized input/output example — not the bracketed skeleton, but an actual email in and response out. Two examples per category (one straightforward, one tricky) would make this skill dramatically more consistent. Start with the two or three categories used most often.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Over-Engineered Architecture Creates Ambiguity

**What happens:** Six layers, twelve tone matrix cells, six classification dimensions, nine quality checks, and an escalation protocol. The "system architecture" framing creates a complex decision tree that Claude has to navigate before generating a single word.

**Why it matters:** Complexity doesn't equal quality in prompts. When layers conflict (does the tone matrix override the template structure? Does the quality check override the brand voice parameters?), Claude guesses. The more moving parts, the more places drift can creep in. A focused skill that does one thing well beats an ambitious skill that tries to handle every scenario.

**Scoring impact:** Instructions

**What good looks like:** The best skills start narrow and expand. Pick the one or two email categories that come up most often. Nail those with specific instructions, strong examples, and clear constraints. Once those are rock solid, add the next category. A skill that handles support emails perfectly is more valuable than one that handles eight categories inconsistently.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Description Field Won't Trigger Naturally

**What happens:** The description reads: "Enterprise-grade email response generator with multi-stakeholder awareness, sentiment-adaptive tone modulation, hierarchical priority classification, and brand voice compliance verification for B2B SaaS customer communications across support, sales, and success departments."

**Why it matters:** The description is what Claude uses to decide whether to activate a skill. Nobody asks Claude for "sentiment-adaptive tone modulation." They say "help me reply to this customer email" or "draft a response to this support ticket." A description full of buzzwords means the skill fires unpredictably or not at all.

**Scoring impact:** Usability

**What good looks like:** The description should contain the words a real person would actually say when they need this skill. Something like: "Drafts professional customer email responses for support tickets, sales inquiries, and account check-ins in B2B SaaS." Natural language that matches how people think about the task.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

### 🟡 P2 — Strengthen

#### Classification System Has No Anchoring

**What happens:** The six-dimension classification (urgency P0-P4, sentiment -5 to +5, stakeholder tier, department, communication stage, complexity index) requires Claude to make judgment calls with no examples showing how.

**Why it matters:** Is "I've been waiting two days" a P1 or P2? Is it sentiment -2 or -3? Without anchoring examples for each dimension, classification varies between runs, and different classifications produce different tones and templates. The whole system rests on a foundation of unanchored judgment.

**Scoring impact:** Instructions

**What good looks like:** Each classification dimension should have at least two anchor examples — one on each side of a meaningful boundary. "This is P1 because X. This is P2 because Y." That teaches Claude the decision-making principle, not just the labels.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Missing Common Edge Cases

**What happens:** The edge cases section covers reply-all, angry follow-ups, internal forwards, and out-of-office replies. But it doesn't address non-English emails, emails with no clear question, emails from unknown tiers, one-word emails, or emails that don't fit any template category.

**Why it matters:** The listed edge cases are good but they're the clever ones. The mundane ones — someone sends "thanks" or "ok" or an email in Spanish — are what actually trip up users day to day.

**Scoring impact:** Edge Cases

**What good looks like:** Think about the most boring, annoying input someone might paste in. The skill should handle those gracefully. Even a simple "If the email doesn't fit any category, say so and suggest the user draft manually" prevents confusion.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Trying to Do Everything at Once

**What happens:** The skill covers Support, Sales, Success, Billing, Legal, and Executive communications across Enterprise, Mid-Market, SMB, and Free tiers with five urgency levels. That's hundreds of potential combinations.

**Why it matters:** One skill stretched across all these scenarios will be mediocre at each of them. The tone differences between a churn risk email and a sales follow-up are significant enough that each really deserves focused attention, examples, and constraints.

**Scoring impact:** Edge Cases

**What good looks like:** The best V1 is a small V1. Ship the core use case — probably support responses — and expand once that's dialed in. Three templates done perfectly are worth more than eight done loosely.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### No User Instructions

**What happens:** The skill tells Claude how to behave in detail but never tells the user what to provide. What should they paste — the whole email thread? Just the latest message? Customer metadata? Their tier?

**Why it matters:** The classification system needs information (tier, department, communication stage) that may not be in the email itself. If the user doesn't know to provide this, Claude has to guess, and the whole classification system becomes unreliable.

**Scoring impact:** Usability

**What good looks like:** A brief section at the top telling the user: "Paste the customer email. Include their company name and plan tier if you know it. I'll draft a response." Sets expectations and ensures Claude gets the input it needs.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

### 🟢 P3 — Polish

- **Critical rules buried in later layers:** Forbidden words and quality checks sit in Layers 3 and 5. Claude weighs earlier instructions more heavily. These constraints should be near the top. *(Instructions)*
- **Templates are skeletons, not demonstrations:** The `[bracketed]` templates show structure but never show a fully filled version. Only the final example demonstrates what a real response looks like. *(Examples)*
- **Version inflation signals scope creep:** The "v3.2.1" versioning and changelog suggest the skill has been growing in complexity rather than sharpening in focus. Each version added a layer instead of tightening the core. *(Instructions)*

---

## Bottom Line

There's a builder here who clearly thinks deeply about communication — the category breakdowns, the tone awareness, the forbidden words list, and the edge case section all show real thoughtfulness. The gap isn't thinking. It's the trap of building for complexity instead of consistency. Right now, this skill tries to be a complete email platform in a prompt, and that ambition works against it. Narrowing the scope and adding real examples would turn a 40 into a 70+ without writing more — just writing sharper.

**Start with:** Add one fully realized input/output example for each of your top three most-used template categories. This single change will do more for consistency than all six layers combined.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
