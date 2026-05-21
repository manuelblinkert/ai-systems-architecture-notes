# LLM Orchestration vs Prompting

## The distinction that matters

Prompting is how you communicate with an LLM.  
Orchestration is how you build a system around it.

Most production AI systems fail not because the prompts are bad, but because teams put orchestration logic inside prompts — and then wonder why the system is fragile, expensive, and impossible to debug.

---

## What prompting handles

A prompt handles:

- What task the model should perform
- The format and constraints of the output
- Relevant context the model needs to reason over
- Tone, persona, or style (when it matters)

A prompt does not handle state, routing, error recovery, multi-step coordination, or application logic. The moment you start embedding those into a prompt, you've crossed into orchestration territory — and you're doing it in the wrong place.

---

## What orchestration handles

Orchestration is everything the application does around the LLM call:

- **Routing** — deciding which model, prompt, or workflow path to use
- **State management** — tracking what has happened across steps or turns
- **Multi-step coordination** — sequencing calls, passing outputs between steps
- **Retrieval** — fetching and formatting external context before the call
- **Error handling and retries** — what happens when a call fails or returns bad output
- **Output validation** — checking whether the response meets the contract
- **Cost and latency control** — deciding when to skip, cache, or fall back

These belong in application code, not in the prompt.

---

## The common mistake

Teams discover that their simple prompt isn't enough. The model needs more context, needs to follow multi-step logic, needs to handle edge cases. So they make the prompt longer.

The prompt becomes a page of instructions. It includes conditionals: "if the user asks X, do Y, otherwise do Z." It includes state: "remember that in the previous step we did...". It includes error handling: "if you can't answer, say...".

This creates several problems:

- **Prompts are not code** — there's no type safety, no unit tests, no refactoring. Logic buried in a prompt is logic you can't debug.
- **Long prompts degrade quality** — models lose track of complex instructions at the edges of a long context. The more you put in, the less reliably each part is followed.
- **Changes are opaque** — a prompt change that moves conditional logic around looks the same as a wording fix. You can't diff the intent.
- **Cost scales with length** — prompt tokens cost money. Putting application logic there is paying the model to parse your control flow every request.

---

## Better pattern: thin prompts, explicit orchestration

The goal is prompts that do one thing, and application code that handles the rest.

A prompt should be readable in thirty seconds and answer: what is the model doing, and what should it return? If it takes longer than that, orchestration logic has leaked into it.

What the application layer handles instead:

```
step 1: retrieve relevant documents          ← application layer
step 2: format documents into context        ← application layer
step 3: call LLM with focused prompt         ← prompt (thin)
step 4: validate output format               ← application layer
step 5: if invalid → retry or fallback       ← application layer
step 6: pass result to next step             ← application layer
```

Each step is visible, testable, and independent. The prompt in step 3 doesn't know about retrieval, retries, or what comes next. It just does its job.

---

## Framework choice and over-reliance

LangChain, LlamaIndex, and similar frameworks provide orchestration primitives. They're useful at the prototype stage for moving fast. In production they create a different problem: too much magic, too little control.

When a chain fails, the error is often deep inside framework internals. Adding a logging step, a custom retry, or a non-standard validation means fighting the abstraction. Teams that started with a framework often spend more time working around it than using it.

This isn't an argument against frameworks — it's an argument for understanding what they're doing. If you can't explain what happens between the user's message and the LLM call in your own system, you don't own your orchestration.

---

## Common mistakes

- **Logic in prompts** — conditionals, state references, multi-step instructions all belong in code
- **God prompts** — a single prompt that tries to handle every case; breaks on edge cases and degrades quality
- **No output validation** — trusting that the model returns the right format without checking
- **Invisible retries** — retries happening in the framework without visibility or limits
- **One-size-fits-all routing** — using the same model and prompt for fast cheap tasks and slow expensive ones
- **No step boundaries** — a single LLM call does retrieval, reasoning, and formatting all at once; impossible to diagnose which part failed

---

## Practical checklist

- [ ] Can you describe each orchestration step independently from the prompt?
- [ ] Are prompts short enough that a new engineer can read them in one pass?
- [ ] Is output validation explicit and before results reach users or downstream steps?
- [ ] Are retries bounded, logged, and not hidden inside a framework?
- [ ] Is routing logic (which model, which prompt) in code, not in a mega-prompt?
- [ ] Can you run a single orchestration step in isolation for testing or debugging?
- [ ] Do you know the token cost and latency of each step?
