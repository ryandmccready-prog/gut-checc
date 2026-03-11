# Gut Checc Report

**Project:** AI Content Pipeline (test-cc-content-pipeline.py)
**Mode:** Claude Code Project
**Score:** 16 / 100
**Date:** 2026-03-09

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Functionality | 6/25 | Two of six functions work on the happy path — the rest are empty TODOs, and there's zero error handling on API calls. |
| Shareability | 2/25 | No README, no requirements.txt, no setup instructions — someone cloning this has to reverse-engineer everything from the source. |
| Security | 0/25 | API key and Buffer token hardcoded directly in source code — automatic zero. |
| Value | 8/25 | Content pipeline is a real recurring problem worth solving, but right now it barely saves time over asking Claude directly in chat. |

---

## Progress Since Last Run

**Previous Score:** 16 → **Current Score:** 16 (no change)

| Category | Before | After | Change |
|----------|--------|-------|--------|
| Functionality | 6/25 | 6/25 | 0 |
| Shareability | 2/25 | 2/25 | 0 |
| Security | 0/25 | 0/25 | 0 |
| Value | 8/25 | 8/25 | 0 |

**Still Open:**
- API Keys in Source Code — Still hardcoded on lines 17 and 18
- No Error Handling on API Calls — Still no try/except on either API call
- The TODO Graveyard — Four functions still empty
- Raw LLM Output Never Parsed — Still returning raw text
- No README — Still missing
- No Requirements File — Still missing
- Trying to Do Everything at Once — Scope unchanged

**New:**
- No new findings. The code hasn't changed since the last run.

---

## What's Solid

- **The idea is genuinely useful:** A pipeline that takes a topic and produces a blog post plus social content is a real workflow that content teams repeat weekly. You picked something worth building.
- **The API calls are correctly structured:** `generate_blog()` and `create_social_posts()` use the Anthropic SDK properly — model, max_tokens, message format all check out. The foundation is there.

---

## Findings

### P1 — Fix First

#### API Keys in Source Code

**What happens:** Lines 17-18 have an Anthropic API key and a Buffer token hardcoded directly in the Python file. The moment this code is shared, pushed to GitHub, or shown in a screenshot, those credentials are exposed.

**Why it matters:** Leaked API keys get exploited within minutes. Automated scrapers constantly scan GitHub for patterns like `sk-ant-`. Even placeholder keys teach the habit of putting secrets in code, and real ones tend to follow.

**Scoring impact:** Security (automatic zero)

**What good looks like:** Secrets live in a `.env` file that's listed in `.gitignore` so it never gets committed. Your code reads them with `os.environ.get("ANTHROPIC_API_KEY")` or a library like `python-dotenv`. You also include a `.env.example` file with the variable names (but no values) so someone cloning the project knows what they need to configure. The real keys never touch source code.

**Read more:** https://docs.anthropic.com/en/docs/initial-setup — Anthropic's setup docs cover API key handling

---

#### No Error Handling on API Calls

**What happens:** Both `generate_blog()` and `create_social_posts()` call the Anthropic API with no try/except wrapping. If the API is down, the key is invalid, you hit a rate limit, or the response comes back unexpected, the whole script crashes with a raw Python traceback.

**Why it matters:** External APIs fail. It's not if, it's when. A user hitting a rate limit sees `anthropic.RateLimitError` and a stack trace instead of "You're sending requests too fast — wait a minute and try again." That's the difference between a tool and a script.

**Scoring impact:** Functionality

**What good looks like:** Every external API call gets wrapped in error handling that catches the specific things that can go wrong — authentication errors, rate limits, network timeouts, unexpected responses. Each one gets a plain-language message telling the user what happened and what to do about it. The script should communicate, not crash.

**Read more:** https://docs.anthropic.com/en/api/errors — Anthropic's error codes and what they mean

---

### P2 — Strengthen

#### The TODO Graveyard

**What happens:** Four out of six functions are empty stubs with TODO comments — `schedule_to_buffer()`, `track_engagement()`, `generate_report()`, and `setup_database()`. The docstring at the top promises five features. Two actually work.

**Why it matters:** A project that promises five things and delivers two isn't a tool yet — it's a sketch. The gap between what the code says it does and what it actually does creates confusion for anyone looking at it (including future you).

**Scoring impact:** Functionality, Value

**What good looks like:** The best V1 is a small V1. If the project generates blogs and social posts, that's the scope. Name it that. Ship it. Add Buffer scheduling as V2 when it's actually built. A focused tool that does two things well beats an ambitious tool that promises five and delivers two. Your docstring and your code should tell the same story.

**Read more:** Established software engineering patterns — ship the core, expand later.

---

#### Raw LLM Output Never Parsed

**What happens:** Both API functions return `msg.content[0].text` directly and print it raw. The social posts function asks for "3 LinkedIn posts and 5 tweets" but the response comes back as one block of text with no way to split individual posts or feed them into scheduling.

**Why it matters:** The AI call is step one. Turning that into something usable is step two. Right now the pipeline stops at step one. If you ever want `schedule_to_buffer()` to work, it needs individual posts, not a wall of mixed content.

**Scoring impact:** Functionality, Value

**What good looks like:** After getting the LLM response, parse it into structured data. For social posts, that might mean asking Claude to return JSON with separate post objects (using a system prompt that enforces the format), or parsing the text into a list. The output of one function should be ready to be the input of the next function. That's what makes it a pipeline instead of two separate API calls.

**Read more:** https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering — structured output techniques

---

#### No README

**What happens:** There's no README file. No explanation of what this does, how to set it up, what dependencies it needs, or how to run it.

**Why it matters:** Without a README, this project is only usable by the person who wrote it. Someone cloning it has to read the source code to figure out what to install, what keys are needed, and what to expect.

**Scoring impact:** Shareability

**What good looks like:** A README with four things: what this does (one paragraph), how to set it up (numbered steps), what environment variables are needed, and how to run it. Takes ten minutes to write and makes the project accessible to anyone.

**Read more:** Established open source documentation patterns.

---

#### No Requirements File

**What happens:** The project imports `anthropic` and `requests` (both third-party packages) but there's no `requirements.txt` declaring them as dependencies.

**Why it matters:** Someone cloning this project has to read through imports and guess which ones are third-party. `anthropic` and `requests` aren't built-in — they need to be installed. Without a requirements file, the setup process starts with guesswork.

**Scoring impact:** Shareability

**What good looks like:** A `requirements.txt` file listing every third-party package with pinned versions. That way `pip install -r requirements.txt` gets someone from zero to running in one command.

**Read more:** pip.pypa.io/en/stable/reference/requirements-file-format/

---

#### Trying to Do Everything at Once

**What happens:** The docstring promises blog generation, social post creation, Buffer scheduling, engagement tracking, and weekly reports. That's five distinct features across content creation, API integration, data collection, and analytics. Two work.

**Why it matters:** Scope ambition is great for a roadmap. Not great for a V1. Each unfinished feature makes the project feel incomplete instead of focused. And the architecture you'd need for engagement tracking (database, API polling, time series) is totally different from content generation.

**Scoring impact:** Value

**What good looks like:** Build the content generation piece. Make it solid — error handling, structured output, save results to a file. Ship that as V1. Add Buffer scheduling as V2. Engagement tracking as V3. Each version should be complete and useful on its own.

**Read more:** Established software engineering patterns — the best V1 is a small V1.

---

### P3 — Polish

- **No .gitignore:** When this goes to GitHub, `.env`, `__pycache__`, `content_pipeline.db`, and other local artifacts go with it. Easy to overlook but important before sharing. *(Security, Shareability)*
- **Model hardcoded:** `claude-sonnet-4-20250514` is hardcoded in two places (lines 22, 30). When a newer model ships, you'd need to find and edit both. A config variable at the top makes this a one-line change. *(Functionality)*
- **No input validation:** `topic = input(...)` accepts anything — empty string, a novel-length paste, whatever. A quick check that the topic isn't empty catches the most common user mistake. *(Functionality)*
- **Print statements as logging:** All status updates use `print()`. Works now, but if you ever need to control verbosity or debug issues in a longer pipeline, proper logging gives you that flexibility. *(Functionality)*

---

## Bottom Line

You picked a genuinely useful problem — content pipelines are exactly the kind of repeatable workflow worth automating. The Anthropic API calls are structured correctly, which means the fundamentals are there. Nothing has changed since the last run, so the same two things are still blocking progress: get the API keys out of source code (security), and add error handling around the API calls (resilience). Those two changes would roughly double your score.

**Start with:** Get the API keys out of source code and into environment variables. That's the single highest-impact change because it fixes the security zero AND makes the project shareable. Everything else builds from there.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
