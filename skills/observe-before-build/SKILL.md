---
name: observe-before-build
description: Observation-first constraint for all agentic work. Agents must verify ground truth before writing code. Fires on any error, failure, or debugging context. Integrates with systematic-debugging, TDD, parallel-agents, and code-review skills.
argument-hint: ""
---

# Observe Before Build

## The Constraint

**No agent writes code until ground truth is verified.**

This is not a debugging skill. It is a constraint on all skills. It modifies how systematic-debugging, test-driven-development, dispatching-parallel-agents, and requesting-code-review operate.

## When This Fires

- Any error, failure, or unexpected behavior
- Before dispatching parallel agents with shared assumptions
- Before writing tests that mock external boundaries
- Before proposing infrastructure changes
- Before any PR that changes configuration values

## The Protocol

### Phase 0: Observe (before any other phase of any other skill)

1. **Read the error output.** Every field. What does the system say?
2. **Read the configuration.** What does the system expect?
3. **Compare.** Do they match? State the comparison explicitly.
4. **Verify boundaries.** Can you reach the endpoint? Does the domain resolve? Does the token's issuer match the verifier's domain?
5. **State the root cause in one sentence.** If you cannot, you have not observed enough. Do NOT proceed.

### Integration with Existing Skills

#### systematic-debugging
Phase 0 (Observe) inserts BEFORE Phase 1 (Root Cause Investigation). You cannot investigate until you have observed. The observation phase produces a **fact sheet** — verified URLs, decoded tokens, config values compared against runtime values.

#### test-driven-development
Before writing any mocked unit test, write ONE test that validates the real boundary:
- Auth: does the configured issuer match the token's issuer?
- API: does the configured URL resolve?
- Database: does the connection string connect?
- Certificates: does the configured domain match the cert's CN/SAN?

Mocked tests are the second layer. Boundary tests are the foundation.

#### dispatching-parallel-agents
The observation agent runs FIRST. It produces a verified fact sheet. All other agents receive this fact sheet as context. No agent inherits unverified assumptions from the dispatching agent.

Pattern:
```
1. Dispatch observation agent -> produces fact sheet
2. Read fact sheet, verify it makes sense
3. Dispatch implementation agents WITH the fact sheet
```

Never dispatch implementation agents in parallel with the observation agent.

#### requesting-code-review
The reviewer checks: do hardcoded values (URLs, domains, team names, audience strings) match verified ground truth? If a PR changes configuration, was the new value verified against the real system?

## The Fact Sheet

The observation phase produces a structured fact sheet:

```
## Verified Ground Truth — [date]

| Claim | Expected | Actual | Match? |
|-------|----------|--------|--------|
| Worker URL resolves | yes | [curl result] | Y/N |
| Token issuer | [config value] | [decoded JWT iss] | Y/N |
| JWKS endpoint returns kid | [token kid] | [JWKS kids] | Y/N |
| DB connection | [conn string] | [ping result] | Y/N |
```

Any N in the Match column is the root cause. Fix it before writing any other code.

## Why This Exists

On 2026-03-23, a team of agents spent hours building a health dashboard, writing 75 tests, creating 20+ PRs, and redesigning infrastructure — while the actual problem was a one-line config mismatch visible in the first console paste. Every agent inherited an unverified assumption. Every mock hid the real failure. The observation-first constraint prevents this class of error.

## Anti-Patterns

| Pattern | Fix |
|---------|-----|
| "Let me add logging to understand this" | Read the existing output first |
| "Let me write a test for this" | Verify the boundary before mocking it |
| "Let me refactor for observability" | The system is already observable — read it |
| "Let me build a dashboard" | Fix the one-line config first |
| "I'll dispatch agents to investigate in parallel" | One observation agent first, then parallel |
| "The tests all pass so it must work" | Tests that mock the failure point prove nothing |

## Key Principle

**Prefer observation over inference.** When you can look at the actual state of the system, don't reason about what it should be.
