# AI Prototypes vs Production Systems

## The core problem

A prototype proves that an AI workflow can work once.  
A production system proves that it can work repeatedly, reliably, observably, and economically.

These are different problems. Most teams treat them as the same problem.

---

## Why prototypes are misleading

A prototype works under controlled conditions: you choose the input, you retry until it looks right, you skip edge cases, latency is tolerable, and costs are irrelevant at zero users.

The demo goes well. The team ships.

Then real users arrive with inputs you didn't test. The model returns plausible-sounding garbage. Nobody notices because there's no tracing. The prompt that worked in the demo breaks on a slightly different phrasing. You find out when users complain, not when it happens.

The prototype wasn't wrong — it proved the idea can work. But it said nothing about whether the system can work reliably under real conditions. Those are different claims.

---

## What actually changes in production

**Reliability**  
A 95% success rate sounds acceptable. At 100k requests/month, that's 5,000 failures. Most LLM failures aren't hard errors — they're bad outputs that reach users silently.

**Latency**  
LLM calls are slow. Prototype users tolerate 5-second waits. Production users don't. Streaming, timeouts, fallbacks, and async patterns are architectural decisions that need to be designed in early.

**Cost**  
$0.002 per call sounds negligible. At scale with long contexts and high volume, it isn't. Cost visibility — per request, per user, per workflow — needs to be built in. Teams that ignore this discover it after a billing surprise.

**Observability**  
When a user gets a bad answer, can you reproduce it? Do you know which prompt version, model, and context was used? Most prototypes can't answer this. Production systems must.

**Evals**  
How do you know if a prompt change improved output quality or degraded it? Without an evaluation pipeline, you're guessing. Teams that skip evals lose quality silently over time.

**Fallback behavior**  
What happens when the model times out? Returns a refusal? The API is down? Prototypes crash or hang. Production systems need explicit fallback paths for every failure mode.

---

## Common failure modes

- **Fragile prompts** — designed for one happy-path input, break on real user variation
- **No tracing** — bad outputs can't be reproduced or diagnosed after the fact
- **No evals** — quality regressions after prompt changes go undetected
- **Cost blindness** — usage patterns at scale weren't modeled during design
- **Missing fallbacks** — no graceful degradation when the model or API fails
- **Silent retrieval failures** — RAG returns bad context, the model answers confidently from it

---

## The architecture gap

A prototype is usually:

```
input → prompt → LLM call → output
```

A production system needs explicit layers:

```
API layer            — auth, rate limiting, request IDs
Application layer    — business logic, workflow steps, routing
Context layer        — retrieval, user data, memory, formatting
LLM provider layer   — abstracted, swappable, with retries and timeouts
Observability layer  — traces, costs, latency, token usage
Evaluation layer     — quality signals, regression detection
```

Each layer can fail independently. Each layer needs to be observable. The LLM call is one component of the system, not the whole system.

---

## Production-readiness checklist

- [ ] Can you reproduce a bad answer from production logs?
- [ ] Are prompts versioned and deployable independently from application code?
- [ ] Is cost per request and per user visible?
- [ ] Are outputs validated before reaching users?
- [ ] Are fallback paths defined and tested for every failure mode?
- [ ] Is there an eval baseline to detect quality regressions?
- [ ] Are LLM providers abstracted enough to swap or fall back?
