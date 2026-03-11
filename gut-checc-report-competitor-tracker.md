# Gut Checc Report

**Project:** Competitor Mention Tracker
**Mode:** Claude Code Project
**Score:** 76 / 100
**Date:** 2026-03-09

---

## Progress Since Last Run

**Previous Score:** 75 (5 categories at /20) → **Current Score:** 76 / 100 (4 categories at /25, corrected rubric)

Note: The previous report used a non-standard 5 categories at 20 points each. This run uses the correct Claude Code Project rubric: 4 categories at 25 points each. Treat this as the clean baseline going forward.

---

## Score Breakdown

| Category | Score | Notes |
|----------|-------|-------|
| Functionality | 20/25 | Works reliably with thoughtful error handling across file I/O, API calls, and JSON parsing. Loses points for fragile JSON extraction and no validation of individual result objects. |
| Shareability | 16/25 | Excellent docstring with numbered setup steps and usage examples. Clean CLI with argparse. Loses points because supporting files (requirements.txt, .env.example) aren't visible alongside the code. |
| Security | 19/25 | API key properly loaded from environment variable. No secrets in code. References .env.example. Can't verify .gitignore or .env.example actually exist. |
| Value | 21/25 | Clear recurring use case in competitive intelligence. Structured CSV output feeds directly into analysis workflows. Configurable competitor list and output path with smart defaults. |

---

## What's Solid

- **Overwrite protection that most builders skip:** The CSV writer checks if the output file exists and appends a timestamp instead of destroying previous results. This shows real care for the user's data.
- **Layered error handling:** Catches JSONDecodeError, APIError, and general exceptions separately with clear messages. File reads check for existence and emptiness. This is above average.
- **Configurable where it counts:** Competitor list, output path, and model are all configurable via CLI args or env vars. Good defaults so it works immediately, with flexibility when you need it.
- **Clean CLI design:** argparse with help text, default values, and usage examples in the docstring. Someone can pick this up and use it right away.

---

## Findings

### P1 — Fix First

#### JSON Code Fence Fragility

**What happens:** Claude frequently wraps JSON responses in markdown code fences (` ```json ... ``` `), even when told "Return ONLY valid JSON." When that happens, `json.loads()` fails, the JSONDecodeError catch triggers, and the tracker reports zero mentions — even though the actual data was right there inside the fences.

**Why it matters:** This isn't a rare edge case. It's one of the most common LLM response behaviors. In real usage, this causes intermittent "no mentions found" failures that look confusing because the error preview on line 109 shows what looks like valid JSON wrapped in fences.

**Scoring impact:** Functionality

**What good looks like:** Clean the response before parsing. Check if the response starts with a code fence marker and extract the content between them. Some builders find the first `[` and last `]` to isolate the JSON array. The key principle: LLM output is raw material. Always clean it before parsing it. This is the difference between a tool that works in testing and one that works in real life.

**Read more:** Anthropic prompt engineering docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### Result Objects Not Validated Before CSV

**What happens:** The code checks that Claude's response is a JSON array (`isinstance(results, list)`) but doesn't check that each item has the expected keys: competitor, quote, speaker, sentiment, context. If Claude returns an object with an extra key like "confidence", `csv.DictWriter` raises a ValueError and the whole run crashes. If Claude omits a key, the CSV gets silent empty cells.

**Why it matters:** This works perfectly in testing because Claude returns exactly what you expect. It breaks in production when varied transcripts cause Claude to add or skip fields. One unexpected key crashes everything. One missing key silently loses data.

**Scoring impact:** Functionality

**What good looks like:** After parsing the JSON array, validate each item against the expected keys. Filter out items that don't match and log a warning so the user knows something was skipped. For DictWriter specifically, the `extrasaction='ignore'` parameter handles unexpected keys gracefully. The principle: never trust LLM output structure. Validate every item before using it.

**Read more:** Python csv.DictWriter docs — docs.python.org/3/library/csv.html#csv.DictWriter

---

### P2 — Strengthen

#### Supporting Files Not Included

**What happens:** The docstring references `.env.example` and `requirements.txt` as setup steps, but only the single Python file was submitted. If those files don't exist alongside the code, someone following the setup instructions hits a dead end at step 1.

**Why it matters:** The gap between "works on my machine" and "works on anyone's machine" is usually just three small files: requirements.txt, .env.example, and a README. The docstring already contains great setup instructions — they just need to also exist as standalone files so someone cloning the repo finds them where they expect.

**Scoring impact:** Shareability, Security

**What good looks like:** Three files complete the picture. `requirements.txt` with `anthropic` pinned to a version. `.env.example` with `ANTHROPIC_API_KEY=your-key-here` and `CLAUDE_MODEL=claude-sonnet-4-20250514` as a template. A brief `README.md` that consolidates the setup steps from the docstring. The docstring is already doing the heavy lifting — these files just meet people where they look.

**Read more:** Python packaging user guide — packaging.python.org/en/latest/guides/

---

#### No System Message in the API Call

**What happens:** All instructions — role, format constraints, the competitor list, and the transcript — go into a single user message. There's no system message to establish Claude's role or anchor behavioral constraints like "always return valid JSON."

**Why it matters:** System messages set durable behavior. User messages set the task. When everything is in one user message, Claude treats "Return ONLY valid JSON" with the same weight as the transcript content. A system message that says "You are a structured data extractor. Always return valid JSON arrays. Never include markdown formatting or commentary" makes that constraint stickier and more reliable.

**Scoring impact:** Functionality

**What good looks like:** Split the API call into a system message for role and constraints, and a user message for the task and data. The system message handles "who you are and how you behave." The user message handles "here's the transcript, here are the competitors, go." This separation is a best practice in the Anthropic API and improves consistency across varied inputs.

**Read more:** Anthropic messages API docs — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

#### No Verification That Mentions Exist in the Transcript

**What happens:** The tracker trusts Claude's results completely. There's no post-check to verify that a reported competitor mention actually appears in the transcript text. If Claude hallucinates a mention or confuses context, the CSV includes it as fact.

**Why it matters:** LLMs occasionally generate plausible results that aren't grounded in the input. For a tool that's building competitive intelligence, a hallucinated mention could lead to bad business decisions. Even a simple check — does this competitor name actually appear in the transcript? — adds a meaningful trust layer.

**Scoring impact:** Functionality, Value

**What good looks like:** After getting results from Claude, do a basic sanity check: for each result, verify the competitor name appears somewhere in the transcript (case-insensitive). Flag or filter results where the competitor name isn't found in the text at all. This doesn't catch everything, but it catches the most obvious hallucinations. The principle: trust the LLM for analysis, verify it for facts.

**Read more:** Anthropic docs on reducing hallucination — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

### P3 — Polish

- **Print statements as logging:** Uses `print()` for all status output. Works now but makes it hard to add quiet mode or filter output later. *(Functionality)*
- **No .gitignore visible:** When this goes to GitHub, .env, __pycache__, and output CSVs could all get committed. *(Security)*
- **No input validation on competitor list:** `--competitors ",,,"` produces a list of empty strings sent to Claude. Not a crash, but garbage in. *(Functionality)*
- **No sample transcript for testing:** A new user has to find or create their own transcript just to try the tool. One small example file makes the difference between "I'll try it later" and "oh cool, it works." *(Shareability)*

---

## Bottom Line

This is a genuinely well-built tool. The error handling is thoughtful, the CLI is clean, the overwrite protection shows care for the user's data, and the use case has clear recurring value. You're ahead of most first builds. The main area to strengthen is the LLM integration layer — the tracker trusts Claude's output format and content more than it should, and a common Claude behavior (code fences around JSON) would cause the most likely real-world failure.

**Start with:** The JSON code fence issue. It's the highest-impact fix because it affects every run where Claude wraps the response. A few lines of cleanup before `json.loads()` and this tool gets dramatically more reliable.

---

## Next Steps

1. Address the P1 findings above
2. Run Gut Checc again after changes
3. Compare your new score to this report

*Save this report so you can track your progress across runs.*
