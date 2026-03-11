# Gut Checc Report

**Project:** AI Content Pipeline
**Mode:** Claude Code Project
**Score:** 16 / 100
**Date:** 2026-03-09

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Functionality | 6/25 | Two of five promised features work on the happy path — the rest are empty TODOs with zero error handling. |
| Shareability | 2/25 | No README, no requirements.txt, no setup instructions — only runs on the builder's machine as-is. |
| Security | 0/25 | API keys hardcoded directly in source code — automatic zero per security rubric. |
| Value | 8/25 | Content pipeline is a real problem worth solving, but right now it doesn't save much time over asking Claude in chat directly. |

---

## What's Solid

- **The idea is genuinely useful:** A pipeline that goes from topic to blog to social posts to scheduling is a real workflow that content teams repeat weekly. You picked a problem worth automating.
- **The API calls work:** `generate_blog()` and `create_social_posts()` are correctly structured Anthropic API calls. The bones are right — the model, max_tokens, and message format are all valid.

---

## Findings

### P1 — Fix First

#### API Keys in Source Code

**What happens:** Lines 17-18 have an Anthropic API key and Buffer token hardcoded directly in the Python file. The moment this code is shared, pushed to GitHub, or shown to anyone, those credentials are exposed.

**Why it matters:** Leaked API keys get exploited within minutes. Automated scrapers scan GitHub for exactly this pattern. Even "placeholder" keys teach the habit of putting secrets in source code, and the next version might have real ones.

**Scoring impact:** Security (automatic zero)

**What good looks like:** Secrets live in a `.env` file that's listed in `.gitignore` so it never gets committed. Your code reads them with `os.environ.get("ANTHROPIC_API_KEY")` or a library like `python-dotenv`. You include a `.env.example` file with the variable names (but no values) so someone cloning the project knows what they need to set up. The real keys never touch the code.

**Read more:** https://docs.anthropic.com/en/docs/initial-setup — Anthropic's setup docs cover API key handling

---

#### No Error Handling on API Calls

**What happens:** Both `generate_blog()` and `create_social_posts()` call the Anthropic API with zero try/except wrapping. If the API is down, your key is invalid, you hit a rate limit, or the response comes back weird, the entire script crashes with a raw Python traceback.

**Why it matters:** External APIs fail. It's not a question of if but when. A user running this for the first time with a bad key gets a wall of error text instead of "Hey, your API key doesn't seem to work." That's the difference between "this tool is broken" and "I know what to fix."

**Scoring impact:** Functionality

**What good looks like:** Every external API call gets wrapped in error handling that catches the specific things that go wrong — authentication errors, rate limits, network timeouts, unexpected responses. Each one gets a plain-language message telling the user what happened and what to do about it. The script shouldn't crash; it should communicate.

**Read more:** https://docs.anthropic.com/en/api/errors — Anthropic's error codes and what they mean

---

### P2 — Strengthen

#### The TODO Graveyard

**What happens:** Four out of six functions are empty with TODO comments — `schedule_to_buffer()`, `track_engagement()`, `generate_report()`, and `setup_database()`. The docstring at the top promises five features. Two of them actually work.

**Why it matters:** A project that promises five things and delivers two isn't a tool yet — it's a sketch. Anyone who reads the docstring expects a pipeline. What they get is a blog generator and a social post generator with three pass statements. The gap between what the project says it does and what it actually does creates confusion.

**Scoring impact:** Functionality, Value

**What good looks like:** The best V1 is a small V1. If the project generates blogs and social posts, that's the scope. Name it that. Ship it. Add Buffer scheduling as V2 when it's actually built. A focused tool that does two things well is more valuable than an ambitious tool that promises five things and delivers two. Your docstring and your code should tell the same story.

**Read more:** Established software engineering patterns — ship the core, expand later.

---

#### Raw LLM Output Never Parsed

**What happens:** Both API functions return `msg.content[0].text` directly and print it raw. The social posts function asks for "3 LinkedIn posts and 5 tweets" but the response comes back as one block of text. There's no parsing to split them into individual posts, no structure, no way to feed them into the next step of the pipeline.

**Why it matters:** The AI call is step one. Turning that into something usable is step two. Right now the pipeline has a step one and no step two. If you ever want `schedule_to_buffer()` to work, it needs individual posts — not a wall of text.

**Scoring impact:** Functionality, Value

**What good looks like:** After getting the LLM response, parse it into structured data. For social posts, that might mean asking Claude to return JSON with separate post objects, or parsing the text response into a list. The output of one function should be ready to be the input of the next function. That's what makes it a pipeline instead of two separate API calls.

**Read more:** https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering — structured output techniques

---

#### No README

**What happens:** There's no README file. No explanation of what this does, how to set it up, what dependencies are needed, or how to run it.

**Why it matters:** Without a README, this project is only usable by the person who wrote it. Someone cloning it has to read the source code to figure out what to install, what API keys are needed, and how to run it. That's a wall that stops most people.

**Scoring impact:** Shareability

**What good looks like:** A README with four things: what this does (one paragraph), how to set it up (numbered steps), what environment variables are needed, and how to run it. Takes ten minutes to write and makes the project accessible to anyone.

**Read more:** Established open source documentation patterns.

---

#### No Requirements File

**What happens:** The project imports `anthropic` and `requests` (both third-party packages) but there's no `requirements.txt` declaring them as dependencies.

**Why it matters:** Someone cloning this project has to read the imports to figure out what to install, then guess the right package names. `pip install anthropic requests` isn't hard, but they shouldn't have to figure that out themselves.

**Scoring impact:** Shareability

**What good looks like:** A `requirements.txt` file listing every third-party package the project needs, ideally with pinned versions so it works the same way six months from now as it does today.

**Read more:** Python packaging documentation — pip.pypa.io/en/stable/reference/requirements-file-format/

---

#### Trying to Do Everything at Once

**What happens:** The docstring promises blog generation, social post creation, Buffer scheduling, engagement tracking, and weekly reports. That's five distinct features. Two work. Three are empty.

**Why it matters:** Scope ambition is great for a roadmap but not for a V1. The unfinished features make the project feel incomplete instead of focused. And the architecture decisions you'd need for engagement tracking and reporting are different from what you need for content generation — trying to plan for all five at once makes each one harder.

**Scoring impact:** Value

**What good looks like:** Build the content generation piece. Make it solid — error handling, structured output, maybe save results to a file. Ship that. Then add Buffer scheduling as V2. Each version should be complete on its own.

**Read more:** Established software engineering patterns — the best V1 is a small V1.

---

### P3 — Polish

- **No .gitignore:** When this goes to GitHub, `.env`, `__pycache__`, `content_pipeline.db`, and other local artifacts go with it. Easy to overlook but important before sharing. *(Shareability)*
- **Model hardcoded:** `claude-sonnet-4-20250514` is hardcoded in two places. When a newer model ships, you'd need to edit both. A config variable at the top makes this a one-line change. *(Functionality)*
- **No input validation:** `topic = input(...)` goes straight to the API. Empty string, a novel-length paste, anything goes. A quick check that the topic isn't empty would catch the most common mistake. *(Functionality)*
- **Print statements as logging:** All status updates use `print()`. Works fine now, but if you ever need to control verbosity or debug issues, proper logging gives you that flexibility. *(Functionality)*

---

## Bottom Line

You picked a genuinely useful problem to solve — content pipelines are exactly the kind of repeatable workflow that's worth automating. The Anthropic API calls are structured correctly, which means you understand the fundamentals. The main thing holding this back is that the project promises more than it delivers right now, and the security basics (keys out of code, error handling on API calls) need to be in place before anything else.

**Start with:** Get the API keys out of source code and into environment variables. That's the single most important change because it's both a security fix and it makes the project shareable. Everything else builds on top of that.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
