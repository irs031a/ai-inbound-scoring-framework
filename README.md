# AI-Powered Inbound Scoring & Routing Framework (Make.com)

This repository contains a **Make.com scenario blueprint** that converts messy, unstructured inbound messages into **structured, scored decisions** using an LLM (OpenAI), with results logged to Google Sheets.

It was originally built for **job opportunity screening** from LinkedIn job alert emails, but the same architecture generalizes to:

- Lead qualification (sales / RevOps)
- Support ticket triage
- Vendor or supplier evaluation
- Candidate screening
- Intake form scoring and routing

The core idea: **LLM judgment combined with deterministic automation** (logging, retries, validation, and controlled failure paths).

---

## Overview

**Input:** New LinkedIn job alert emails (Gmail)  
**Process:** Extract job title → score alignment using OpenAI → receive structured JSON output  
**Output:** Append a structured row to Google Sheets for tracking and review  

This is **not** a copy-paste “AI toy.” It demonstrates how to make LLM outputs **predictable, auditable, and safe** inside real automation workflows.

---

## Problem This Solves

Inbound data (emails, forms, messages) is typically unstructured and inconsistent, making it difficult to automate decision-making reliably.

This framework:

- Accepts messy real-world text
- Extracts deterministic inputs (job title)
- Uses AI **only for judgment**, not control flow
- Forces **structured JSON output**
- Logs results in a stable schema for filtering and review

---

## High-Level Architecture

1. **Gmail Trigger** – Watch for new LinkedIn job alert emails  

2. **Regex Text Parser** – Extract job title from the email body  

3. **Set Variables (Context & Scoring Logic)**  
   - Candidate profile and constraints  
   - Tier definitions and scoring rubric  
   - Alignment rules and dealbreakers  

4. **Set Variables (Job Input Preparation)**  
   - Normalizes the extracted job title  
   - Assembles the authoritative input payload for LLM evaluation  

5. **OpenAI (ChatGPT) Call**  
   - Evaluates job title alignment  
   - Returns **JSON-only** structured output  

6. **JSON Parse (Validation)**  
   - Parses structured output  
   - Ignore-error handler prevents scenario failure on malformed responses  

7. **Data Store (Deduplication Check)**  
   - Uses a persistent key (job title) to detect previously evaluated roles  
   - Skips processing if the job title already exists  
   - Prevents duplicate scoring and duplicate log entries  

8. **Google Sheets (Logging)**  
   - Appends a structured row with score, tier, rationale, and timestamp  

---

## Error Handling & Reliability

### OpenAI Module: Break + Retry

A **Break** error handler is attached to the OpenAI module with:

- **3 retries**
- **5-second interval**

This protects against transient API failures.

### JSON Parse Module: Ignore Errors

An **Ignore** handler ensures the scenario continues even if the LLM returns invalid JSON.

---

## Why This Is Not a “Basic Automation”

Typical beginner scenarios:

- Move clean data from A → B
- Assume perfect inputs
- Fail hard on API errors
- Treat AI output as trustworthy by default

This scenario includes:

- Deterministic input normalization
- Explicit AI trust boundaries
- JSON-only AI output enforcement
- Retry logic on external APIs
- Controlled failure paths
- Persistent logging for auditability

---

## Tech Stack

- **Make.com** — scenario orchestration  
- **Gmail module** — inbound trigger  
- **RegExp parser** — job title extraction  
- **OpenAI (ChatGPT)** — structured evaluation  
- **JSON module** — parsing and validation  
- **Data Store** — deduplication  
- **Google Sheets** — logging and review  

---

## 🔧 Customization Required

This repository provides a **framework**, not a turnkey product.

You must customize:

- Evaluation prompt and criteria  
- Scoring thresholds and tiers  
- Output schema fields  
- Routing logic (currently logs only)  

**The architecture is reusable; the business logic is not.**

---

## ⚠️ What’s Not Included

**Not included:**
- Industry-specific prompts  
- CRM integrations  
- Real-time webhooks (polling-based)  
- Model fine-tuning  

**Included:**
- Full Make.com scenario blueprint  
- Structured OpenAI integration  
- Error handling and retries  
- Deduplication logic  
- Example scoring rubric  

---

## Status & Future Enhancements

Possible next steps:

- Route Tier-1 matches to Slack  
- Add queueing for Tier-2  
- Swap email input for ATS or job board APIs  
- Add execution observability (error logs, alerts)  
- Persist results to a database (Postgres)  

---

## Files

- `JOB SEARCH SCEN.blueprint.json` — Make.com scenario export  
- `README.md` — documentation  

---

## 🚀 Quick Start

1. Import blueprint into Make.com  
2. Connect Gmail, OpenAI, and Google Sheets  
3. Customize scoring logic in **Set Variables (Context & Scoring Logic)**  
4. Configure Gmail filter  
5. Test with a sample email  
6. Review results in Google Sheets  

**Estimated setup time:** 30–60 minutes (excluding prompt tuning)

---

## 💰 Cost Estimate

~50 evaluations/month:

- **Make.com:** Free tier or ~$9/month  
- **OpenAI API:** ~$0.05–0.15/month  
- **Gmail / Sheets:** Free

