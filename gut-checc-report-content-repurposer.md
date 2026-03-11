# Gut Checc Report

**Project:** Weekly Content Repurposer
**Mode:** Coworker
**Score:** 37 / 100
**Date:** 2026-03-09

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Task Clarity | 8/25 | Four outputs are named but none have defined formats, lengths, or structure. |
| Reliability | 5/25 | Vague adjectives and zero examples mean output drifts every single run. |
| Time Value | 18/25 | Content repurposing is a genuinely recurring task with real time savings. |
| Handoff Ready | 6/25 | "My voice" is referenced but never defined — only the builder knows what that means. |
| Integration Health | N/A | No external integrations. Points redistributed across other categories. |

---

## What's Solid

- **The concept is real:** Weekly content repurposing across platforms is exactly the kind of repeatable task Coworkers are made for. This isn't a toy project — it solves a genuine weekly pain point.
- **Multi-platform thinking:** Recognizing that LinkedIn, Twitter, and email need different treatment shows smart platform awareness. Most builders try to create one piece and blast it everywhere.

---

## Findings

### P1 — Fix First

#### Vague Adjective Instructions

**What happens:** "Professional but engaging," "punchy," and "feel personal" are open to interpretation. Claude reads those differently every run. One time you get a 3-sentence LinkedIn post, next time you get a 3-paragraph essay. Both technically qualify as "professional but engaging."

**Why it matters:** Every vague adjective is a coin flip. You'll spend time reformatting output that should have been right the first time, eating the time savings this Coworker is supposed to create.

**Scoring impact:** Reliability, Task Clarity

**What good looks like:** Replace adjectives with specific behaviors. Instead of "professional but engaging," define what that means: "Use short sentences. No jargon. Open with a surprising stat or question. Write in first person. Keep under 200 words." Now Claude can't drift because every instruction is verifiable — you can look at the output and check each rule. The more specific the instruction, the more consistent the result.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Zero Examples

**What happens:** There are no input/output examples anywhere. Without seeing what a good LinkedIn post or Twitter thread looks like for YOUR content, Claude improvises from scratch every time. It has no anchor.

**Why it matters:** One clear example does more for consistency than ten lines of instructions. This is the single highest-impact improvement you can make. Two examples covering different blog post types (say, a how-to vs. an opinion piece) would nearly eliminate drift.

**Scoring impact:** Reliability, Task Clarity

**What good looks like:** Include at least one complete example showing: "Here's a blog post paragraph" followed by "Here's the LinkedIn post I want from it," "Here's the Twitter thread," and "Here's the email intro." The example becomes the template Claude mirrors. Two examples covering different scenarios make the pattern unmistakable.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Output Format Never Shown

**What happens:** The Coworker lists four outputs (LinkedIn post, Twitter thread, email intro, quotes) but never shows what any of them should actually look like. How long is the LinkedIn post? How many tweets in the thread? What structure should the email intro follow? All undefined.

**Why it matters:** Describing a format in words produces inconsistent results. Showing the exact template produces consistent results. Right now Claude is guessing the shape of every output, and that shape will change every run.

**Scoring impact:** Task Clarity, Reliability

**What good looks like:** Show the exact template for each output. For the Twitter thread: how many tweets, what goes in tweet 1 vs. tweet 5, max character count per tweet. For the LinkedIn post: target word count, whether to include a hook question, whether to use line breaks for readability. The template IS the spec. When Claude can see the structure you want, it mirrors it.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

### P2 — Strengthen

#### Output Format Not Locked Down

**What happens:** Without structural requirements, the LinkedIn post could be a paragraph or a mini-essay. The Twitter thread could be 3 tweets or 12. The email intro could be two sentences or two paragraphs. Every run is a surprise.

**Why it matters:** If you're reformatting output every time, the Coworker isn't saving you time — it's just moving the work around. Locked format means you can copy-paste and go.

**Scoring impact:** Reliability

**What good looks like:** Define structural rules for each output: word count ranges, number of items (e.g., "Twitter thread: exactly 5 tweets"), section structure, and any formatting requirements (headers, bullet points, emojis or no emojis). The tighter the format spec, the more consistent the output.

**Read more:** Task automation principles and Claude patterns

---

#### No Edge Case Handling

**What happens:** The Coworker assumes every blog post is the same shape. What if the blog post is 200 words? What about 5,000 words? What if it's a listicle? A technical tutorial? A personal story? Each of these needs different treatment.

**Why it matters:** The second or third time you use this with a blog post that's different from what you had in mind when you built it, you'll get output that doesn't quite fit. That's when builders lose trust in their own tools.

**Scoring impact:** Task Clarity, Reliability

**What good looks like:** Address at least the most common variations: short vs. long posts, different content types, posts with lots of data vs. narrative posts. Even a simple "If the blog post is under 300 words, generate a single tweet instead of a thread" shows Claude you've thought about this.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### No Constraints or "Don'ts"

**What happens:** The Coworker only has positive instructions — what TO do. There's nothing about what NOT to do. No banned patterns, no off-limits behaviors.

**Why it matters:** Without constraints, Claude defaults to its own patterns. It might add hashtags you hate, use emoji you'd never use, start every LinkedIn post with "I'm excited to share..." or add CTAs you didn't ask for. Constraints create the boundaries that keep output in your lane.

**Scoring impact:** Reliability, Task Clarity

**What good looks like:** Add explicit don'ts: "Never use hashtags on LinkedIn." "Don't start tweets with 'Thread:'" "No generic CTAs like 'What do you think?'" "Never use the phrase 'In this article.'" Negative constraints are often more powerful than positive instructions because they create hard walls instead of soft suggestions.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### "My Voice" With No Voice Definition

**What happens:** "Keep my voice consistent across all platforms" is a strong instruction with zero backing. Claude has no samples, no style guide, no tone markers. It doesn't know what your voice sounds like.

**Why it matters:** Without a defined voice, Claude uses its own default voice — which is helpful and slightly formal. If your actual voice is casual, irreverent, data-heavy, or storytelling-focused, you won't see that reflected.

**Scoring impact:** Handoff Ready, Reliability

**What good looks like:** Define your voice with specific traits: "I write in short, direct sentences. I use metaphors from sports and cooking. I never use exclamation points. I curse occasionally. I always back up opinions with data." Or include 2-3 paragraphs of your actual writing as a style reference. The more specific the voice definition, the more it sounds like you.

**Read more:** Task automation principles and Claude patterns

---

### P3 — Polish

- **No output delivery guidance:** The Coworker generates content but doesn't say where it goes — copy-paste to LinkedIn? Into a scheduling tool? A Notion page? The output exists in a vacuum. *(Handoff Ready)*
- **Builder-only context:** "My blog posts" and "my voice" assume context only the builder has. A teammate or VA couldn't pick this up and run it. *(Handoff Ready)*
- **No documentation or workflow notes:** No explanation of when this runs, what the weekly cadence looks like, or how it fits into a broader content workflow. *(Handoff Ready)*

---

## Bottom Line

You picked a genuinely useful task to automate — weekly content repurposing across platforms is exactly the kind of thing Coworkers are built for, and the multi-platform awareness is smart. Right now though, the instructions are too loose for Claude to deliver consistent results. The gap between "what you want" and "what Claude knows about what you want" is wide, and that gap shows up as drift every time you run it.

**Start with:** Add one complete input/output example. Take a real blog post you've already repurposed manually and show Claude exactly what the LinkedIn post, Twitter thread, and email intro should look like. This single change will have more impact on consistency than anything else you could do.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
