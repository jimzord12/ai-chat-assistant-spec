# SPEC STATE

> **Last Updated:** 2026-06-30
> **Spec Book Status:** IN PROGRESS
> **Chapters Locked:** 2 / 12

---

## Current Focus

| Field | Value |
|------|-------|
| **Active Chapter** | Ch 00: Agent Conventions & Shared Contract |
| **Chapter Stage** | NOT STARTED |
| **Decisions Resolved** | 16 / ~150-200 (estimated) |
| **Open Questions** | — |
| **Last Session** | 2026-06-30 |
| **Last Session Summary** | Ch 02 scoped, self-reviewed, and locked in one session (D-009 to D-016). 5 cross-chapter impacts queued. Ch 00 is now unblocked. |
| **Next Recommended Action** | Scope Ch 00: Agent Conventions & Shared Contract — both Ch 01 and Ch 02 are LOCKED, dependency satisfied. |
| **Blocked Items** | None |

---

## Chapter Status Board

| Chapter | Stage | Decisions (resolved/est) | Open Q's | Depends On | Blocks | Last Touched |
|---------|-------|--------------------------|----------|------------|--------|--------------|
| Ch 01: Overview & NFRs | **LOCKED** | 8 / 8 | 0 | — | Ch 02 | 2026-06-30 |
| Ch 02: Architecture & Topology | **LOCKED** | 8 / 8 | 0 | Ch 01 ✅ | Ch 00 | 2026-06-30 |
| Ch 00: Agent Conventions & Shared Contract | NOT STARTED | 0 / 15-25 | — | Ch 02 ✅ | Ch 06, Ch 05 | — |
| Ch 06: Config & Extensibility | NOT STARTED | 0 / 12-18 | — | Ch 00 | Ch 05, Ch 04 | — |
| Ch 05: Data Layer & Storage | NOT STARTED | 0 / 15-20 | — | Ch 00, Ch 06 | Ch 04 | — |
| Ch 04: Backend: API & Streaming | NOT STARTED | 0 / 18-25 | — | Ch 06, Ch 05 | Ch 07, Ch 08, Ch 03, Ch 10 | — |
| Ch 07: Auth, Security & Rate Limiting | NOT STARTED | 0 / 12-18 | — | Ch 04 | Ch 03, Ch 10 | — |
| Ch 08: Guardrails & Moderation | NOT STARTED | 0 / 8-12 | — | Ch 04 | Ch 09 | — |
| Ch 03: Frontend: Widget & UI | NOT STARTED | 0 / 15-25 | — | Ch 04, Ch 07 | Ch 09 | — |
| Ch 09: Testing Strategy | NOT STARTED | 0 / 10-15 | — | Ch 03, Ch 08 | Ch 11 | — |
| Ch 10: Deployment & Observability | NOT STARTED | 0 / 8-12 | — | Ch 07, Ch 04 | Ch 11 | — |
| Ch 11: Phased Delivery & Acceptance Criteria | NOT STARTED | 0 / 5-10 | — | Ch 09, Ch 10 | — | — |

---

## Cross-Chapter Impact Queue

| Impact ID | From Chapter | To Chapter | Description | Status |
|-----------|--------------|------------|-------------|--------|
| IMP-001 | Ch 01 (D-005) | Ch 05 | Storage must be adapter-based; IndexedDB is Phase 1 default | Pending |
| IMP-002 | Ch 01 (D-003) | Ch 04 | Backend must implement Vercel AI SDK v7 streaming contract | Pending |
| IMP-003 | Ch 01 (D-006) | Ch 07 | Auth is host-app responsibility; component receives userId prop only | Pending |
| IMP-004 | Ch 01 (D-008) | Ch 06 | Single config per instance; no internal persona routing needed | Pending |
| IMP-005 | Ch 02 (D-009) | Ch 00 | Monorepo structure defines folder conventions and import paths | Pending |
| IMP-006 | Ch 02 (D-010) | Ch 00, Ch 03 | Package names define all import statements across the spec | Pending |
| IMP-007 | Ch 02 (D-011) | Ch 03 | Headless + styled pattern defines component API surface | Pending |
| IMP-008 | Ch 02 (D-013) | Ch 00 | ESM-only output constrains all import/export conventions | Pending |
| IMP-009 | Ch 02 (D-016) | Ch 04 | Example app API route is the canonical backend contract reference | Pending |

---

## Recent Sessions Log

| # | Date | Chapter | Session Type | Decisions Resolved | Notes |
|---|------|---------|-------------|--------------------|---------|
| 1 | 2026-06-30 | — | Protocol Setup | 0 | Established spec protocol, dependency graph, write order, chapter template, and state file. Repo pushed to GitHub. |
| 2 | 2026-06-30 | Ch 01 | Scoping | 8 | All 8 scoping questions answered. D-001 to D-008 created. Chapter advanced to SCOPED. |
| 3 | 2026-06-30 | Ch 01 | Review + Lock | 8 | Self-review checklist passed. Chapter fast-tracked SCOPED → LOCKED. 4 cross-chapter impacts queued. |
| 4 | 2026-06-30 | Ch 02 | Scoping + Lock | 8 | All 8 questions answered (D-009 to D-016). Self-review passed. Chapter fast-tracked to LOCKED. 5 impacts queued. |

---

## Quick-Reference: Next-Action Decision Tree

1. **Pending cross-chapter impacts?** → Run Propagation Session
2. **Any chapter in COMPLETE (not REVIEWED)?** → Run Review Session
3. **Any chapter in IN PROGRESS?** → Continue grilling (next open question)
4. **Any chapter in REVIEWED?** → Lock it, update state, re-evaluate
5. **Any chapter CREATED with deps ≥ IN PROGRESS?** → Scope it (identify 15-30 questions)
6. **Any chapter SCOPED with deps ≥ IN PROGRESS?** → Start grilling (first session)
7. **All chapters LOCKED?** → Spec book complete. Ready for implementation.
8. **All in-progress chapters blocked?** → Report blocker chain, find root cause.
