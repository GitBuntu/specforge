# SpecForge — Domain‑First, Requirement‑Driven Engineering

SpecForge is a lightweight workflow for building software in a strict, auditable sequence.  
It uses a chain of Markdown templates to ensure that every piece of work is grounded in the domain and traceable from context → requirement → feature → scenario → test.

There is no CLI and no tooling.  
The workflow is enforced by the structure of the templates themselves.

---

## Quick Start: How to Use SpecForge

🔴 **This SpecForge directory contains TEMPLATES ONLY.** Do not create feature files, requirements, or tests here.

### For End Users (You)

1. **Have SpecForge available** — clone/download the SpecForge repo.
2. **Point to your project** — invoke SpecForge on your project directory.
3. **Describe your requirements** — tell the LLM what you want to build.
4. **SpecForge handles the rest automatically** — see below.

Example (for reference):
```
Tell the LLM:

  "Execute SpecForge on D:\source\SpecForgeTest
   
   SpecForge template library location: D:\source\SpecForge
   Bounded Context: health-monitoring
   
   Requirements:
   1. Add a .NET console app that runs as a health monitoring service
   2. Add a health check HTTP endpoint that returns the service's status and uptime"
```

**That's it.** The LLM will automatically:
- Check if `features/health-monitoring/` already exists
  - If yes: work ONLY within that existing feature directory
  - If no: create it
- Copy and rename all templates (skip any that already exist)
- Fill in the Context
- Generate Requirements, Features, Scenarios, Tests, Plans, Tasks
- Stop when all tasks are complete
- Optionally apply visual mapping to completed requirements (post-task, optional)
- **NEVER delete or reorganize existing features**

**Visual Mapping (Optional Post-Task Step)**
After all tasks complete, you can optionally generate visual maps for each requirement. Visual mapping does NOT modify immutable test artifacts (locked after [RED] phase). Instead, it validates that requirement concepts are explicit and test coverage is complete. See [VISUAL-MAPPING-DISCIPLINE.md](VISUAL-MAPPING-DISCIPLINE.md) for the 5-pass method.

### For LLMs (Automatic Bootstrap)

When invoked on a project, you MUST automatically:

1. **Identify the project root** — the directory where you're invoked.
2. **Extract the bounded context name** from the user's requirements.
   - Example: "health check service" → "health-monitoring"
3. **Detect technology stack** — If React is detected, also consult [REACT-DISCIPLINE.md](REACT-DISCIPLINE.md) for component mapping, testing patterns, and folder organization.
4. **Check if the feature already exists** — look for `{{project-root}}/features/{{bounded-context}}/`
   - If it exists: **Work ONLY within that existing directory. Do not delete, reorganize, or overwrite existing artefacts.**
   - If it does not exist: Proceed to step 5.
5. **Create the directory structure** (only if it doesn't already exist):
   ```
   {{project-root}}/features/{{bounded-context}}/
   ├─ domain/
   ├─ requirements/
   ├─ features/
   │  └─ {{feature-name}}/
   ├─ tests/
   │  └─ {{feature-name}}/
   ├─ planning/
   └─ tasks/
   ```
6. **Copy and rename all templates** from SpecForge into the feature directory (skip any that already exist; remove all `{{PLACEHOLDER}}` names).
7. **Fill in the Context file (C1)** — extract the domain model from requirements.
8. **Proceed through the chain** — Context → Requirement → Feature → Scenario → Test → Plan → Task.

**DO NOT ask the user to manually copy templates or create directories. Bootstrap is automatic.**
**DO NOT delete, reorganize, or overwrite existing features or artefacts. Existing files are sacred.**

If invoked on an existing project with partial templates, identify the first incomplete artefact in the current feature and continue from there.

---

## Key Selling Points

### 1. **Zero Accidental Test Duplication**
Every requirement, scenario, and test case is uniquely identified and traceable. The structural enforcement of atomicity (one requirement = one rule) makes duplicate tests structurally impossible, not just discouraged. Test volume stays minimal and deterministic.

### 2. **Audit-Ready by Design**
The SpecForge Contract enforces verbatim requirement traceability from domain → requirement → feature → scenario → test. Every artefact references the requirements that justify it. Regulators and auditors can walk the chain without reinterpretation.

### 3. **No Requirement Drift**
The Contract forbids reinterpretation, summarization, or restating requirement text. Requirements flow forward exactly as written. Implementation cannot diverge from specification because the specification is the enforcer, not the output.

### 4. **EARS + Gherkin by Structural Enforcement**
Templates enforce both EARS (for requirements) and Gherkin (for scenarios) through their structure, not through naming or documentation. The format itself is the rulebook — engineers cannot deviate without breaking the chain.

### 5. **Deterministic Test Volume Prediction**
Because requirements are atomic and scenarios are derived systematically, test volume is predictable. A simple rule generates 3–10 test cases; a moderate rule generates 10–30; a complex rule generates 30–200. The explosion is quantifiable, not surprising.

### 6. **No Skipped Steps**
The linear chain (Context → Requirement → Feature → Scenario → Test → Plan → Task) is enforced by template dependencies. You cannot create tests without scenarios, scenarios without requirements, or requirements without context. The workflow is idempotent and reproducible.

### 7. **Scenario Outlines Collapse Permutations Automatically**
Data-driven variation is handled once, in the scenario outline. This eliminates 30–70% of accidental test duplication without losing coverage. Engineers cannot accidentally add redundant scenarios because the template structure prevents it.

### 8. **Invariants and Domain Events as Enforcement**
Requirements must reference ≥1 invariant and ≥1 domain event from context. This prevents invented behaviour and ensures every requirement preserves the domain model. Invariants become structural constraints, not suggestions.

### 9. **Single Source of Truth**
Artefacts are not generated; they are the source of truth. Templates are read-only. Engineers cannot edit, reinterpret, or regenerate artefacts mid-chain. This eliminates drift and makes the workflow deterministic and reproducible across any team or generation.

### 10. **Traceability Without Tooling**
Every artefact contains references to the requirements, scenarios, and tests that justify it. This traceability is baked into the Markdown structure itself. No database, no CI/CD, no integration — the files are the audit trail.

### 11. **Built-In Defence Against Scope Creep**
The Contract rule "MUST NOT introduce new invariants or events" prevents scope creep from sneaking into features, scenarios, or tests. The domain model stays fixed. Scope expansion requires a new requirement, which is visible and auditable.

### 12. **Task Decomposition Grounded in Tests**
The Plan artefact decomposes test cases into executable tasks. This is the reverse of traditional development: start with the test, then plan the work. This ensures implementation never outpaces verification.

### 13. **Syntax Enforced Through Structure, Not Through Naming**
SpecForge doesn't require engineers to "remember" to use EARS or Gherkin — the file format and template structure enforce it. A `.feature` file with `Given/When/Then` blocks is Gherkin. A `REQ-*.md` file with EARS patterns is atomic requirements. The syntax is self-documenting and impossible to violate without breaking the chain. No policy documents, no training, no deviation.

### 14. **Deterministic, Repeatable Engineering Across Sessions**
SpecForge encodes every step directly into artefacts, so the LLM never guesses, never infers, and never depends on session memory. Every decision is recorded in the Chain Position and Next Step Directive. This produces consistent, reproducible outputs across sessions, models, and engineers — the workflow is idempotent regardless of interruptions or context gaps.

### 15. **Scales Cleanly to Large Features and Large Teams**
Because each artefact is isolated and state is encoded in files, SpecForge scales without confusion or collisions:
- Hundreds of requirements across multiple features
- Dozens of scenarios per feature  
- Hundreds of test cases
- Hundreds of tasks across teams

No centralized database. No merge conflicts. No coordination overhead.

### 16. **Automatic Resume and Progress Tracking**
The Plan's Task Index and Resume Logic allow any LLM to restart work at any time, on any machine, without session history. The LLM reads the Task Index, finds the first NOT STARTED task, and continues deterministically. This makes the workflow robust to interruptions, model changes, and team hand-offs.

### 17. **Eliminates Assumption-Driven Development**
SpecForge forces explicit articulation of invariants, domain events, requirements, behaviour, and tests. The Contract forbids inference: "MUST NOT infer missing behaviour." Nothing is implicit. Nothing is hidden. This removes the silent assumptions that normally cause rework, defects, and regulatory findings.

### 18. **Model-Agnostic and Tool-Agnostic**
SpecForge works with any LLM (GPT, Claude, Gemini, local models) and any IDE (VS Code, JetBrains, browser tools). Because the rules live in Markdown and the state lives in files, there is no lock-in, no proprietary integration, and no dependency on a particular toolchain. The workflow is portable across any environment.

### 19. **Session-Independent Workflow**
SpecForge does not rely on chat history or model memory. All state is encoded directly inside artefacts through Chain Position, Next Step Directive, Task Index, and Completion State. This makes the workflow restartable at any time, on any model, without loss of continuity — the resumption logic is completely deterministic and requires no context beyond reading the files.

### 20. **Template Hardening Prevents LLM Overwrites**
Templates use visible, non-comment hardening blocks and live in a dedicated `.specforge-templates/` folder separate from generated artefacts. This prevents LLMs from accidentally overwriting templates — solving a failure mode that occurs in unstructured workflows.

### 21. **Explicit State Machine Encoded in Markdown**
Each artefact contains a Chain Position, Next Step Directive, Traceability block, and Contract rules. This turns the entire workflow into a deterministic state machine that the LLM must follow. No interpretation, no branching, no alternative paths — the state machine is visible and enforced.

### 22. **Plan-Driven Task Orchestration**
The Plan artefact contains the authoritative Task Index, resume logic, task sequencing, and completion state. This makes task execution predictable and scalable, even with 100+ tasks. Tasks are not invented on-the-fly; they are orchestrated from the Plan.

### 23. **LLM-Safe Task Progression**
Tasks include Completion State, Task Update Rules, and Next Task Directive. This ensures the LLM:
- Marks tasks complete (not invents new states)
- Updates only the correct task (not multiple tasks)
- Moves to the next NOT STARTED task (not skips or reorders)
- Never generates multiple tasks (not creates a backlog)
- Never skips tasks (strictly sequential)

This is a unique, built-in safety mechanism.

### 24. **Verbatim Requirement Propagation**
Requirements are copied forward exactly into Feature, Scenario, Test, Plan, and Task. The Contract forbids summarizing, paraphrasing, or rewording. This prevents drift and ensures every downstream artefact is anchored to the original requirement text.

### 25. **Strict No-Reinterpretation Enforcement**
Every artefact includes explicit no-reinterpretation rules that forbid summarizing, paraphrasing, inferring behaviour, adding assumptions, or introducing new invariants/events. This is a core anti-drift mechanism built into every template.

### 26. **Multi-Artefact Traceability**
SpecForge enforces traceability across requirements, scenarios, tests, plans, and tasks. Every artefact must reference the IDs it depends on. This creates a complete, auditable chain of evidence with no missing links.

### 27. **Feature-Scoped Isolation**
Each feature has its own requirements, scenarios, tests, plan, and tasks. This prevents cross-feature contamination, keeps the LLM focused on a single bounded context, and allows features to be developed in parallel without collision.

### 28. **LLM-Enforced Termination Conditions**
The Task template explicitly states: "This is the final artefact in the chain" and "Stop after generating exactly one task." This prevents runaway generation and ensures deterministic, bounded stopping.

### 29. **Location-Independent, Model-Independent Design**
Because all rules live inside artefacts and templates (not in a database or CLI), any LLM can execute the workflow, any IDE or environment can host it, and no proprietary features are required. This is a factual property of the architecture.

### 30. **Human-Readable, Machine-Enforceable Governance**
The SpecForge Contract lives in a dedicated file that both humans and LLMs read and enforce. This gives you a single, authoritative governance document that applies uniformly across all workflows and all team members.

### 31. **No Hidden State, No Ambiguity**
SpecForge eliminates implicit assumptions, hidden rules, invisible state, and reliance on conversation context. Everything is encoded in the artefacts themselves. This removes the hidden complexity that normally causes defects in large projects.

### 32. **Supports Large-Scale Decomposition**
Because tasks are flat, isolated, stateful, and traceable, SpecForge can handle 100+ tasks, 500+ tasks, even 1000+ tasks without losing determinism or continuity. The workflow scales linearly with project size.

### 33. **Prevents LLM Creativity Where It Is Harmful**
SpecForge's rules explicitly forbid inventing new behaviour, adding new requirements, modifying domain events, altering invariants, or generating extra artefacts. This is a factual constraint built into every template, preventing the well-intentioned LLM drift that causes silent defects.

### 34. **Mathematically Deterministic Resume Logic**
The Plan's Task Index uses a formula-based resume mechanism: `MIN(TASK-NNN where Status == "NOT STARTED")`. This means: find the task with the lowest ID that has NOT STARTED status. No ambiguity, no user choice, no LLM interpretation. Resume is automatic, repeatable, and idempotent across any model or session.

### 35. **DDD-First Architecture**
SpecForge is grounded in Domain-Driven Design. Bounded Contexts, Aggregates, Value Objects, Invariants, and Domain Events are not afterthoughts—they are first-class citizens that appear in Context, flow through Requirements, and are preserved (never invented) in Feature, Scenario, Test, Plan, and Task. This ensures the domain model remains stable and intentional.

### 36. **File Existence Validation & Broken Chain Prevention**
Every artefact validates that all referenced files exist. If a requirement, scenario, test, or task file is missing, the chain breaks — this is a fatal error, not silently ignored. This prevents orphaned requirements, dangling references, and hidden gaps that normally cause silent defects.

### 37. **Zero-Gap Coverage Verification**
Structural enforcement of coverage completeness: every test case listed in a Plan must have a corresponding planning step, and every assertion in a test must appear in a task description. Gaps are structural impossibilities, not oversight risks.

### 38. **Domain-Only Constraint: Forbidden Terms**
Approach descriptions MUST use only domain concepts (Aggregate names, Value Objects, Invariants, Domain Events, properties, state). Infrastructure, frameworks, patterns, and technology terms are structurally forbidden. This forces engineers to think in domain terms, not implementation details, and prevents premature architectural decisions.

### 39. **Bidirectional Traceability Enforcement**
Requirements must link backward to Context (reference ≥1 invariant OR ≥1 event). Features must reference upstream Requirements. This creates a bidirectional chain that prevents invented behaviour and ensures every requirement is grounded in the domain model.

### 40. **Structural State Guards on Task Completion**
Task artefacts include auto-blanking fields: Completed By and Completed On are only filled when Status==COMPLETE; otherwise they are auto-blanked. This prevents partial states, conflicting completion data, and silent corruption of the task index.

### 41. **Atomic Requirements by Design**
The EARS pattern and Requirement Statement section enforce atomicity: one requirement = one unambiguous business rule. The structure makes bundling multiple rules impossible. This prevents requirement bloat and ensures test volume is predictable.

---

## 1. Workflow Overview

SpecForge defines a strict, linear chain with **7 artefact types**:

1. **Context** (C1) — define the domain, bounded context, aggregates, value objects, invariants, and domain events.
2. **Requirement** (C2) — express atomic behaviour using EARS syntax, referencing ≥1 invariant and/or ≥1 domain event from Context.
3. **Feature** (C3) — describe how one or more requirements are realised; list only aggregates, invariants, and events from Context.
4. **Scenario** (C4) — define concrete examples using Gherkin (Given/When/Then), tagged with requirement IDs, demonstrating domain events and invariant preservation.
5. **Test** (C5) — assert that invariants are preserved and domain events are emitted when scenarios execute; verify requirement outcomes.
6. **Plan** (C6) — decompose test cases into discrete planning steps grounded in specific test assertions; list all tasks required to satisfy requirements.
7. **Task** (C7) — execute a discrete unit of work; update status (NOT STARTED → IN PROGRESS → COMPLETE); resume logic uses MIN(TASK-NNN) formula.

Each step depends on all previous steps.  
No step may be skipped or created out of order.  
The chain is idempotent: interrupted work can be resumed at any time using the Task Index resume formula.

---

## 2. Multi-Feature Organization

SpecForge is designed for **multi-developer environments** where multiple features are developed simultaneously.

**Key principle**: Each feature (bounded context) is a **completely independent SpecForge instance**. Features use **semantic names** (not sequential numbers) to avoid collisions and enable parallel development.

### Directory Structure

⚠️ **CRITICAL**: This structure is created **in your PROJECT workspace**, NOT in the SpecForge directory. SpecForge is templates only. When you invoke SpecForge on a project, you create this structure in that project's root directory.

```
features/
├─ health-monitoring/
│  ├─ domain/
│  │  └─ 00-context.md
│  ├─ requirements/
│  │  ├─ REQ-001.md
│  │  └─ REQ-002.md
│  ├─ features/
│  │  └─ monitoring-alerts/
│  │     ├─ spec.md
│  │     ├─ SCENARIO-001.feature
│  │     └─ SCENARIO-002.feature
│  ├─ tests/
│  │  └─ monitoring-alerts/
│  │     ├─ TEST-001.md
│  │     └─ TEST-002.md
│  ├─ planning/
│  │  └─ PLAN-001.md
│  └─ tasks/
│     ├─ TASK-001.md
│     └─ TASK-002.md
│
└─ chronic-illness-medication/
   ├─ domain/
   │  └─ 00-context.md
   ├─ requirements/
   │  ├─ REQ-001.md
   │  └─ REQ-002.md
   ├─ features/
   │  └─ medication-adherence/
   │     └─ ... (same structure)
   ├─ tests/
   ├─ planning/
   └─ tasks/
```

### Why Semantic Names, Not Sequential Numbers?

| Approach | Problem |
|----------|---------|
| ❌ FEATURE-001, FEATURE-002 | Collision risk, no semantic meaning, requires coordination |
| ✅ health-monitoring, chronic-illness-medication | Self-documenting, no collisions, developers use bounded context name |

### Key Benefits of Semantic Multi-Feature Organization

1. **Zero Collision Risk** — Semantic names (health-monitoring, chronic-illness-medication) prevent simultaneous-developer collisions. No coordination needed to pick the next feature number.

2. **Parallel Development** — Multiple developers work independently on different features without conflicts, merges, or blocked branches. Each feature is a complete isolated instance.

3. **Independent Releases** — Each feature can be released, versioned, and maintained as a separate codebase. No entanglement with other features. Enables independent deployment pipelines.

4. **Self-Documenting** — Directory names match bounded context names (health-monitoring, not FEATURE-003). Context is immediately obvious; no need to look up what "FEATURE-002" does.

5. **Isolated Maintenance** — Changes to one feature (bug fix, requirement update, refactor) do NOT touch or affect other features. Feature maintenance is scoped and safe.

---

## TDD Discipline: Preventing Implementation Cheating

SpecForge enforces **Test-Driven Development (TDD)** to prevent LLMs and engineers from modifying tests instead of fixing implementation.

**Read**: [`TDD-DISCIPLINE.md`](TDD-DISCIPLINE.md) (CRITICAL for understanding task execution)

**Key enforcement rules**:
- Tests are immutable after [RED] commit
- Tasks alternate [RED] → [IMPL] → [RED] → [IMPL]
- Implementation cannot pass tests by changing tests
- Git history is auditable at atomic level
- Violations are detected and escalated

This governance is embedded directly into the Plan and Task templates — no manual discipline required.

---

### Single Developer, Multiple Features

If developing locally in your **project workspace** (not in SpecForge):
1. Create directory: `features/health-monitoring/` (in your project, not in SpecForge)
2. Copy all 10 templates into appropriate subdirectories
3. Work through the complete SpecForge chain
4. When starting a new feature, create `features/next-feature-name/` (in your project) with fresh copies of templates
5. Each feature is isolated; no shared directories

### Multi-Developer Environment

- Developer A works on `features/health-monitoring/` independently
- Developer B works on `features/chronic-illness-medication/` independently
- No coordination needed; parallel development is safe
- Features can be released, maintained, and versioned separately
- Each feature has its own git branch, CI/CD pipeline, codebase

---

## 3. Folder Structure

