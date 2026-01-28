# Code Review Protocol for AI-Assisted Development

> Derived from failure modes identified in high-adoption AI coding workflows (Karpathy, Cherny, Osmani et al.)

---

## Core Principle

**The agent optimizes for coherent output, not for questioning your premises.**

Your role shifted from implementer to orchestrator. Review accordingly.

---

## Pre-Review Setup

### 1. Fresh Context Review

Before reviewing AI-generated code, use a **new context window**. The same model catches its own mistakes when given a clean slate.

```
# In Claude Code or similar:
# Open fresh session, paste the code, ask:
"Review this implementation for assumption errors, unnecessary complexity, and dead code. What would you change?"
```

### 2. Architectural Context Gathering

Before examining implementation details:

- [ ] Review relevant project specs / CLAUDE.md
- [ ] Identify invariants and unwritten rules for this area
- [ ] Note adjacent systems that might be affected
- [ ] Understand the *intent* behind the request, not just the literal ask

---

## The Four-Phase Review

### Phase 1: Assumption Audit

**Targets:** Assumption propagation, the "two-steps-back" pattern

| Question | Red Flag |
|----------|----------|
| What assumptions were made about inputs, state, or dependencies? | Unstated assumptions treated as facts |
| Were clarifying questions warranted but not asked? | Agent ran with ambiguity |
| Does implementation match *intent* or just *literal request*? | Letter of the law, not spirit |
| Is there early misunderstanding compounded across files? | Faulty premise baked into architecture |

**Action:** If assumptions are wrong, don't patch—reframe and regenerate.

---

### Phase 2: Complexity Audit

**Targets:** Abstraction bloat, over-engineering

Apply the **"Couldn't you just...?" test**:

```
For each class:      Could a function suffice?
For each 1000 lines: Could 100 lines do the job?
For each abstraction: Is this solving a real problem or an imagined one?
```

**Quantitative checks:**

- [ ] Lines of code vs. functional requirements ratio
- [ ] Number of new abstractions introduced
- [ ] Inheritance depth / composition complexity
- [ ] Count layers of indirection—each needs justification

**The agent's tell:** When you ask "couldn't this be simpler?" and it immediately agrees and simplifies, the complexity was unnecessary. It was optimizing for looking comprehensive, not for maintainability.

---

### Phase 3: Hygiene Audit

**Targets:** Dead code accumulation, collateral damage

| Check | What to Look For |
|-------|------------------|
| Old implementations | Previous versions left behind, not cleaned up |
| Comment damage | Comments removed as side effects of edits |
| Adjacent code alterations | Changes to code "nearby" that wasn't part of the task |
| Orphaned imports/dependencies | Added but unused, or removed but still referenced |

**Diff discipline:** The changeset should contain *only* what's necessary for the task. Nothing adjacent, nothing "while I'm here."

**Tooling:**
```bash
# Run dead code detection
# Run unused import checks
# Diff should be minimal and focused
```

---

### Phase 4: Comprehension Check

**Targets:** Comprehension debt, rubber-stamping

#### The Explanation Test

Can you explain **how** this works, not just **that** it works?

- If no → This is a **learning signal**, not a merge signal
- Have the AI explain its approach
- Trace the logic manually until you understand
- Don't merge what you can't explain

#### The Modification Test

Could you confidently modify this code tomorrow **without AI assistance**?

- If no → You're accumulating comprehension debt
- Spend the time now or pay interest later

#### The 2am Test

Could you debug this at 2am when something breaks?

- If no → You don't understand it well enough to own it

---

## Verification Strategy

### Declarative Success Criteria (Define Before Implementation)

```markdown
## Success Criteria for [Feature/Task]

### Tests (write first)
- [ ] Unit test: [specific case]
- [ ] Integration test: [specific flow]
- [ ] Edge case: [boundary condition]

### Constraints
- [ ] Performance: [latency/throughput requirement]
- [ ] Security: [auth/validation requirements]
- [ ] API contract: [interface specification]

### Acceptance
- [ ] Existing tests still pass
- [ ] No regression in [adjacent system]
- [ ] Code reviewable by team member unfamiliar with AI context
```

Let the agent iterate until criteria are met. Review solutions, not partial progress.

### Automated Guardrails

**Rule:** For every class of mistake caught more than once, create a guardrail.

```
Mistake spotted → Write lint rule / test case / update AI instructions
```

This compounds. Your guardrails get smarter over time.

---

## Time Allocation

| Activity | Allocation | Focus |
|----------|------------|-------|
| Problem definition, specs, success criteria | 70% | What, not how |
| Execution / prompting | 10% | Declarative, not imperative |
| Review / verification | 20% | Design and architecture, not syntax |

Syntax is the AI's job. Judgment is yours.

---

## Anti-Patterns

### The "Almost There" Trap

> "The agent got 90% right, I can fix this in 5 more minutes..."
> *—5 hours later*

**Fix:** Set time bounds. Three iterations without convergence → step back, reframe the problem.

### The Velocity Illusion

More PRs ≠ more progress.

**Data:** 98% more PRs merged, but 91% longer review times. The bottleneck moved, you didn't get faster.

**Fix:** Measure outcomes (features shipped, bugs avoided), not outputs (PRs merged, lines written).

### The Rubber Stamp

If you're merging code you can't explain, you're not reviewing—you're hoping.

**Fix:** Apply the explanation test. Every time.

### The Sycophancy Blindspot

The agent agrees enthusiastically with whatever you described, even if incomplete or contradictory. No "Are you sure?" or "Have you considered...?"

**Fix:** Explicitly ask for pushback:
```
"What's wrong with this approach? What alternatives did you consider? What could break?"
```

---

## Escalated Scrutiny Triggers

Apply heightened review when:

| Trigger | Why |
|---------|-----|
| Adjacent system changes | Agent altered something "nearby" without being asked |
| New architectural patterns | Abstractions, patterns, or dependencies introduced |
| Security-adjacent code | Auth, data handling, external interfaces |
| High-confidence + odd feeling | Looks plausible, tests pass, but something's off |
| Large diff size | More code = more surface area for hidden issues |

---

## Skill Maintenance

To prevent atrophy while using AI assistance:

1. **Alternate**: Write some features manually to maintain muscle memory
2. **TDD discipline**: Write test cases before AI implements
3. **Pair reviews**: Discuss AI suggestions with seniors to learn decision-making
4. **Demand explanations**: Have AI justify its approach, then verify the reasoning
5. **Periodic solo coding**: Regularly implement without AI to calibrate your abilities

---

## Pre-Merge Checklist

Before merging any AI-generated code:

```markdown
## Pre-Merge Verification

### Comprehension
- [ ] I can explain how this works without referring to the code
- [ ] I could modify this tomorrow without AI assistance
- [ ] I could debug this at 2am if it breaks

### Quality
- [ ] Fresh-context review completed (AI reviewed its own code in new session)
- [ ] No unnecessary complexity (passed "couldn't you just...?" test)
- [ ] No dead code or collateral damage
- [ ] Diff contains only what's necessary for the task

### Verification
- [ ] All success criteria met
- [ ] Tests pass and actually validate behavior
- [ ] No regression in adjacent systems

### Ownership
- [ ] I'm comfortable being accountable for this code
- [ ] A teammate unfamiliar with the AI context could review this
- [ ] This follows patterns I'd want juniors to copy
```

---

## Integration with CLAUDE.md

Add to your project's CLAUDE.md or equivalent:

```markdown
## Code Review Requirements

All AI-generated code must pass review protocol before merge:
1. Fresh-context self-review in new session
2. Four-phase review: Assumptions → Complexity → Hygiene → Comprehension
3. Pre-merge checklist completed

### Automated Checks Required
- Dead code detection
- Unused import check
- Lint rules pass
- Test coverage maintained

### Complexity Constraints
- Prefer functions over classes unless state management required
- Maximum [N] layers of abstraction without explicit justification
- New patterns require design discussion before implementation
```

---

## Summary Heuristic

Three questions before every merge:

1. **Could I write this from scratch?** → If no, learning opportunity
2. **Could I debug this at 2am without AI?** → If no, comprehension debt
3. **Would I want a junior to copy this pattern?** → If no, fix it now

The goal isn't to slow down. It's to ensure velocity converts to progress, not accumulated debt.
