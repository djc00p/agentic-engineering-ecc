# 🤖 Agentic Engineering

[![ClawHub Skill](https://img.shields.io/badge/ClawHub-Skill-blue)](https://clawhub.ai/djc00p/agentic-engineering-ecc) [![Agent Skill](https://img.shields.io/badge/Agent-Skill-blue)](#) [![Eval-First](https://img.shields.io/badge/Eval--First-brightgreen.svg)](https://en.wikipedia.org/wiki/Test-driven_development) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Agentic Engineering** is a workflow pattern for AI-assisted engineering built on three disciplines: **eval-first execution**, **task decomposition**, and **cost-aware model routing**.

The premise is simple — an AI agent can generate code faster than any human, but speed without measurement produces code nobody can trust. This skill puts a measurable loop around the work: define what "done" means before writing anything, establish a baseline, break the work into verifiable units, route each unit to the cheapest model that can actually do it, then measure the delta.

---

## 🧪 The Eval-First Loop

The core idea: **you cannot improve what you never measured.** Evals are written *before* implementation, not after.

### Phase 1 — Define Evals

Two kinds, both written up front:

* **Capability eval** — "Can the system do the required task?"
* **Regression eval** — "Does existing functionality still work?"

```python
def test_webhook_handler_processes_valid_event():
    handler = WebhookHandler()
    event = create_test_event(type="user.created", data={"id": 123})
    result = handler.process(event)
    assert result.success == True
    assert result.event_id == event.id
```

### Phase 2 — Establish a Baseline

Run the evals *before* touching the implementation. Record which pass, which fail, and the exact failure signatures. That recorded state becomes the benchmark — without it, "it still works" is a guess.

### Phase 3 — Implement

Code, test, debug — with the evals still running. A failing eval is a blocking issue, not a note for later.

### Phase 4 — Measure the Delta

Re-run the evals. Compare against the baseline. Confirm no regressions landed, and document any *new* failure patterns the implementation surfaced. New failure patterns get added to the regression set.

> **Anti-pattern:** *"We'll test in staging."* Evals catch issues in seconds. Staging catches them after deployment.

---

## ⏱️ The 15-Minute Unit Rule

Each task unit must be **independently verifiable**, have a **single dominant risk**, and expose a **clear done condition**. Target roughly 15 minutes of focused work per unit.

### ✅ Good decomposition

**Unit 1 — "Parse CSV upload and validate rows"**

* *Risk:* CSV format malformed → validation logic catches and reports
* *Done:* Validation returns valid rows + error list
* *Tests:* `test_parse_valid_csv()`, `test_parse_malformed_csv()`
* *Time:* ~10 minutes

**Unit 2 — "Write valid rows to database"**

* *Risk:* Duplicate key constraint → should log and skip, not crash
* *Done:* All valid rows inserted, duplicates logged, error count matches
* *Tests:* `test_insert_valid_rows()`, `test_handle_duplicate_keys()`
* *Time:* ~12 minutes

### ❌ Bad decomposition

**"Implement user import feature"** — parsing, validation, database writes, error handling, rollback logic, and notifications all in one unit. Multiple dominant risks, no clear done condition, and a time estimate that could be 30 minutes or 3 hours.

### The decomposition checklist

For every proposed unit, ask:

1. **Independent?** Can it be tested without waiting on adjacent units?
2. **Single dominant risk?** Can you describe the most likely failure in one sentence?
3. **Clear done?** Can you write a test that proves completion?
4. **15-ish minutes?** Is it properly scoped, or a placeholder?

Any "no" means split it further.

---

## 🎯 Model Routing

Complexity picks the model. This is where cost discipline lives.

| Tier | Context | Time | Use For |
|------|---------|------|---------|
| **Haiku** | <200 tokens | <15 min | Boilerplate, narrow edits, classification, parsing a single CSV row |
| **Sonnet** | 200–1000 tokens | 15–45 min | Feature implementation, small refactors, test writing, building a parser + validator |
| **Opus** | >1000 tokens | 45+ min | Multi-file changes, root-cause analysis, architecture decisions |

Smaller units unlock faster, cheaper execution at lower tiers. **Escalate only when the lower tier fails for a clear reasoning gap — not on uncertainty.**

### Cost tracking

Per task, track: model tier, token estimate, retries, wall-clock time, and success/failure. That record is what makes the routing decisions tunable instead of vibes.

---

## 🔍 Review Focus

When reviewing AI-generated code, spend cycles on the things that actually break:

### Prioritize

* **Invariants and edge cases** — What assumptions does this make about input? What breaks when violated?
* **Error boundaries** — Where can external calls fail? Is it handled? Do error paths log enough to debug later?
* **Security and auth assumptions** — Is input validated? Are permissions checked at the right layer?
* **Hidden coupling** — Does this break other modules? Are there undocumented ordering dependencies? Does it affect rollout safety?

### De-prioritize

* **Style disagreements** — Formatters, linters, and type checkers handle this. Don't burn review cycles on variable names.
* **Micro-optimization** — Unless it's a measurably slow hot path.
* **Taste debates** — "I prefer this pattern" vs "this pattern is demonstrably better" are different conversations.

### Red flags

* ❌ No regression tests for affected domains
* ❌ Error handling is `try/except: pass`
* ❌ New execution-order dependencies that aren't documented
* ❌ Security-sensitive code with no explanation of *why*
* ❌ Feature flag logic that isn't tested

---

## 🔁 Session Strategy

* **Continue** for coupled units — they share context and context is expensive to rebuild.
* **Reset** after major phase transitions.
* **Compact** at milestones rather than letting the transcript bleed across unrelated work.

---

## 🚀 Quick Start

```text
1. Define acceptance criteria (user-facing behavior)
2. Write capability eval (can the system do the required task?)
3. Write regression eval (does existing functionality still work?)
4. Execute feature implementation with model routing
5. Re-run evals, compare deltas
6. Document any new risks discovered during review
```

---

## 📚 References

* `references/eval-patterns.md` — the eval-first loop in detail, with a worked webhook-handler example
* `references/decomposition-rules.md` — the 15-minute unit principle and worked breakdown examples
* `references/review-checklist.md` — what to focus on in code review (invariants, boundaries, security, coupling)

---

**Adapted from [everything-claude-code](https://github.com/affanm/everything-claude-code) by [@affaan-m](https://github.com/affaan-m) (MIT).**

**Published by [@djc00p](https://github.com/djc00p)**
