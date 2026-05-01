Status: Approved

# FR-1.1: Dry-run pipeline initiates planner phase

The system shall allow a dry-run pipeline to invoke the planner role and produce a phase plan artifact without calling a live LLM.

## Acceptance Criteria

- Planner runs and writes a `phase_plan_*.md` artifact.
- No real LLM API is called when `DRY_RUN=1`.

---

**FR Document ID:** FRD-001
Status: Approved
