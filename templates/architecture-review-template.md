# AI System Architecture Review

Use this before shipping a new AI system or before making significant changes to an existing one. Not every section applies to every system — skip sections that aren't relevant, but be honest about why.

---

## System overview

**What does this system do?**  
_One or two sentences. If you can't summarize it, the scope may be unclear._

**What triggers an LLM call?**  
_User action / background job / event / scheduled / other_

**What does a bad output look like, and who sees it?**  
_Be specific. "Wrong answer" is not enough._

---

## Orchestration

**What are the steps between a user action and the LLM response?**  
_List each step. If you can't list them, the orchestration isn't explicit enough._

| Step | What it does | Owned by |
|------|-------------|----------|
| 1    |             |          |
| 2    |             |          |
| 3    |             |          |

**Is any application logic embedded inside a prompt?**  
_Conditionals, state references, routing decisions, multi-step instructions — if yes, move them to code._

**Can each orchestration step be tested or run in isolation?**  
☐ Yes — describe how  
☐ No — explain why not

---

## Prompts

**How many prompts does this system have?**  

**Are prompts versioned separately from application code?**  
☐ Yes — where?  
☐ No

**Can a new engineer read each prompt and understand what it does in under a minute?**  
☐ Yes  
☐ No — likely too long or carrying logic that belongs in code

**What happens if the model doesn't follow the prompt instructions?**  
_Is there validation? A fallback? Or does bad output reach the user silently?_

---

## Observability

**Can you reproduce a specific bad answer from production logs?**  
☐ Yes  
☐ No — the system is not observable enough to ship

**What is logged per request?**  
☐ Request ID  
☐ Fully rendered prompt (not template)  
☐ Raw model response  
☐ Model, version, temperature  
☐ Token usage (input and output)  
☐ Latency per step  
☐ Prompt version or hash  
☐ Retrieval context (if RAG)

**Is cost visible at request granularity?**  
☐ Yes  
☐ No — you will discover cost problems from a billing surprise

---

## Retrieval (skip if no RAG)

**What is the retrieval strategy?**  
_Vector search / keyword / hybrid / other_

**Is retrieved context logged alongside the response?**  
☐ Yes  
☐ No — you can't diagnose wrong answers that come from bad context

**How do you know retrieval is returning relevant documents?**  
_Eval set / relevance scoring / manual spot checks / nothing yet_

**What happens when retrieval returns nothing or low-quality results?**  
_Does the model hallucinate? Refuse? Fall back gracefully?_

---

## Evaluation

**Is there an eval set for this system?**  
☐ Yes — number of cases: ___ , last run: ___  
☐ No

**Does the eval set include known edge cases and failure modes?**  
☐ Yes  
☐ No

**How do you detect quality regression after a prompt change?**  
_Eval pipeline / human review / user feedback / you don't_

**What is the bar for shipping a prompt change?**  
_Define it before you need it._

---

## Reliability and fallbacks

**What happens when the LLM API is unavailable?**  
_Hard error / graceful degradation / cached response / other_

**What happens when the model times out?**  

**What happens when the model returns an output that fails validation?**  
_Retry / fallback / surface error / pass bad output downstream_

**Are retries bounded and logged?**  
☐ Yes — max retries: ___ , logged: ___  
☐ No

**Is there a circuit breaker or fallback model if the primary provider is down?**  
☐ Yes  
☐ No

---

## Cost and latency

**What is the expected token cost per request (input + output)?**  
_Estimate at P50 and P95 input sizes._

| Scenario | Input tokens | Output tokens | Est. cost |
|----------|-------------|---------------|-----------|
| P50      |             |               |           |
| P95      |             |               |           |

**Is latency acceptable for the user-facing context?**  
_Synchronous UI response, background job, and streaming have different tolerances._

**Is streaming implemented where latency matters?**  
☐ Yes  
☐ No  
☐ Not applicable

**Are there expensive steps that could be cached or skipped for repeat inputs?**  

---

## Production readiness

Before marking a system production-ready, confirm each of the following:

- [ ] Bad answers can be reproduced from logs
- [ ] Prompts are versioned and their changes are reviewable
- [ ] Output validation is in place before results reach users
- [ ] Fallback behavior is defined and tested for every failure mode
- [ ] Cost per request is visible and acceptable at projected scale
- [ ] An eval baseline exists to detect regressions
- [ ] LLM provider can be swapped or fallen back to without a code rewrite
- [ ] Orchestration steps are explicit and independently testable

---

## Open issues

_List anything this review surfaced that isn't resolved. Don't ship with unknowns you haven't named._

| Issue | Severity | Owner | Resolution |
|-------|----------|-------|------------|
|       |          |       |            |
