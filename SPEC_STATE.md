# SPEC STATE

> **Last Updated:** 2026-07-01
> **Spec Book Status:** IN PROGRESS
> **Chapters Locked:** 3 / 12

---

## Current Focus

| Field | Value |
|------|-------|
| **Active Chapter** | Ch 06: Config & Extensibility |
| **Chapter Stage** | NOT STARTED |
| **Decisions Resolved** | 41 / ~150-200 (estimated) |
| **Open Questions** | — |
| **Last Session** | 2026-07-01 (session ended ~00:12 EEST) |
| **Last Session Summary** | Ch 00 fully scoped and locked in one pass (D-022 to D-041). 20 decisions covering type structure, error shape, Result pattern, naming conventions, import rules, and all shared type shapes. End-of-session wrap-up committed. |
| **Next Recommended Action** | Scope Ch 06: Config & Extensibility — Ch 00 now LOCKED. Target ~15 questions per calibration rules. |
| **Blocked Items** | None |

---

## Chapter Status Board

| Chapter | Stage | Decisions (resolved/est) | Open Q's | Depends On | Blocks | Last Touched |
|---------|-------|--------------------------|----------|------------|--------|--------------|
| Ch 01: Overview & NFRs | **LOCKED** | 8 / 8 | 0 | — | Ch 02 | 2026-06-30 |
| Ch 02: Architecture & Topology | **LOCKED** | 13 / 13 | 0 | Ch 01 ✅ | Ch 00 | 2026-06-30 |
| Ch 00: Agent Conventions & Shared Contract | **LOCKED** | 20 / 20 | 0 | Ch 02 ✅ | Ch 06, Ch 05 | 2026-07-01 |
| Ch 06: Config & Extensibility | NOT STARTED | 0 / 12-18 | — | Ch 00 ✅ | Ch 05, Ch 04 | — |
| Ch 05: Data Layer & Storage | NOT STARTED | 0 / 15-20 | — | Ch 00 ✅, Ch 06 | Ch 04 | — |
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
| IMP-003 | Ch 01 (D-006) | Ch 07 | Auth is host-app responsibility; userId prop only | Pending |
| IMP-004 | Ch 01 (D-008) | Ch 06 | Single config per instance; no persona routing | Pending |
| IMP-005 | Ch 02 (D-009) | Ch 00 | Monorepo structure defines folder conventions | **Propagated** |
| IMP-006 | Ch 02 (D-010) | Ch 00, Ch 03 | Package names define all import statements | Partially propagated (Ch 00 ✅, Ch 03 pending) |
| IMP-007 | Ch 02 (D-011) | Ch 03 | Headless + styled defines component API surface | Pending |
| IMP-008 | Ch 02 (D-013) | Ch 00 | ESM-only constrains import/export conventions | **Propagated** |
| IMP-009 | Ch 02 (D-016) | Ch 04 | Example API route is canonical backend reference | Pending |
| IMP-010 | Ch 02 (D-017) | Ch 00 | Root tsconfig.base.json is TS config source of truth | **Propagated** |
| IMP-011 | Ch 02 (D-020) | Ch 03 | Sub-path exports define headless vs. styled imports | Pending |
| IMP-012 | Ch 02 (D-021) | Ch 10 | CI pipeline steps and budget defined here | Pending |
| IMP-013 | Ch 00 (D-022) | Ch 05, Ch 06 | Domain-split types structure defines how each chapter imports its types | Pending |
| IMP-014 | Ch 00 (D-030) | Ch 04, Ch 05 | All failable ops must return Result<T, AppError> | Pending |
| IMP-015 | Ch 00 (D-034) | Ch 05, Ch 06 | KnowledgeEntry shape drives storage and config decisions | Pending |
| IMP-016 | Ch 00 (D-036) | Ch 03, Ch 06 | AIChatAssistantConfig is the top-level public API | Pending |
| IMP-017 | Ch 00 (D-037) | Ch 05 | StorageAdapter interface must be satisfied by Phase 1 IndexedDB impl | Pending |
| IMP-018 | Ch 00 (D-039) | Ch 04 | Backend must implement Vercel AI SDK v7 streaming contract (reinforced) | Pending |

---

## Recent Sessions Log

| # | Date | Chapter | Session Type | Decisions Resolved | Notes |
|---|------|---------|-------------|--------------------|---------|
| 1 | 2026-06-30 | — | Protocol Setup | 0 | Established spec protocol, repo created. |
| 2 | 2026-06-30 | Ch 01 | Scoping + Lock | 8 | D-001 to D-008. LOCKED. |
| 3 | 2026-06-30 | Ch 02 | Scoping (partial) | 8 | D-009 to D-016. Prematurely locked. |
| 4 | 2026-06-30 | Ch 02 | Reopen + Complete | 13 | Added D-017 to D-021. Re-locked properly at 13 decisions. |
| 5 | 2026-07-01 | Ch 00 | Scoping + Lock | 20 | D-022 to D-041. Full pass, locked in one session. |

---

## Quick-Reference: Next-Action Decision Tree

1. **Pending cross-chapter impacts?** → Run Propagation Session
2. **Any chapter in COMPLETE (not REVIEWED)?** → Run Review Session
3. **Any chapter in IN PROGRESS?** → Continue grilling
4. **Any chapter in REVIEWED?** → Lock it
5. **Any chapter CREATED with deps ≥ IN PROGRESS?** → Scope it
6. **Any chapter SCOPED with deps ≥ IN PROGRESS?** → Start grilling
7. **All chapters LOCKED?** → Spec book complete
8. **All blocked?** → Report blocker chain
