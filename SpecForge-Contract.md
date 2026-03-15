# SpecForge Contract (Global)

🔴 **CRITICAL: SpecForge is TEMPLATES ONLY.** All artefacts must be created in your PROJECT workspace, not in the SpecForge directory.

To use SpecForge:
1. Copy templates from SpecForge into `your-project/features/{{feature-name}}/`
2. Remove template placeholders (e.g., rename `REQ-{{REQ_ID}}.template.md` → `REQ-001.md`)
3. Follow this contract ONLY in your project's feature directory
4. Never create or modify files in the SpecForge directory itself

This contract governs ALL SpecForge artefacts and templates.

The SpecForge framework enforces deterministic, requirement-driven development by ensuring **zero creative latitude** at every step. Rules are enforced via **template structure, not policies**.

## Multi-Feature Organization

SpecForge is designed for **multi-developer, multi-feature environments**. Each feature (bounded context) is a completely isolated SpecForge instance:

⚠️ **CRITICAL**: The following directory structure is created **in your PROJECT workspace**, NOT in the SpecForge directory. SpecForge contains only templates; actual artefacts are created in your project.

**Directory Structure** (per feature, in your project):
```
features/
├─ health-monitoring/             ← Feature/Bounded Context 1
│  ├─ domain/
│  │  └─ 00-context.md
│  ├─ requirements/
│  │  └─ REQ-001.md, REQ-002.md
│  ├─ features/
│  │  └─ monitoring-alerts/
│  │     ├─ spec.md
│  │     ├─ SCENARIO-001.feature
│  │     └─ SCENARIO-002.feature
│  ├─ tests/
│  │  └─ monitoring-alerts/
│  │     └─ TEST-001.md
│  ├─ planning/
│  │  └─ PLAN-001.md
│  └─ tasks/
│     └─ TASK-001.md
│
└─ chronic-illness-medication/    ← Feature/Bounded Context 2
   └─ (same structure as above)
```

**Key Rules**:
- Each feature uses **semantic naming** (health-monitoring, not FEATURE-001) to avoid simultaneous-developer collisions
- All artefacts for one feature are isolated in its own directory tree
- No shared directories; no cross-feature references
- Multiple developers can work on different features in parallel without coordination
- Features can be released, versioned, and maintained independently
- 🔴 **CRITICAL: NEVER delete, modify, or reorganize existing features.** Each feature directory is sacred. If the bounded context name already exists as a feature, ONLY work within that existing feature. Leave all other existing features untouched.

---

## Bootstrap / Initialization (Automatic)

When invoked on a **new project**, the LLM MUST automatically:

1. **Identify the project root** — the directory where the user invoked SpecForge.
2. **Detect technology stack** — If React is detected, also consult [REACT-DISCIPLINE.md](REACT-DISCIPLINE.md) for component-to-aggregate mapping, test assertion patterns, and folder organization during Task execution.
3. **Check if the feature already exists** — look for `{{project-root}}/features/{{bounded-context-name}}/`
   - If it exists, **work ONLY within that existing feature directory. Do not delete, reorganize, or create a new one.**
   - If it does not exist, proceed to step 4.
4. **Extract the bounded context name** from the user's requirements or domain description.
   - Example: "health check service" → bounded context = "health-monitoring"
5. **Create the feature directory structure** (only if it doesn't already exist):
   ```
   {{project-root}}/features/{{bounded-context-name}}/
   ├─ domain/
   ├─ requirements/
   ├─ features/
   │  └─ {{feature-name}}/
   ├─ tests/
   │  └─ {{feature-name}}/
   ├─ planning/
   └─ tasks/
   ```
6. **Copy and rename all templates** from SpecForge (only if files don't already exist):
   - `SpecForge/domain/00-context.template.md` → `{{project}}/features/{{context}}/domain/00-context.md`
   - `SpecForge/requirements/REQ-{{REQ_ID}}.template.md` → `{{project}}/features/{{context}}/requirements/REQ-001.md`
   - `SpecForge/features/{{FEATURE_NAME}}/spec.template.md` → `{{project}}/features/{{context}}/features/{{feature-name}}/spec.md`
   - All others similarly renamed (no template placeholders remain)
   - **SKIP any files that already exist. Do not overwrite or delete existing artefacts.**
7. **Fill in the Context file** (first artefact, C1) if it's empty:
   - Extract domain model from user requirements
   - Define Bounded Context, Aggregates, Value Objects, Invariants, Domain Events
   - Use only the user's domain concepts; no technology or framework names
8. **Proceed with the chain** — do not stop until task completion.
9. **(Optional) Apply Visual Mapping** after all tasks complete:
   - For each requirement (REQ-{{REQ_ID}}.md), optionally generate a visual map at `visual-mapping/REQ-{{REQ_ID}}-MAPPING.md`
   - Use the 5-pass method from [VISUAL-MAPPING-DISCIPLINE.md](../VISUAL-MAPPING-DISCIPLINE.md)
   - Visual maps validate that requirements are unambiguous and test coverage is complete
   - Visual maps cannot modify immutable tests (locked after [RED] phase)
   - This step is OPTIONAL; visual mapping is supplementary documentation, not required for completion

**Resume Logic**: If SpecForge is invoked on an **existing project** with partially filled templates, the LLM MUST:
1. Identify the first incomplete artefact in the CURRENT feature
2. Complete that artefact
3. Continue the chain
4. After all tasks complete (Plan.Task Index shows all COMPLETE), optionally offer to apply visual mapping
5. **Never touch other features.**

**The LLM MUST NOT ask the user to manually set up directories or copy templates.** Bootstrap is automatic.
**The LLM MUST NOT delete, rename, or reorganize existing features.** Existing features are immutable.

---

The LLM MUST obey the following rules at all times:

---

# Core Principles

## Chain Dependency
The SpecForge chain is strictly ordered:
```
Context → Requirement → Feature → Scenario → Test → Plan → Task
  (1)          (2)         (3)       (4)      (5)    (6)    (7)
```

**Strict Rule**: Artefacts downstream CANNOT be created until ALL upstream artefacts are complete.

## Structural Enforcement
Rules are enforced via **template structure and explicit formatting**, not via policies or guidelines:
- Forced field formats (Status: NOT STARTED | IN PROGRESS | COMPLETE)
- Mandatory copy-forward sections (exact syntax from upstream artefacts)
- File existence validation (fatal error if referenced file missing)
- Coverage gap detection (zero gaps: every test has planning step, every assertion has task description)
- Mathematical operation definitions (e.g., MIN(TASK-NNN where Status == "NOT STARTED"))
- Forbidden term lists (database, REST, HTTP, framework names, etc.)

## Domain-Driven Design Foundation
All SpecForge artefacts ground in Domain-Driven Design (DDD):
- **Context**: Defines Bounded Context, Aggregates, Value Objects, Domain Events, Invariants
- **Requirement**: References DDD concepts from Context (no invention)
- **Feature**: Lists Aggregates, Invariants, Events from Context (no new DDD concepts)
- **Scenario, Test, Plan, Task**: Preserve Aggregates, Invariants, Events from Context (no drift)

---

# 1. No Reinterpretation & Verbatim Copy-Forward

---

# 3. Traceability
Every artefact MUST reference:
- all relevant requirement IDs,
- all relevant scenario IDs,
- all relevant test IDs.

IDs MUST NOT be changed, removed, or invented.

---

# 4. Chain Position Enforcement
Every artefact MUST include a Chain Position block.

The LLM MUST:
- generate only the next artefact in the chain,
- never skip ahead,
- never regenerate an artefact,
- never create multiple artefacts at once.

---

# 5. Next Step Directive Enforcement
Every artefact MUST include a Next Step Directive.

The LLM MUST:
- follow the directive exactly,
- create the artefact at the specified path,
- use the specified template,
- stop after generating exactly one artefact.

---

# 6. Template Execution Rules
Templates are READ-ONLY.

The LLM MUST NOT:
- edit templates,
- fill in templates,
- update templates,
- overwrite templates.

Templates MUST live only in `.specforge-templates/`.

---

# 7. Task Execution Rules
For Task artefacts, the LLM MUST:
- update ONLY the current task’s Completion State,
- update ONLY the corresponding entry in the Plan’s Task Index,
- not modify any other task,
- not reorder tasks,
- not create new tasks.

---

# 8. Resume Logic
To resume work:
1. Open the Plan.
2. Read the Task Index.
3. Identify the first NOT STARTED task.
4. Instantiate that task using the Task template.
5. If all tasks are COMPLETE, stop.

The LLM MUST NOT ask the user what to do next.

---

# 9. Determinism
SpecForge MUST behave deterministically.

The LLM MUST:
- follow all blocks exactly,
- never guess,
- never infer,
- never rely on session memory,
- treat artefacts as the single source of truth.

---

# 10. Termination
The chain ends at the Task artefact.

The LLM MUST stop when:
- all tasks are COMPLETE, or
- the Task template’s Next Task Directive indicates no remaining tasks.
