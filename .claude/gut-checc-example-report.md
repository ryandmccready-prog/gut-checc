# Gut Checc Report

**Project:** LinkedIn Post Writer Skill
**Mode:** Skill
**Score:** 38 / 100
**Date:** 2026-03-09

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Instructions | 17/25 | Strong constraints and clear don'ts, but a couple of key instructions ("hook," "stories from experience") leave room for Claude to interpret freely. |
| Examples | 3/25 | Zero input/output examples anywhere in the Skill. |
| Edge Cases | 4/25 | No handling for empty input, vague ideas, off-topic requests, or unusually long input. |
| Usability | 14/25 | Output expectation is clear and the description is decent, but globs are empty and the description isn't optimized as a trigger phrase. |

---

## What's Solid

- **Hard constraints that actually prevent drift:** "No emojis," "no hashtags," "don't use buzzwords like synergy, leverage, disrupt," and "never start with I'm excited to announce" are exactly the kind of negative boundaries that keep Claude from sliding into its defaults. Most new builders forget to add these. You didn't.
- **Word count range is locked:** 150-250 words gives Claude a clear lane. Without this, posts would range from 50 to 500 words depending on the input. Small detail, big consistency win.
- **The Output section is clean:** "Just give me the post. No preamble, no 'here's your post,' no explanation." This is the right way to tell Claude what the deliverable looks like. Straight to the point.

---

## Findings

### P1 — Fix First

#### Zero Examples

**What happens:** Claude interprets every instruction from scratch each time, so post quality and style drift between runs.

**Why it matters:** Examples are the single most powerful anchoring tool in a Skill. One clear input/output example does more for consistency than ten lines of rules. Without any, your style rules are suggestions Claude guesses at rather than patterns it mirrors.

**Scoring impact:** Examples

**What good looks like:** Include at least two examples — one with a simple idea as input and the finished post as output, and one with rough bullet points as input and the finished post as output. These examples should demonstrate your actual style: the short paragraphs, the hook, the story, the ending question. When Claude can see the target, it hits it consistently. When it can't, it improvises.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### "Hook" and "Takeaway" Never Shown

**What happens:** The Skill says "start with a hook — something that makes people stop scrolling" and "end with a question or a takeaway." But it never shows what those look like in your voice. Claude knows what a hook is generically, but your hook style might be a bold claim, a surprising stat, a short story opener, or a contrarian take. Without seeing one, Claude picks a random approach each time.

**Why it matters:** When you describe a format in words but never show it, output shifts structurally between runs. One post starts with a question, the next with a stat, the next with a bold statement. Showing the exact structure you want produces consistent results. Describing it in words produces varied results.

**Scoring impact:** Instructions, Examples

**What good looks like:** Instead of just saying "start with a hook," show two or three hooks in your voice so Claude sees the pattern. Same for endings — show what a "question" ending and a "takeaway" ending look like in a real post. The examples you add for the first P1 finding naturally solve this one too, as long as those examples include your hook and ending style.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

### P2 — Strengthen

#### Happy Path Only

**What happens:** The Skill assumes it'll always get a clean idea or tidy bullet points. What happens when someone provides a single vague word? A full paragraph? A link? Three different topics at once? Nothing at all?

**Why it matters:** Real usage gets messy fast. The second or third time someone uses this Skill with input that doesn't look like the ideal case, Claude either freezes up, asks a bunch of questions, or just guesses. A few lines handling the obvious edge cases ("if the input is vague, ask one clarifying question" or "if multiple topics, pick the strongest one") go a long way.

**Scoring impact:** Edge Cases

**What good looks like:** Think about the three weirdest things someone might paste in and add a short instruction for each. You don't need to cover every possibility — just the obvious ones. "If the input is too vague to write a full post, ask one specific clarifying question. If the input contains multiple topics, focus on the one with the strongest hook potential."

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### "Stories From My Own Experience" Creates Identity Tension

**What happens:** The Skill says "I tell stories from my own experience" under My Style. But Claude doesn't have personal experiences. This creates ambiguity — should Claude fabricate experiences? Treat the user's input as the experience to narrate? Write in the user's voice and assume they'll fill in real details?

**Why it matters:** Without clarifying what this means in practice, Claude might invent fake anecdotes, write generic "I once worked with a client who..." filler, or just ignore the rule entirely. Any of those outcomes drift from what you probably want.

**Scoring impact:** Instructions

**What good looks like:** Clarify what this rule means for Claude specifically. Something like "Frame posts as first-person stories. Use the details from my input as the basis for the narrative. Don't invent specific experiences I didn't provide — build the story around what I gave you." This tells Claude exactly how to handle the storytelling style without making things up.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Empty Globs

**What happens:** The globs field in frontmatter is empty. This means the Skill only activates when manually invoked — it won't auto-trigger on any file type.

**Why it matters:** For a LinkedIn post writer, manual-only activation probably makes sense since you're providing ideas conversationally, not working with files. But leaving it empty without an intentional decision means a future builder (or future you) won't know if this was deliberate or an oversight.

**Scoring impact:** Usability

**What good looks like:** If manual-only is intentional, that's fine — this is a low-priority finding. If you ever wanted it to trigger on specific files (like a drafts folder), you'd set globs to match those files. The key principle is being intentional about activation rather than leaving it at the default.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

### P3 — Polish

- **Output rule buried at the bottom:** The "just give me the post, no preamble" instruction is arguably the most important behavior rule, but it's the last section. Claude weighs earlier instructions more heavily, so moving this up (or repeating the key point near the top) strengthens it. *(Instructions)*
- **No version or changelog:** As this Skill evolves, there's no way to track what changed and when. A simple version line in the frontmatter helps future you remember what iteration you're on. *(Usability)*
- **Description could be a stronger trigger:** "Generates LinkedIn posts from rough ideas or bullet points" works but doesn't contain the natural phrases someone would say. Something like "Turn my rough ideas or bullet points into a LinkedIn post in my voice" matches how people actually ask for this. *(Usability)*

---

## Bottom Line

You clearly know your voice and you've built real constraints to protect it — that's more than most first-time Skill builders do. The "don't" rules are strong and the word count lock is smart. The gap is that Claude has rules but no reference point. It knows what not to do but has never seen what "right" looks like in your style. Two solid examples would transform this Skill's consistency overnight.

**Start with:** Add two input/output examples. One with a simple idea, one with messy bullet points. Make sure they show your hook style, your short paragraphs, and your ending style. This single change will likely move your score by 15-20 points.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
