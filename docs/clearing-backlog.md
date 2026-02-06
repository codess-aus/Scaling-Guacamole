````markdown name=clearing-backlog-with-coding-agents.md
# Clearing a stubborn backlog with GitHub Copilot coding agent
## A deep technical playbook for Product Owners and DevOps teams

Backlogs get stuck for predictable technical reasons: work items are underspecified, hard to test, risky to review, and tangled across systems. GitHub Copilot coding agent can help you convert that backlog into merged pull requests, but only if you treat issue writing and instructions as engineering artifacts, not admin work.

This guide is hands-on: how to structure agent-ready issues, how to create instruction layers that compound over time, and how to design small atomic tasks that produce reviewable PRs. Examples use Python and JavaScript.

---

## 1) Mental model: agents are deterministic executors, not product thinkers

A human engineer can infer missing context, ask questions, and negotiate scope. A coding agent will try to complete what you asked, even if it has to guess. Your job is to remove guessing.

The fastest way to do that is to provide:

- **What** to change (file paths, modules, components)
- **Why** (user impact, incident link, performance goal)
- **How** (preferred patterns, examples, constraints)
- **How to validate** (tests, commands, acceptance criteria)

Interesting fact: most “AI output quality” problems in software teams are actually **requirements quality** problems. The agent just makes that more visible.

---

## 2) The “agent-ready issue” spec (copy and paste template)

Use this template for tech debt, DevOps toil, reliability improvements, and small features.

```markdown
## Goal
Describe the user or operational outcome (1 to 3 sentences).

## Context
- Why this matters now (incident link, customer impact, cost, time saved)
- Where in the codebase this lives (paths, services, packages)
- Any constraints (backwards compatibility, latency budget, security requirements)

## Current Behavior
Describe what happens today. Include logs, screenshots, or error messages if relevant.

## Desired Behavior
Describe what should happen after the change.

## Implementation Notes (optional but powerful)
- Preferred libraries or patterns to use
- Specific functions or modules to modify
- Example code style to follow (paste a small snippet)

## Acceptance Criteria
- [ ] Unit tests added or updated
- [ ] Existing test suite passes (include command)
- [ ] Lint or format passes (include command)
- [ ] Observability updated if relevant (logs, metrics, traces)
- [ ] Docs updated if relevant

## How to Test
Step-by-step reproduction and validation steps.
```

### Why this works

It encodes the same information you would give a new hire. The agent can explore the repo, but it cannot read your mind about constraints, validation, or team conventions.

---

## 3) Writing titles that scale when you have 10 PRs in flight

When you assign many issues, you will review many PRs. Use a title format that makes PR triage easy:

**Pattern:** `<area>: <change> (<impact>)`

Examples:

- `api: validate payload for POST /users (prevent 500s)`
- `ci: cache pip dependencies (cut build time)`
- `terraform: add s3 lifecycle retention policy (cost control)`

This is not cosmetic. It is operational hygiene for PR review throughput.

---

## 4) Provide “golden path” examples, not vague guidance

“Add robust error handling” is ambiguous. A small example removes ambiguity. Paste a snippet in the issue that shows the shape you want.

### Example A: Python error handling and structured logging

```python
from dataclasses import dataclass
import logging

logger = logging.getLogger(__name__)

@dataclass(frozen=True)
class Result:
    """
    Small Result object makes return values explicit and testable.

    Why this pattern helps:
    - Reduces exception-driven control flow for expected failures.
    - Encourages caller-side handling that is easy to unit test.
    - Gives the coding agent a clear, repeatable structure to follow.
    """
    ok: bool
    error: str | None = None

def parse_user_id(raw: str) -> Result:
    """
    Validate and normalize a user_id input without throwing.

    What:
    - Ensure raw is numeric.

    How:
    - Strip whitespace
    - Return Result(ok=False, error=...) on invalid input
    - Log a structured message with context fields

    Interesting fact:
    - Logging with key-value fields (via extra=...) improves searchability in log tools
      and reduces ambiguity compared to interpolated strings.
    """
    raw = (raw or "").strip()

    if not raw.isdigit():
        # Clear, structured logging. Do not log secrets.
        logger.info("Invalid user id", extra={"user_id": raw})
        return Result(ok=False, error="user_id must be numeric")

    return Result(ok=True)
```

How to use this in an issue:

- Paste the snippet
- Add: “Follow this `Result` pattern for validation failures, and log with `extra={...}` fields.”

### Example B: JavaScript input validation with clear errors

```javascript
export function validateEmail(email) {
  /**
   * Keep validation deterministic and testable.
   *
   * Why this pattern helps:
   * - Pure functions are easy to unit test.
   * - The agent can implement changes safely when behavior is encoded in tests.
   *
   * Interesting fact:
   * - Many production bugs come from inconsistent normalization (null, undefined, whitespace).
   *   Converting early (String(...) and trim()) standardizes the input surface.
   */
  const value = String(email ?? "").trim();

  if (!value.includes("@")) {
    return { ok: false, error: "email must contain @" };
  }

  return { ok: true, value };
}
```

---

## 5) Custom instructions: make quality automatic

Think of instructions as policy as code for your agent.

Instruction layers (in increasing scope):

1. **Repository instructions**: conventions that always apply in this repo
2. **Organization instructions**: standards across all repos
3. **Custom agents**: reusable instruction profiles for repeatable workflows

Interesting fact: well-maintained instructions reduce review time more than they reduce coding time, because fewer PRs come back with “wrong test approach” or “wrong logging style.”

### Example: repository instructions content you can adapt

Put something like this in your repo-level Copilot instructions (location varies by your setup, but the content shape is the key).

```markdown
# Repo Engineering Standards (for GitHub Copilot)

## Code style
- Prefer small, pure functions with explicit inputs and outputs.
- Avoid clever abstractions. Choose readability over micro-optimizations.
- Add comments only where intent is non-obvious.

## Python
- Target Python 3.12.
- Prefer standard library first. Add dependencies only with justification.
- Use pytest for tests. Name tests test_*.py.

## JavaScript or Node
- Target Node.js 22 LTS.
- Prefer built-in fetch and standard Web APIs where possible.
- Use minimal dependencies.

## Testing and validation
- Always add or update tests for behavior changes.
- Provide commands in the PR description:
  - python -m pytest
  - npm test (or the repo's standard)
- Ensure lint or format passes.

## Observability
- Use structured logs (key value fields).
- Do not log secrets or tokens.
```

---

## 6) Backlog decomposition: turning “big scary work” into safe slices

Agents handle small atomic tasks best. For large initiatives, create a tree of issues where each issue is:

- Independently mergeable
- Independently testable
- Low blast radius
- Clearly scoped to one area

### Example: “Improve authentication reliability”

Instead of one epic:

- Issue A: Add unit tests for token refresh edge cases
- Issue B: Add retry with jitter to auth upstream calls
- Issue C: Add metrics for refresh failures by reason
- Issue D: Document runbook for common auth incidents

Each becomes an agent task. Each yields a PR you can review without fear.

---

## 7) A practical workflow for Product Owners and DevOps leads

1. **Pick a backlog cluster**
   - Examples: CI slow, noisy alerts, dependency drift, flaky tests, Terraform hygiene

2. **Write 5 to 10 agent-ready issues**
   - Use the template in section 2
   - Keep scope narrow

3. **Assign them in parallel** to the coding agent
   - You are trading developer time for reviewer time
   - That is usually a good trade when the changes are atomic

4. **Review PRs with a checklist**
   - Does it meet acceptance criteria
   - Are tests meaningful, not just present
   - Does it align with repo instructions
   - Any cross-system impacts (humans own this)

5. **Update instructions** when you notice repeat review comments
   - Over time, your instruction set becomes the guardrails

---

## 8) PR review checklist (optimized for agent output)

Use this checklist to keep reviews fast and consistent.

```markdown
- [ ] Scope matches the issue (no surprise refactors)
- [ ] Tests cover success and failure cases
- [ ] Error messages are actionable
- [ ] Logs are structured and do not include secrets
- [ ] Performance considerations acknowledged (if relevant)
- [ ] Docs or runbooks updated (if relevant)
- [ ] Commands to validate are included in PR description
```

---

## 9) Where humans must stay in the loop

Even with perfect issues, keep humans responsible for:

- “Are we solving the right problem”
- Cross-service side effects
- Security posture and threat modeling
- Rollout strategy (feature flags, migrations, SLO risk)

Agents are great at implementing, but they do not own the system.

---

## Closing: backlog reduction as an engineering system

Treat backlog clearing like a production pipeline:

- Inputs: issue quality and instruction quality
- Automation: coding agents executing well-scoped tasks
- Quality control: tests, review checklist, and human judgment

If you improve the inputs, the whole system accelerates.

If you want to tailor this, pick one backlog theme (CI performance, Terraform cleanup, flaky tests, dependency upgrades, logging standardization) and generate a set of 8 to 12 agent-ready issues with acceptance criteria and validation commands.
````