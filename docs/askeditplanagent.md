Published Date: 2026-29-01 29th January 2026

Title: A deep dive into GitHub Copilot modes: Agent, Ask, Edit, Plan, and AIAgentExpert

Hero image: askeditplan.png


# A deep dive into GitHub Copilot modes: Agent, Ask, Edit, Plan, and AIAgentExpert

GitHub Copilot is no longer “just autocomplete.” Modern Copilot experiences expose multiple modes that deliberately change how Copilot behaves: how much context it considers, how it balances explanation vs action, and how autonomous it is allowed to be.

These labels can vary a bit depending on where you use Copilot (VS Code, Visual Studio, JetBrains, GitHub.com), but the intent is consistent.

## The autonomy spectrum

A useful mental model is a spectrum from “explain only” to “take the wheel”:

- **Ask**: explain and clarify, minimal action
- **Edit**: apply a specific change you describe
- **Plan**: break down work, identify risk, define steps and acceptance criteria
- **Agent**: execute toward an outcome across multiple steps and files
- **AIAgentExpert**: like Agent, but optimized for deeper reasoning and higher verification on complex or risky work

A quick decision heuristic:

- If you are uncertain what is going on: choose **Ask**
- If you know exactly what you want changed: choose **Edit**
- If you need sequencing, risk reduction, or team alignment: choose **Plan**
- If you want end to end implementation with repo discovery: choose **Agent**
- If the cost of being wrong is high: choose **AIAgentExpert**

---

## Ask mode: understand the system, reduce ambiguity

### What it optimizes for
Ask mode is optimized for comprehension and decision support. It is where you interrogate the code, the architecture, and the tradeoffs before changing anything.

### When to use it
Use Ask when:
- you are onboarding to an unfamiliar codebase
- you need an explanation of what code does and why it exists
- you want to compare alternatives before choosing an approach
- you are debugging conceptually (what could cause this symptom?)

### Example prompts
- “Explain what this function does and list edge cases.”
- “Why might this be causing a race condition?”
- “What’s the difference between these two caching strategies?”

### Sample: asking about a code snippet

```python
def slugify(name: str) -> str:
    return "".join(c.lower() if c.isalnum() else "-" for c in name).strip("-")
```

Good Ask prompt:
- “Explain what `slugify` does, list edge cases (unicode, multiple dashes), and suggest improvements without changing the signature.”

Interesting fact:
Ask mode is a quality multiplier because it helps you surface constraints. Most “bad AI edits” happen because constraints were never stated.

---

## Edit mode: controlled, scoped code changes

### What it optimizes for
Edit mode is optimized for predictability. You already know what you want and you want Copilot to apply it quickly.

### When to use it
Use Edit when:
- the change is localized (single function, file, or small set of files)
- the work is mechanical (rename, convert style, add validation)
- you want minimal architectural improvisation

### Example prompts
- “Convert this callback based function to `async/await`.”
- “Add input validation. Return 400 with a JSON error if invalid.”
- “Replace deprecated API usage with the current stable equivalent.”

### Sample: Edit request with clear constraints

Before:

```js
function parsePort(value) {
  const n = parseInt(value, 10);
  if (Number.isNaN(n)) return 3000;
  return n;
}
```

Great Edit prompt:
- “Edit `parsePort` to reject ports outside 1..65535, return default 3000, and add comments explaining why.”

After (example output you might want):

```js
function parsePort(value) {
  // Parse using base 10 to avoid surprises like "08" being treated as octal in older environments.
  const n = Number.parseInt(value, 10);

  // Default when value is missing or not a number.
  if (Number.isNaN(n)) return 3000;

  // Ports are valid only in the 1..65535 range.
  if (n < 1 || n > 65535) return 3000;

  return n;
}
```

Interesting fact:
Edit mode tends to be best for the “boring engineering” that makes systems reliable: validation, consistency, and small refactors that preserve behavior.

---

## Plan mode: de risk big changes before writing code

### What it optimizes for
Plan mode is optimized for sequencing, risk management, and clarity. Think mini design doc: goals, non goals, steps, and how you will validate.

### When to use it
Use Plan when:
- the task is large or ambiguous
- you need to coordinate work across components
- you want acceptance criteria before implementation
- you want rollout and rollback steps

### Example prompts
- “Plan a migration from REST to GraphQL with minimal client disruption.”
- “Plan how to add rate limiting with feature flags and observability.”
- “Break this refactor into incremental steps with tests.”

### Sample Plan structure you should expect
- **Goals** and **non goals**
- **Assumptions** and **unknowns**
- **Step by step changes** with dependencies
- **Test strategy**
- **Rollout plan** (feature flags, canary, progressive delivery)
- **Rollback plan**
- **Observability** (metrics, logs, tracing)

Interesting fact:
Planning reduces overall cycle time because it prevents expensive backtracking once code has already been changed.

---

## Agent mode: outcome driven multi step execution

### What it optimizes for
Agent mode is optimized for reaching an outcome, not just producing a snippet. It often involves repo exploration, multi file edits, updating tests, and iterating.

### When to use it
Use Agent when:
- you want end to end completion (“make X work”)
- the change spans multiple files
- you need discovery (“where is this implemented?”)
- you want Copilot to iterate toward green tests / passing CI

### Example prompts
- “Add OAuth login end to end: UI, backend, callback handling, tests.”
- “Refactor this module to remove duplication and update unit tests.”
- “Investigate why CI fails on Windows and propose a fix.”

### Sample: defining acceptance criteria for an Agent
A strong Agent prompt includes:

- “Done means: tests pass, lint passes, no breaking API changes, docs updated.”
- “Do not: rename public endpoints, change database schema, or upgrade major versions.”

Interesting fact:
Agent effectiveness correlates with feedback loops. Repos with good tests, linting, and type checking make agentic work safer and faster.

---

## AIAgentExpert: deeper reasoning and higher verification

### What it optimizes for
AIAgentExpert is typically the more thorough version of Agent mode. It biases toward correctness, edge cases, and validation.

### When to use it
Use AIAgentExpert for:
- security sensitive changes (auth, sessions, permissions)
- performance and scalability work
- intermittent bugs where root cause is unclear
- high impact refactors

### Example prompts
- “Audit authentication and session handling for common vulnerabilities.”
- “Find root cause of this intermittent deadlock and propose a minimal fix.”
- “Reduce p95 latency by 30 percent and justify tradeoffs.”

Interesting fact:
Hard debugging is hypothesis elimination. Expert style agents tend to propose structured investigation loops: reproduce, isolate, measure, change one variable, validate.

---

## Combining modes in real workflows

### Workflow: diagnosing and fixing a production bug
1. **Ask**: explain the stack trace and likely causes
2. **Plan**: propose an investigation plan and acceptance criteria
3. **AIAgentExpert**: implement a minimal fix and regression test
4. **Edit**: polish naming, messages, and consistency

### Workflow: adding a feature safely
1. **Plan**: define API, rollout, tests, and constraints
2. **Agent**: implement across backend, frontend, and tests
3. **Edit**: final cleanup and readability improvements
4. **Ask**: sanity check edge cases and failure modes

---

## Common failure modes and how to avoid them

- **Skipping constraints**: state “keep behavior unchanged” if that matters.
- **No validation loop**: always ask for tests or commands to run.
- **Over broad prompts**: “make it better” invites scope creep. Define “done.”
- **Silent assumptions**: spell out version constraints, compatibility needs, and performance targets.

---

