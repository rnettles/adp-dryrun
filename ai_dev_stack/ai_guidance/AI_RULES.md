# AI Development Rules


All AI agents must follow these rules.

Runtime source-of-truth:

- Runtime policy: `ai_dev_stack/ai_guidance/AI_RUNTIME_POLICY.md`
- Runtime gates: `ai_dev_stack/ai_guidance/AI_RUNTIME_GATES.md`
- Runtime paths: `ai_dev_stack/ai_guidance/AI_RUNTIME_PATHS.md`
- Side-quest process: `ai_dev_stack/ai_guidance/AI_SIDE_QUEST_PROCESS.md`
- Intake process: `ai_dev_stack/ai_guidance/AI_INTAKE_PROCESS.md`
- Phase process: `ai_dev_stack/ai_guidance/AI_PHASE_PROCESS.md`
- Design process: `ai_dev_stack/ai_guidance/AI_DESIGN_PROCESS.md`
- Architecture policy model: `ai_dev_stack/agent-architecture.md`

`AI_RULES.md` remains the highest-authority constraint file, while runtime execution references are centralized in the runtime files above.

**Governance Authority — Three-Layer Model**
This file is the **Layer 1** invariant authority across all execution contexts. Two execution paths consume it:

- **Copilot Chat Path (Layer 1 + Layer 2):** Agent files in `dotfiles/.github/agents/*.agent.md` (Layer 2) add human-session mechanics — "Always Load" sequences, HARD STOPs, operator confirmation gates. Agents load `AI_RULES.md` directly; Layer 1 changes are picked up automatically on the next session.
- **Platform Path (Layer 1 + Layer 3):** `ai-delivery-platform/platform/governance/` (Layer 3) adds platform mechanics — composed system prompts, TypeScript pre/post-condition guards, manifest-driven loading. `GovernanceService.getComposedPrompt()` injects Layer 1 invariants into every LLM call.

**Additive-only constraint:** Layers 2 and 3 may add execution-context mechanics. Neither may redefine, override, relax, or remove any invariant in this file. Conflicts are defects in Layer 2 or Layer 3.
> **Sync requirement:** If any safety rule, role boundary, lifecycle gate, or implementation limit in this file changes, `ai-delivery-platform/platform/governance/rules/process_invariants.md` (the Layer 1 distillation for the platform path) must be updated to match. Drift between them is a governance defect.
> **Reference:** ADR-009 (governance layer model), ai-delivery-platform ADR-031.
> **Precedence rule:** `ai_dev_stack/agent-architecture.md` is a governing architecture companion document. If prose in that document ever conflicts with Layer 1 invariants in this file or with runtime execution guides (`AI_RUNTIME_*`, `AI_PHASE_PROCESS.md`, `AI_INTAKE_PROCESS.md`, `AI_DESIGN_PROCESS.md`), this file and the runtime execution guides are authoritative. Treat conflicting architecture prose as documentation drift and update it.

**Human-AI Authority Boundary**
AI agents may create or edit any requirements or design artifact in the `Draft` / `Proposed` state.
Only the human operator may advance an artifact's status to `Approved`, `Accepted`, or any equivalent gate-passing state.
No AI agent may self-approve its own output or set intake status to `accepted`.
- AI may create FRDs (`Status: Draft`), ADRs (`Status: Proposed`), TDNs (`Status: Draft`), PRD amendments.
- Human sets: FRD `Status: Approved`, ADR `Status: Accepted`, TDN `Status: Approved`, intake `status: accepted`.
- The Planner operates in two modes: **intake-drafting mode** (produces Draft artifacts, stops for human review) and **phase-planning mode** (reads only Approved FRDs, produces a Draft phase plan).
- These modes must not be conflated. The Planner must not produce a phase plan in intake-drafting mode.
> **Platform enforcement:** The gate service must verify FRD `Status: Approved` before invoking the Planner in phase-planning mode. A Draft FRD must not satisfy the planning gate.
> **Reference:** ADR-008.

**Side Quest Rules**
- Side quests follow the full lifecycle defined in `AI_SIDE_QUEST_PROCESS.md`.
- Side quests are for internally discovered improvements and gaps only. Externally reported bugs (Jira tickets, support tickets) must use the Defect track (`AI_DEFECT_PROCESS.md`).
- A spec file in `project_work/ai_project_tasks/side_quests/SQ-*.md` is required before any slice is staged.
- One slice per task — do not implement multiple slices in a single task.
- No scope expansion beyond the spec during execution; log new scope as a separate intake or SQ item.

**Defect Rules**
- Defects follow the full lifecycle defined in `AI_DEFECT_PROCESS.md`.
- A spec file in `project_work/ai_project_tasks/defects/DEF-YYYYMMDD-##.md` is required before any slice is staged.
- One slice per task — do not implement multiple slices in a single task.
- If a defect fix requires a new or amended FR, an ADR change, or spans >3 slices: stop and route to Feature Intake (`INT-*`) instead.
- Reconciliation intake (`RC-*`) must be opened within 48 hours of any P0/P1 fix deployed without intake.

**Intake Rules**
- Do not implement any work whose scope requires an FR change without a corresponding accepted intake item.
- Do not create a new phase plan without a corresponding accepted intake item or explicit operator direction.
- Use the routing table in `AI_INTAKE_PROCESS.md` to decide: feature intake vs. defect vs. side quest vs. usability backlog.
- Externally reported production bugs with no FR/ADR impact are Defects (`AI_DEFECT_PROCESS.md`), not intake items.
- Reconciliation intake (`RC-*`) must be opened within 48 hours of any P0/P1 fix deployed without intake.

**Role Boundary Rules**
- Planner owns heavy design work and lifecycle planning: defines phase scope/objectives/FR coverage, creates development phase plans, identifies design artifacts/sequencing constraints, creates sprint plans, closes sprints, and closes phases.
- Planner never writes code.
- Sprint Controller owns task packaging and task closeout: prepares implementation instructions, selects the next incomplete sprint task, verifies/decomposes task size, stages active implementation brief/current task state, and archives previous execution state.
- Sprint Controller must not derive sprint plans from phase scope and must not perform final sprint closure.
- Verifier PASS handoff sequence is invariant: `Verifier PASS -> Sprint Controller task closeout -> Planner sprint close decision`.
- Sprint Controller task closeout must emit sprint-complete artifacts from the last completed task context; Planner must use passing gate artifacts to close the sprint.

**Phase Rules**
- Do not begin sprint staging from a phase in `Draft` status.
- Do not mark a phase `Complete` unless all checklist items are verified and exit criteria are met.
- Do not expand phase scope mid-sprint without an accepted intake item.
- Every sprint task must trace to a phase checklist item or an accepted intake item.
- Phase process is defined in `AI_PHASE_PROCESS.md`; use `ai_dev_stack/templates/TEMPLATE_phase_plan.md` for new phase plans.
- Planner must only plan FRs not already claimed by an existing phase plan.
  Cross-reference `staged_phases/*.md` to determine the claimed FR set before planning.
- If all known FRs are already covered by existing phase plans, the Planner must stop and
  report that there are no pending FRs — do not create an empty or redundant phase.
- Every phase plan must include a non-empty `fr_ids_in_scope` field referencing actual FR
  identifiers from the project’s FR/PRD documents. Phase plans without FR traceability are invalid.- **Planner (phase-planning mode) must not plan from FRDs that are not `Status: Approved`.**
  If no Approved FRDs exist, the Planner must stop and surface the list of Draft FRDs requiring
  human approval. Do not produce a phase plan from Draft requirements.
- Planner (intake-drafting mode) produces Draft FRDs/PRDs from intake context and stops for
  human review. It does not produce a phase plan and does not set intake status to `accepted`.
**Design Artifact Rules**
- Do not stage the first sprint of a phase that requires a TDN until that TDN has `Status: Approved`.
- Do not invent component interfaces, field schemas, or algorithm designs not specified in an approved TDN or architecture doc; flag the gap instead.
- Do not produce working code as output of a Spike; Spikes produce written findings only.
- Do not delete or overwrite an ADR; mark it `Deprecated` or `Superseded` and create a new one.
- An implementer that encounters a design contradicting an `Accepted` ADR must flag the conflict before implementing.
- Planner must evaluate all loaded ADRs for compliance and congruency before producing a phase plan.
  Any conflict with an Accepted ADR must be flagged in `required_design_artifacts` — do not silently ignore ADR conflicts.
- Planner must consider all related TDNs during phase design. Any TDN required before
  implementation can begin must be listed in `required_design_artifacts`.
- **AI agents (including the Planner) may draft ADRs (`Status: Proposed`) and TDNs (`Status: Draft`).**
  These drafts are not binding until the human operator sets them to `Accepted` / `Approved`.
  No implementer may be required to comply with a `Proposed` ADR or `Draft` TDN.
- Design process is defined in `AI_DESIGN_PROCESS.md`; use `ai_dev_stack/templates/TEMPLATE_tdn.md`, `ai_dev_stack/templates/TEMPLATE_adr.md`, and `ai_dev_stack/templates/TEMPLATE_spike.md` for new artifacts.
- Template ownership is invariant: `ai_dev_stack/templates/` is the only canonical editable source for machine- or agent-consumed templates.
- Do not maintain duplicate editable copies of templates under `docs/`.
- Files under `docs/` may contain produced artifacts and human reference material only.

**UX Artifact Rules**
- Do not advance a phase from `Planning → Active` for any user-facing feature without an approved `user_flow.md` at `project_work/ai_project_tasks/active/ux/user_flow.md`.
- Do not implement user-facing behavior without reading the approved `user_flow.md` first.
- Do not implement an interaction model different from what is defined in `user_flow.md`; flag conflicts to the planner.
- For UI or Hybrid features: a `wireframe.md` is required before staging Sprint 1. Do not stage without it.
- For complex UI features: a `ui_spec.md` is required. Component specs, data bindings, and error behavior in `ui_spec.md` are binding.
- Verifier must check UX Gate compliance and flow coverage — UX validation is not optional for user-facing tasks.
- UX artifacts are created using `ai_dev_stack/templates/TEMPLATE_user_flow.md`, `ai_dev_stack/templates/TEMPLATE_wireframe.md`, and `ai_dev_stack/templates/TEMPLATE_ui_spec.md`.
- The UX Architect agent is the primary owner of UX artifact creation. The planner may create draft artifacts when no UX Architect session is active, but they must be approved before implementation begins.

**Prototype Folder Rule**
- Do not reference any files in the prototype folders (e.g., document_tagger_prototype) in implementation or tests.
- Any files needed from the prototype system must be copied and integrated into the cks-platform project structure.
- The document_tagger_prototype folder is for reference only.

Safety Rules

- Never delete files unless explicitly instructed
- Never modify migrations unless required
- Never refactor unrelated modules
- Do not implement future sprint tasks

Code Quality

- Follow existing project conventions
- Prefer explicit typing
- Avoid overly complex abstractions

Implementation Limits

- Standard sprint mode:
  - Modify no more than 7 files per step
  - Keep changes under ~400 lines of code per step
- Fast Track sprint mode (operator-requested):
  - Expanded envelope: up to 12 files and ~1200 lines per step
  - Maximum ceiling: never exceed 15 files or ~1500 lines per step
  - Default planning posture: prefer larger bundled tasks over fine-grained task fragmentation
  - Bundled-task target: typically 3-7 tasks for a Fast Track sprint unless governance scope requires more
  - Each bundled task must map to explicit phase checklist item IDs in the sprint plan
  - Do not split one logical change into multiple tasks solely to mimic standard-sprint granularity
- Activation rules for Fast Track mode:
  - Any sprint may be requested as Fast Track by operator direction
  - Request must be recorded in the sprint plan execution mode and in `next_steps.md` as a Fast Track lane entry
  - The intake item (if any) and rationale must be linked from the lane entry
  - Fast Track mode expires when the sprint closes; standard limits resume automatically

Testing

- All new behavior must have tests
- Tests must include:
  - success case
  - failure case
  - edge case
- Execution-gate command baseline is maintained in `AI_RUNTIME_GATES.md`.
- A fixer must not exceed 3 repair attempts on a single task. After 3 consecutive FAIL verdicts, the fixer must stop, escalate to the operator with a summary of the failure pattern, and await explicit direction before attempting any further repair. This ceiling applies regardless of which orchestrator is active.

**Admin UI features (interactive / async)**
- Any new admin UI section that fetches data, renders filter controls, or supports pagination **must** include Playwright browser automation.
- The implementation brief for such a task must specify: server start command, validation URL, seed data shape, Playwright script path, and evidence format.
- Saying "no automated tests apply to a frontend JS change" is not acceptable for features that meet the above criteria.
- See `ai_dev_stack/ai_guidance/AI_TEST_GUIDELINES.md` (UI Automation section) for full requirements.