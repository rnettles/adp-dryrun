**Status:** Approved

# FR-1.3: Dry-run verifier failure causes implementer retry

When the active scenario forces a verifier FAIL, the pipeline shall pause in `paused_takeover` or route back to implementer, depending on execution mode.

## Acceptance Criteria

- Verifier FAIL increments `implementer_attempts`.
- In `next` mode, pipeline reaches `paused_takeover`.
- In `full-sprint` mode, implementer is retried automatically.

---

**FR Document ID:** FRD-001
**Status:** Approved
