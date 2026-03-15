# Visual Mapping Discipline in SpecForge

**Purpose**: Provide a deterministic, visual-first method for understanding software requirements so that ambiguity is eliminated, design clarity is achieved, and visual learners can grasp complex flows at a glance.

**Scope**: This guidance applies to all SpecForge projects during Requirement and Feature review (artefacts 2–3) and can be extended during Task implementation for design verification.

---

## Why Visual Mapping Matters in SpecForge

SpecForge's textual requirements (EARS format) are precise but can obscure relationships and causality:

- **Ambiguous actor-system-data flows** — Which box depends on what? Is it Sequential or parallel?
- **Implicit invariants** — Are all domain objects covered or is something inferred?
- **Hidden side effects** — When are events emitted? Do they block or run alongside?
- **Unclear scope** — How many separate boxes/actors does this requirement actually touch?
- **Skipped steps** — What happens between "validates" and "stores"? Is a step hidden?

**With visual mapping**: Requirements become diagrams. Boxes are nouns (explicit actors, systems, data). Arrows are verbs (explicit actions). Every element is accounted for. Missing steps jump out. Ambiguity evaporates.

---

## The 5-Pass Method

### PASS 1: Domain Picture (Extract Nouns)

**Goal**: Create a left→right inventory of all domain elements (actors, UI surfaces, systems, data objects, domain events).

**Output**: One Mermaid diagram showing:
- All nouns from the requirement text
- Color-coded by type (orange=actor, blue=system, yellow=data, green=event)
- Ordered left→right: Actors → UI → Systems → Data → Events
- No arrows, only boxes

**Why**: Forces explicit identification of every entity without inference. If you can't name it in the requirement text, it doesn't exist.

---

### PASS 2: Interaction Picture (Extract Verbs)

**Goal**: Trace how actions flow through the domain picture, showing causality and blocking/non-blocking behavior.

**Output**: One Mermaid diagram showing:
- Vertical flow (top-to-bottom = time progression)
- All verbs from the requirement as numbered arrows
- Solid arrows (-->)  = primary sequence (blocking)
- Dashed arrows (-.->)  = side effects (non-blocking, events)
- Strict linear causality: no fanning, no loops

**Why**: Forces explicit sequencing. "Does A happen before B or in parallel?" Answer: look at the arrow order.

---

### PASS 3: Requirement Segments (Map Sentences → R-IDs)

**Goal**: Assert that every sentence or clause in the requirement maps to explicit boxes and arrows, and vice versa.

**Output**: Table with one row per sentence:
- R-ID (R1, R2, ..., Rn)
- Literal requirement text (copy-paste from source, no paraphrasing)
- Boxes referenced (e.g., "Box: Actor-1, System-1")
- Arrows referenced (e.g., "Arrow: 1, 5")

**Why**: Proves zero-gap coverage: every element in the requirement exists in the diagrams, and every element in the diagrams is justified by a requirement.

---

### PASS 4: Storyboard Panels (Timeline Visualization)

**Goal**: Break the interaction picture into 3–4 sequential snapshots, showing progression through the story.

**Output**: 3–4 mini-diagrams (one per panel), each showing:
- Panel title (3–5 words, descriptive)
- R-IDs covered in this panel
- Subset of boxes and arrows relevant to that step
- Only elements referenced by those R-IDs (no inferred extras)

**Why**: Helps visual learners see the story unfolding. "At step 3, which boxes are active? Which actors have exited the flow?"

---

### PASS 5: Validation (Non-Ambiguity Checklist)

**Goal**: Verify that the model is complete and unambiguous.

**Checklist**:

- [ ] All requirements have explicit trigger, actor, action, outcome (not inferred)
- [ ] All boxes have ≥1 requirement reference (no orphaned elements)
- [ ] All arrows have ≥1 requirement reference (no invented steps)
- [ ] Domain Picture maintains left→right order
- [ ] Interaction Picture maintains top→bottom causality
- [ ] Storyboard covers all requirements sequentially (no gaps, no overlaps)
- [ ] All verbs from requirements appear as arrows
- [ ] All nouns from requirements appear as boxes

**Why**: Enforces closure. If validation fails, the model is incomplete. Regenerate from Pass 1.

---

## When to Apply Visual Mapping

### ✅ **DO Apply** Visual Mapping To:

1. **Complex requirements** (>5 sentences, multi-actor flows)
2. **Event-driven scenarios** (especially domain events, side effects)
3. **Multi-step validations** (series of cascading checks)
4. **Cross-system interactions** (external services, OCR, AI models)
5. **Requirements with hidden ambiguity** ("stores", "validates", "retrieves" — what order? what sequences?)
6. **Design review sessions** (stakeholders need pictures, not prose)
7. **Test case derivation** (panels → test scenarios automatically)

### ❌ **Skip** Visual Mapping For:

1. **Trivial, single-action requirements** ("User logs in" — obvious, one box, one arrow)
2. **Well-understood domain** (if prose is already crystal-clear, skip it)
3. **Time-critical sprints** (if deadline is hard, prioritize complexity, not simplicity)

---

## Mapping to Code Implementation

Once you have the visual model, each element traces to code:

| Visual Element | Code Mapping | SpecForge Artefact |
|---|---|---|
| Box (Actor) | User role, user persona | Derived from Scenario "Given" blocks |
| Box (System) | Aggregate, Service, Controller | Domain model in Context (C1) |
| Box (Data) | Value object, DTO, domain event | DomainModels.cs |
| Box (Event) | Domain event, published event | Domain event class |
| Arrow (Verb 1-4) | Primary sequence → test assertions | TEST artefact (Assertion 1, 2, ...) |
| Arrow (Dashed) | Side effect → verify emitted, not blocking | TEST artefact (Assertion final) |
| Panel | Test scenario | SCENARIO file (Given/When/Then) |
| R-ID | Test requirement | Plan artefact links Assertions to Tasks |

**Example**: EXAMPLE-REQ-001-MAPPING.md has 7 panels that become 4 test cases (TEST-001 through TEST-004), each covering 1–2 panels.

---

## When to Update Visual Mapping

**Visual mapping is immutable once committed** (like tests in TDD):

- If a requirement changes, regenerate the visual mapping from Pass 1
- If the visual mapping and code diverge, the code is wrong (not the diagram)
- If stakeholders raise concerns about flow, regenerate and re-validate, then update code
- Never skip validation to "fix it faster"

---

## Visual Mapping in Post-Task Phase

### Why Post-Task Visual Mapping?

SpecForge enforces **test immutability after [RED] phase** — tests cannot be modified once locked. This constraint changes how visual mapping integrates:

- **Pre-task scenario**: Visual mapping *would* drive test creation (Requirement Segments → Test Assertions)
  - But SpecForge already locks tests after [RED] commit; no opportunity for visual mapping to influence locked tests
- **Post-task scenario** (recommended): Visual mapping serves as **validation and documentation tool**
  - Tests are already complete and locked
  - Visual maps validate that Requirement Segments (R1, R2, ..., Rn) align with test assertions
  - Coverage gaps are documented for future enhancement, not fixed in locked tests

### Post-Task Visual Mapping Workflow

**When**: After Plan.Task Index shows all tasks COMPLETE (optional, user-triggered)

**What**: For each requirement (REQ-{{REQ_ID}}.md), generate a visual map at `visual-mapping/REQ-{{REQ_ID}}-MAPPING.md`

**How**: Execute the 5-pass method (Passes 1–5 unchanged):
1. Pass 1: Domain Picture (Nouns → Boxes)
2. Pass 2: Interaction Picture (Verbs → Arrows)
3. Pass 3: Requirement Segments (Sentences → R-IDs)
4. Pass 4: Storyboard Panels (Timeline Visualization)
5. Pass 5: Validation (Coverage Checklist)

**Key Constraint**: Test Immutability Rule

⚠️ **Critical**: Tests are **locked immutable** after [RED] commit. Visual mapping cannot and must not modify tests.

- Requirement Segments (R1, R2, ..., Rn) should align with corresponding test assertions
- If visual mapping discovers gaps (e.g., "R3 has no test assertion"), **document the gap** in a Coverage Notes section
- **Do NOT create new tests or modify locked tests** to fill gaps
- Gap notes are for future scoping: "Consider adding test coverage for R3 in next iteration"

### Extension: Coverage Validation Table (Optional)

Add to visual map after Requirement Segments table (PASS 3):

**Coverage Mapping** (Post-Task Validation)

| R-ID | Requirement Text | Test File | Assertion | Status |
|---|---|---|---|---|
| R1 | ... | TEST-001.md | Assert: field == value | ✓ Covered |
| R2 | ... | TEST-002.md | Assert: event emitted with field == value | ✓ Covered |
| R3 | ... | *Not found* | *Gap* | ⚠️ Not Covered |

**Status Legend**:
- `✓ Covered` — Requirement segment has explicit test assertion
- `⚠️ Partial` — Requirement segment partially covered (implicit, inferred)
- `⚠️ Not Covered` — Requirement segment has no corresponding test assertion

If status is not `✓ Covered`, add note: "For future iteration" (do NOT modify locked tests)

### Output Location

Store visual maps at: `visual-mapping/REQ-{{REQ_ID}}-MAPPING.md`

Parallel to: `requirements/REQ-{{REQ_ID}}.md`

Example:
- `requirements/REQ-001.md` (source requirement)
- `visual-mapping/REQ-001-MAPPING.md` (visual map + coverage validation)
- `visual-mapping/REQ-002-MAPPING.md` (visual map + coverage validation)
- etc.

### Benefits of Post-Task Visual Mapping

1. **Requirement Clarity** — Validates that requirements are unambiguous (nouns clear, verbs explicit, steps sequential)
2. **Coverage Visibility** — Documents which requirement segments have test assertions (and which don't)
3. **Stakeholder Communication** — Provides diagrams for non-technical reviewers without changing locked tests
4. **Future Scoping** — Gap notes inform next iteration priorities
5. **Audit Trail** — Visual maps + test coverage form a complete traceability record

---

## Example Workflow

1. **Receive (or write) a requirement** — REQ-001.md with 3 paragraphs
2. **Open [VISUAL-MAPPING-TEMPLATE.md](./visual-mapping/VISUAL-MAPPING-TEMPLATE.md)**
3. **Execute 5 passes** (no skipping):
   - Extract all nouns → Domain Picture
   - Extract all verbs → Interaction Picture
   - Map sentences → Coverage table
   - Draw 4 storyboard panels
   - Run validation checklist
4. **Render Mermaid diagrams** in VS Code (preview pane)
5. **Is validation passing?** Yes → Save as `requirements/EXAMPLE-REQ-NNN-VISUAL.md`  
   No → Regenerate from Pass 1 (requirement may be incomplete)
6. **Review with team** — Show diagrams, not prose. "Does this match your intent?"
7. **Link from requirement** — Add cross-reference in `requirements/REQ-NNN.md`
8. **Proceed to Feature & Scenario** — Scenarios map to storyboard panels automatically

---

## Reference Examples

SpecForge provides two working examples in `visual-mapping/`:

- **[EXAMPLE-REQ-001-MAPPING.md](./visual-mapping/EXAMPLE-REQ-001-MAPPING.md)**
  - Simple: 7 requirements, 1 actor, 3 systems, 3 data objects, 1 event
  - Demonstrates basic pattern: HTTP → Aggregate → Repository
  - Good for first-time users

- **[EXAMPLE-REQ-002-MAPPING.md](./visual-mapping/EXAMPLE-REQ-002-MAPPING.md)**
  - Complex: 11 requirements, 0 actors, 6 systems, 6 data objects, 2 events
  - Demonstrates: multi-hop processing (OCR → AI), parallel validations, query-by-multiple-keys
  - Shows template scales to complexity without ambiguity

Copy the pattern from whichever example is closest to your requirement type.

---

## Anti-Patterns: What NOT To Do

❌ **Don't infer missing nouns**: "The system validates the document" — but no "Validator" box exists in the requirement? Don't invent it. Either the requirement is incomplete, or "validates" is implicit (pull it into the System box instead).

❌ **Don't skip arrows for brevity**: "Stores metadata in repository" — that's still an arrow. Label it. Numbering matters.

❌ **Don't fan arrows from one box**: If Aggregate Box has 3 outgoing arrows to different targets, they MUST be sequentially numbered (1, 2, 3) with clear dependencies, or the flow is ambiguous.

❌ **Don't mix domain events with data objects**: Events are state-changing side effects (green boxes, dashed arrows). Data objects are persistent facts (yellow boxes, solid data flow). Don't blur them.

❌ **Don't validate without the checklist**: Rushing validation means gaps slip through. Run the full checklist every time, even if you "think it's obvious."

---

## Tool Support

Visual mapping works with:

- **VS Code** — Mermaid preview extension (built-in)
- **GitHub** — Renders Mermaid natively in markdown
- **Obsidian, Notion** — Mermaid plugin support
- **Pure Markdown** — No tool required, just read the code blocks

No database, no proprietary software, no lock-in. Just Mermaid markdown.

---

## Summary

**Visual mapping solves a key SpecForge problem**: Requirements are textual, DDD is conceptual, and ambiguity hides until implementation. By forcing explicit nouns, verbs, and causality, visual mapping makes the requirement a blueprint before coding even begins.

**For visual learners**: Pictures > prose. Always.  
**For auditors**: Pictures are traceable. Every box has a requirement. Every arrow has a verb.  
**For teams**: Shared pictures kill ambiguity. Faster reviews, fewer reworkings.

Use it. Reference the template and examples. Validate every time.
