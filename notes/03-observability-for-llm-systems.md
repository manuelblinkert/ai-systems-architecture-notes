# Observability for LLM Systems

## Why standard observability isn't enough

Traditional observability asks: did the request succeed, how long did it take, and what was the error?

LLM systems add a harder question: was the output any good?

A request that returns HTTP 200 in 800ms can still be a complete failure — the model hallucinated, misunderstood the intent, ignored instructions, or returned a response that looks plausible but is wrong. Standard metrics won't catch any of this. The system looks healthy. Users are getting bad answers.

Observability for LLM systems has to cover both the technical layer and the semantic layer.

---

## What you need to observe

**Technical signals** — these are table stakes, same as any service:

- Latency (total, time-to-first-token, streaming throughput)
- Error rate (API errors, timeouts, rate limits)
- Token usage (input, output, per-request, per-user, per-workflow)
- Cost (derived from tokens; needs to be visible at request granularity)
- Retry rate and retry depth

**Semantic signals** — these are specific to LLM systems:

- Output quality (did the response do what was asked?)
- Instruction-following rate (did the model follow format, constraints, persona?)
- Retrieval relevance (did RAG surface the right documents?)
- Refusal rate (how often does the model decline to answer?)
- User corrections or negative feedback signals

Most teams instrument the technical layer reasonably well. Almost no one instruments the semantic layer until a problem is already visible in user behavior.

---

## Tracing: the minimum viable requirement

If you can't reproduce a specific bad answer, you can't fix the system that produced it.

A trace for an LLM request should capture:

- Request ID and timestamp
- The exact prompt sent (not a template — the fully rendered string with all context filled in)
- Model, temperature, and any sampling parameters
- The raw response from the model
- Token usage and latency
- The prompt version or hash, so you can correlate quality with prompt changes
- Retrieval context (if RAG is involved): what documents were fetched, what was their rank
- Any downstream validation outcomes

Without this, every bad answer is a mystery. With it, a production bug becomes a replayable event.

---

## The semantic failure problem

Technical observability catches the system failing. Semantic observability catches the system being wrong.

Semantic failures look like:
- Confident answers to questions the model shouldn't answer
- Correct-sounding responses that contradict the source documents
- Format breakdowns that only appear with specific input patterns
- Gradual quality drift after a prompt change

These failures don't trigger alerts. They accumulate. Teams discover them weeks later when a user screenshots something, or a customer success rep escalates a pattern. By then, the root cause (which prompt version? which model change? which retrieval edge case?) is no longer traceable without good logs.

The fix isn't just better monitoring — it's treating output quality as a first-class signal that you collect data on continuously, not only when something is obviously broken.

---

## Evals: the observability for quality

An evaluation pipeline is how you detect semantic failures systematically.

Evals answer: did this change make things better or worse?

At minimum, a useful eval setup includes:

- A fixed set of representative inputs that cover your important cases and known edge cases
- Expected outputs or quality criteria for each input
- A way to run the set against a prompt change before shipping it
- Baseline scores so you can detect regression, not just spot-check

You don't need a perfect automated scoring system to start. Even human review of a small fixed eval set — run before every prompt change — is dramatically better than shipping changes and hoping.

LLM-as-judge patterns (using a model to score another model's output) are increasingly practical for automating this at scale, but they require calibration against human judgments to be trustworthy.

---

## Common mistakes

- **Logging templates, not rendered prompts** — "You are a {role}, answer in {format}" tells you nothing about what the model actually saw
- **No request IDs** — when a user reports a bad answer, there's no way to find the corresponding trace
- **Token cost invisible until billing** — you discover expensive patterns from a surprise invoice, not from request-level data
- **Evals only when something breaks** — quality drift between a working system and a broken one goes undetected
- **Treating refusals as neutral** — a high refusal rate on legitimate queries is a quality failure, not a safety win
- **No retrieval logging in RAG** — if the context is wrong, the answer will be wrong, but you can't see why without logging what was retrieved

---

## Practical checklist

- [ ] Can you reproduce any production answer from logs alone?
- [ ] Are fully rendered prompts (not templates) logged per request?
- [ ] Is token cost tracked at request and user granularity?
- [ ] Is latency broken down by step, not just total request time?
- [ ] Is the prompt version or hash captured in every trace?
- [ ] Is retrieval context logged in RAG systems (documents, scores, source)?
- [ ] Is there an eval set that runs before every prompt change?
- [ ] Are output quality signals collected, even if informally (thumbs down, escalations, corrections)?
