# AI-Powered Inbound Scoring & Routing Framework (Make.com)

This repository contains a **Make.com scenario blueprint** that turns messy, unstructured inbound messages into **structured, scored decisions** using an LLM (OpenAI) — then logs the results to Google Sheets.

It was originally built for **job opportunity screening** from LinkedIn job alert emails, but the same architecture generalizes to:

- Lead qualification (sales / RevOps)
- Support ticket triage
- Vendor / supplier evaluation
- Candidate screening
- Intake form scoring and routing

The key idea: **LLM judgment + deterministic automation** (logging, retries, controlled failure paths).

---

## Overview

**Input:** New LinkedIn job alert emails (Gmail)  
**Process:** Extract job URLs → iterate → send job + rubric to OpenAI → receive JSON score  
**Output:** Append a structured row to Google Sheets for tracking and review

This is **not** a “toy” copy/paste automation — it’s an extensible pattern for making AI outputs reliable inside real workflows.

---

## Problem This Solves

When inbound data is unstructured (emails, web forms, chat messages), traditional automations struggle because they need clean fields.

This framework:

- Accepts real-world messy text
- Extracts targets (URLs) deterministically
- Uses AI to evaluate and produce **structured JSON**
- Logs results in a consistent schema for review, filtering, and follow-up

---

## High-Level Architecture

1. **Gmail Trigger** – Watch for new LinkedIn job alert emails  
2. **Regex Text Parser** – Extract LinkedIn job URLs from the email body  
3. **Iterator** – Process each extracted URL individually  
4. **Set Variables** – Inject candidate profile context + scoring rubric + the specific URL to evaluate  
5. **OpenAI (ChatGPT) Call** – Return **JSON-only** job evaluation  
6. **JSON Parse (Validation)** – Parse/validate JSON output *(ignore errors to prevent scenario failure)*  
7. **Google Sheets** – Append a new row with score + reasons + timestamp

---

## Scenario Flow (Make.com)

### 1) Gmail — Watch Emails
- Watches `INBOX`
- Filters by sender: `jobalerts-noreply@linkedin.com`
- Optional keyword filter (example): `automation remote`
- Pulls full message body

### 2) Text Parser (RegExp)
Extracts job links from the email body using a regex:

- Pattern: `https://www\.linkedin\.com/comm/jobs/view/\d+`
- Global match enabled (captures multiple jobs per email)

### 3) Iterator
Iterates through each extracted URL so every job gets a separate evaluation and output row.

### 4) Tools — Set Variables
Creates reusable variables, including:

- `profile_context` (candidate background + constraints)
- `job_input` (instructions: find *only* the job matching the current URL inside the email)
- `job_url` (the current extracted URL)

### 5) OpenAI — Generate a Response (JSON)
Sends the prompt + context and enforces structured output:

- Model: `gpt-4o-mini`
- Response format: `json_object`
- Max output tokens: `500`

**Output schema (required):**
```json
{
  "job_title": "string",
  "company": "string",
  "job_url": "string",
  "ideal_alignment_score": 1,
  "tier": "Tier-1 | Tier-2 | Below target",
  "primary_reasons": ["..."],
  "concerns": ["..."],
  "recommendation": "Apply | Consider | Skip"
}
```

### 6) JSON — Parse JSON (Validation Step)
Parses `{{5.result}}` (the LLM response).  
An **Ignore** error handler is attached here so malformed JSON does not crash the run.

> Note: In the current blueprint, Google Sheets maps fields directly from `OpenAI (module 5)` output.  
> The JSON Parse step is kept as a **validation guard / future-proofing hook** (easy to switch Sheets mapping to module 15 later).

### 7) Google Sheets — Add a Row
Appends a row with:

- job title
- company
- URL
- score (1–10)
- tier
- recommendation
- primary reasons (joined)
- concerns (joined)
- timestamp

---

## Error Handling & Reliability

### OpenAI Module: Break + Retry
A **Break** error handler is attached to the OpenAI module with:

- Retries: **3 attempts**
- Interval: **5 seconds**

This improves reliability when the API temporarily fails.

### JSON Parse Module: Ignore Errors
An **Ignore** handler is attached to JSON Parse so the scenario doesn’t fail if the LLM returns invalid JSON.

---

## Why This Is Not a “Basic Automation”

Most beginner Make scenarios:

- Move clean data from A → B
- Don’t loop through multiple items safely
- Don’t enforce structured AI output
- Have no resilience plan for API failures

This scenario includes:

- deterministic URL extraction
- iteration over multiple jobs per message
- LLM call with **strict JSON schema**
- explicit retry strategy on the AI step
- validation step + controlled failure behavior
- normalized logging to a persistent store (Sheets)

---

## Tech Stack

- **Make.com** (scenario orchestration)
- **Gmail module** (email trigger)
- **RegExp parser** (URL extraction)
- **OpenAI (ChatGPT)** (JSON-based evaluation)
- **JSON module** (validation / parsing)
- **Google Sheets** (logging & review)

---

## 🔧 Customization Required

**This system is a framework, not a plug-and-play solution.**

Users must customize:
- **Evaluation prompt** – Define your specific criteria
- **Scoring logic** – Set thresholds and decision rules
- **Output schema** – Structure JSON fields for your use case
- **Routing rules** – Determine what happens at each score/tier *(today it logs to Sheets; routing can be added later)*

**The architecture is reusable; the business logic is not.**

Example prompts and scoring rubrics are provided as templates only.

---

## ⚠️ What’s Not Included

This is a **technical framework**, not a turnkey solution.

**Not included:**
- Pre-configured prompts for specific industries
- Training data or model fine-tuning
- CRM integrations (Salesforce, HubSpot, etc.)
- Deduplication logic (can be added)
- Real-time webhook triggers (uses polling)

**What IS included:**
- Complete Make.com scenario structure
- OpenAI integration with structured output
- Error handling (retries + controlled ignore)
- Documented architecture and flow
- Example scoring logic (adaptable)

**For production deployment, clients typically need:**
- Domain-specific prompt engineering
- Integration into their stack (CRM/DB/ticketing)
- Testing + calibration period
- Documentation + handoff training

---

## 💼 Why This Matters for Businesses

**Traditional automation:**
- Moves data from A → B
- Cannot handle judgment calls
- Fails on unstructured input

**AI-only solutions:**
- Unpredictable outputs
- No audit trail
- Black-box decision-making

**This system:**
- Combines AI judgment with rule-based execution
- Handles messy real-world inputs
- Provides transparency and logging
- Scales across departments and use cases

**Result:** Decision fatigue reduced, response time improved, evaluation consistency increased — without surrendering control to AI.

---

## Status & Future Enhancements

Planned upgrades (typical next steps):
- Add **deduplication** (Data Store keyed by job URL)
- Add **routing logic** (e.g., Slack alert for Tier-1, queue Tier-2, skip below-target)
- Swap input source from email → **job boards / ATS APIs**
- Add **observability** (error log sheet, alerts, execution summaries)
- Store results in a real DB (Postgres) for analytics at scale

---

## Files

- `JOB SEARCH SCEN.blueprint.json` — Make.com scenario blueprint export
- `README.md` — this documentation

## 🚀 Quick Start

**To deploy this scenario:**

1. **Import blueprint** into Make.com
2. **Connect accounts:**
   - Gmail (inbox with target emails)
   - OpenAI (API key required)
   - Google Sheets (for logging)
3. **Customize variables in Module 4:**
   - Update `profile_context` with your criteria
   - Modify `scoring_rubric` for your use case
4. **Test with sample email**
5. **Adjust prompt based on results**

**Estimated setup time:** 30-60 minutes (excluding prompt tuning)

## 💰 Cost Estimate

**Typical monthly costs for 50 evaluations:**

- **Make.com:** Free tier (1,000 operations/month) or $9/month Core plan
- **OpenAI API:** ~$0.05-0.15/month (GPT-4o-mini at ~$0.001/evaluation)
- **Gmail/Sheets:** Free

**Total:** $0.05-9.15/month depending on volume and Make.com tier.

**Note:** Costs scale linearly with evaluation volume.

