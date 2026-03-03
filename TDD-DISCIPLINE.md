# TDD Discipline in SpecForge

**Purpose**: Enforce Test-Driven Development (TDD) so LLMs and engineers cannot cheat by modifying tests instead of fixing implementation.

**Scope**: This governance applies to all SpecForge projects during Task execution (artefact 7 of 7).

---

## Why TDD Matters in SpecForge

SpecForge's power comes from **tests as immutable specifications**. If tests can be modified during implementation, the chain breaks:

- Requirements → Feature → Scenario → Test (spec) → Implementation
- If Test can change: Implementation hides non-compliance
- If Test is immutable: Implementation must conform or fail loudly

**Without TDD enforcement**: An LLM can write a test, realize it's hard to pass, then modify the test to make implementation easier. Result: specification drifts, requirements are betrayed, audit trail is broken.

**With TDD enforcement**: Test is written first, committed as [RED] (failing), then implementation must make it pass without touching the test. If implementation can't pass → implementation is wrong.

---

## The TDD Cycle in SpecForge Tasks

Each planning step generates exactly ONE test assertion, which gets split into two tasks:

```
┌─────────────────────────────────────────────────────────┐
│ TASK-NNN [RED]                                          │
│ Phase: Write ONE failing test                           │
│ Commit: test(feature): [RED] <behavior>                │
│ Result: Test fails ✗ (proves test is validating)       │
│ Test now becomes static contract               │
└─────────────────────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────────────────────┐
│ TASK-NNN+1 [IMPL]                                       │
│ Phase: Write implementation to pass test                │
│ Commit: impl(feature): <behavior>                       │
│ Result: Test passes ✓ (implementation satisfies spec)   │
│ NO test modifications allowed                           │
└─────────────────────────────────────────────────────────┘
```

**Key Points**:
- [RED] task MUST be completed before [IMPL] task begins
- Test is committed and immutable after [RED] task
- Implementation CANNOT modify test
- If test doesn't pass after [IMPL] → implementation fails (not test)

---

## Task Structure

### TASK-NNN Format in Plan Index

```
- TASK-001: Order.Total validates sum of items — [RED] — NOT STARTED
- TASK-002: Order.Total validates sum of items — [IMPL] — NOT STARTED
- TASK-003: Order rejects negative total — [RED] — NOT STARTED
- TASK-004: Order rejects negative total — [IMPL] — NOT STARTED
```

**Rules**:
- Odd-numbered tasks are ALWAYS [RED] (tests)
- Even-numbered tasks are ALWAYS [IMPL] (implementations)
- Both TASK-001 and TASK-002 describe the same behavior
- Task titles must be identical for paired red-impl tasks

### Phase Field in Task Template

Each task file (`TASK-NNN.md`) contains:

```markdown
## TDD Cycle Phase (CRITICAL)
**Phase**: `[RED]` or `[IMPL]`
```

- LLM reads this field
- Determines whether to write test or implementation
- Field is READ-ONLY (set by plan, not changed by task executor)

---

## Enforcement Mechanisms

### Prevention: Template Structure

1. **Task Index forces alternation**
   - Plan template requires `[RED] — [IMPL] — [RED] — [IMPL]` sequence
   - Even one `[RED][RED]` pair is a template violation
   - LLM executing plan MUST fail validation if sequence is wrong

2. **Phase field is explicit**
   - Task template clearly states Phase
   - No ambiguity about whether task is test or code

3. **Test immutability section in task template**
   - Explicitly states: "Tests are immutable contracts"
   - Explains violation consequences

### Detection: Verification Checklist

Each task has pre-completion verification:

```markdown
### If Phase = [RED]:
- ✅ Test file created and contains failing test
- ✅ Test is failing: run test to confirm fails
- ✅ Commit message includes [RED]
- ✅ No test mutations from previous cycles

### If Phase = [IMPL]:
- ✅ Paired [RED] test exists
- ✅ Test passes after implementation
- ✅ Test file is IDENTICAL to [RED] commit (zero changes)
- ✅ Commit message does NOT include [RED]
```

LLM executing task cannot set `Status: COMPLETE` without passing checklist.

### Recovery: Governance

If test mutual discovered AFTER completion:

1. **Identify violation**
   - Git log shows test modification after [RED] commit
   - Example: test(…)[RED] commit followed by later test(…) modifications

2. **Escalate**
   - Flag as policy violation
   - Record in `governance/violations/test-mutation-incident.md`
   - Document date, LLM/engineer responsible, root cause

3. **Remediate**
   - Revert test to [RED] version
   - Re-examine implementation (may not actually pass original test)
   - Decide: fix implementation OR modify spec with formal change request

4. **Prevent recurrence**
   - Review reason for mutation (was original test flawed? ambiguous?)
   - Update requirements/test template if pattern found
   - Add test to ensure this type of mutation is caught earlier

---

## Git History Pattern

A correct TDD sequence shows:

```bash
$ git log --oneline healthcheck-feature | head -20

c2e4a6f impl(health): Add memory usage to metrics response
b1d9a5e test(health): [RED] Metrics endpoint includes memory usage
a0c8b7e impl(health): Add uptime calculation to health response
9f7e6d5 test(health): [RED] Health endpoint returns uptime
8e6d5c4 impl(health): Add basic health endpoint returning 200
7d5c4b3 test(health): [RED] Health endpoint returns HTTP 200
```

**Reading backwards (oldest → newest)**:
1. `test(health): [RED]` — Test written, fails ✗
2. `impl(health):` — Implementation added, test passes ✓
3. `test(health): [RED]` — Next test written, fails ✗
4. `impl(health):` — Next implementation, test passes ✓
5. ... pattern repeats

**Key pattern**: Alternating `test(…)[RED]` and `impl(…)` commits.

If you see:
- ❌ `test(health): [RED]` followed by `test(health): [REFACTOR]` or `test(health): [IMPL]` → test mutation violation
- ❌ Two `impl(health):` commits in a row → implementation didn't pass first test, false restart
- ❌ `impl(…)` before any `test(…)[RED]` → implementation before test (not TDD)

---

## Preventing LLM Cheating

### Common Cheat Patterns (Now Prevented)

**Pattern 1: Modify test to match broken implementation**
```
❌ WRONG (would happen without TDD):
1. Write test
2. Write implementation  
3. Test fails
4. Modify test assertion to match implementation
Result: Test "passes" but spec is betrayed
```

**Prevention**: Test committed after [RED], immutable before [IMPL] begins. LLM cannot modify test.

**Pattern 2: Write all tests at once, implement loosely**
```
❌ WRONG (would happen without TDD):
1. Write 10 tests at once
2. Write implementation for "most" of them
3. Claim ambiguity, skip hard tests
Result: Specification drift, coverage gaps
```

**Prevention**: Test-by-test cycle. Each [RED] test is small, focused, immutable before [IMPL] starts. Progress is visible and auditable.

**Pattern 3: Infer missing behavior instead of testing**
```
❌ WRONG (would happen without TDD):
1. Test doesn't explicitly validate edge case
2. Claim specification allows lenient implementation
3. Skip edge case handling
Result: Requirements not honored
```

**Prevention**: Every assertion in test drives exactly one [IMPL] task. No "extra" implementation without test. Specification is observable, not inferred.

---

## Handling Implementation Failures

**Scenario**: TASK-NNN [IMPL] cannot make paired [RED] test pass.

**Root Causes** (in order of likelihood):
1. ✅ Implementation is wrong → fix implementation (correct use of TDD)
2. ❓ Design prevents passing test → escalate to architecture review
3. ❌ Test is flawed → only if approved spec change exists

**Process**:

### Case 1: Implementation is Wrong (99% of cases)
```
TASK-001 [RED]: "Order.Total returns sum of OrderLine amounts" — COMPLETE
  → Test file: tests/order/TEST-001.md
  → Assertion: order.Total == sum(lines[*].Amount)
  
TASK-002 [IMPL]: "Order.Total returns sum of OrderLine amounts" — IN PROGRESS
  → You write code in Order.cs
  → You run test: FAIL ✗
  → Debug and fix implementation
  → You run test: PASS ✓
  → Commit: impl(order): Order.Total returns sum
  → Status: COMPLETE
```

This is correct TDD. Test never changes.

### Case 2: Design Problem
```
TASK-003 [RED]: "Order rejects negative amounts with error" — COMPLETE
  → Test expects: Order constructor throws ArgumentException
  
TASK-004 [IMPL]: "Order rejects negative amounts" — IN PROGRESS
  → Current design: OrderLine.Amount validated at entry point
  → Order constructor cannot reject (design limitation)
  → Decision needed: move validation OR change design OR change test
```

**Decision options**:
- **A**: Move validation to Order constructor (change design, keep test as-is)
- **B**: Accept validation happens upstream (requires spec change approval)
- **C**: Modify test (requires formal spec change + governance record)

**Action**: Do NOT unilaterally modify test. Escalate design decision. Document reasoning.

### Case 3: Test is Flawed (rare)
```
TASK-005 [RED]: "Order calculates tax on subtotal" — COMPLETE
  → Test: order.Tax == subtotal * 0.08
  
TASK-006 [IMPL]: "Order calculates tax" — IN PROGRESS
  → Implementation: tax = subtotal * taxRate (where taxRate from config)
  → Test fails: expects 0.08, config says 0.07
  → Test was written without checking config source
```

**Action**: 
1. Do NOT modify test in [IMPL] task
2. Create governance issue: `governance/issues/tax-rate-test-conflict.md`
3. Escalate to spec owner
4. Get formal decision: update test? update implementation? update spec?
5. Only after decision, modify test (with justification recorded)

---

## Verification Scripts

### Manual Verification

```bash
# Check TDD pattern in git history
git log --oneline <branch> | grep -E "test\(|impl\("
# Should alternate: test(…) [RED], impl(…), test(…) [RED], impl(…)

# Verify no test mutations
git log -p <branch> -- '**/tests/**/*.md' | grep '^-' | grep -v '^---' | head -20
# Should show minimal changes (only completed refinements with justification)

# Check specific task
git log --oneline <branch> | grep "TASK-001"
# Should show: test(…)[RED], impl(…), nothing else
```

### Automated Verification (Recommended)

Add to CI/CD pipeline:

```bash
#!/bin/bash
# check-tdd-discipline.sh

branch=$1
commits=$(git log --oneline $branch | wc -l)

echo "Scanning $commits commits for TDD discipline..."

# Find all [RED] test commits
red_commits=$(git log --oneline $branch | grep '\[RED\]' | wc -l)
# Find all impl commits
impl_commits=$(git log --oneline $branch | grep 'impl(' | wc -l)

if [ "$red_commits" -ne "$impl_commits" ]; then
  echo "❌ VIOLATION: RED tests ($red_commits) != IMPL commits ($impl_commits)"
  exit 1
fi

# Check ordering (should alternate)
git log --reverse --oneline $branch | grep -E 'test|impl' | while read commit msg; do
  # Track state: RED, IMPL, RED, IMPL...
  # Fail if we see RED after IMPL without IMPL between
done

echo "✅ TDD discipline verified"
```

---

## Summary of Rules

| Rule | Enforced By | Violation |
|------|-------------|-----------|
| Tests are immutable after [RED] commit | Task template + checklist | Test file shows modifications after commit |
| [RED] and [IMPL] must alternate | Plan template (Task Index) | Consecutive [RED] or [IMPL] tasks |
| Each [RED] creates exactly one test | Phase field + test path | Multiple assertions in single [RED] test |
| Each [IMPL] passes exactly one test | Verification checklist | Implementation changes > one test behavior |
| Implementation cannot modify tests | Checklist: "Test file identical to [RED]" | Git diff shows changes to test files in [IMPL] commit |
| Tests must fail before [IMPL] | Checklist: "run test, confirm fails" | [RED] commit with passing test |
| Tests must pass after [IMPL] | Checklist: "run test, must pass ✓" | [IMPL] commit with failing test |
| No skipping [RED] phase | Task structure | Jumping to [IMPL] without [RED] |
| No skipping [IMPL] phase | Task structure + plan | Writing test without follow-up implementation |

---

## References

- **TDD Framework Document**: `C:\Users\chris\Downloads\TDD_FRAMEWORK.md` (source)
- **SpecForge Contract**: `/SpecForge-Contract.md` (specification enforcement)
- **Task Template**: `/tasks/{{TASK_ID}}.template.md` (embedded enforcement)
- **Plan Template**: `/planning/{{PLAN_ID}}.template.md` (cycle structure)

---

**Last Updated**: March 3, 2026  
**Owner**: SpecForge Governance  
**Status**: ACTIVE — Non-negotiable requirement for all SpecForge projects
