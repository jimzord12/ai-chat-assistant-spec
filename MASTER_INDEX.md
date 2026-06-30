# SPEC BOOK — MASTER INDEX

> **This file is your entry point.** If you remember nothing, open this first.

---

## The Three Files You Need

| File | Purpose | When to Read |
|------|---------|-------------|
| `MASTER_INDEX.md` (this file) | What exists, where everything is, how to resume | Always first |
| `SPEC_STATE.md` | Current status of every chapter, last session, next action | Right after this file |
| `SPEC_PROTOCOL.md` | The rules: chapter lifecycle, session types, decision tree | When you need to know HOW to do something |

---

## File Structure

```
spec-book/
├── MASTER_INDEX.md          ← You are here. Entry point.
├── SPEC_STATE.md             ← Project status. Read this second.
├── SPEC_PROTOCOL.md          ← The rules. Read when needed.
├── chapter_template.md       ← Template for new chapters.
├── chapters/
│   ├── ch-01-overview-and-nfrs.md
│   ├── ch-02-architecture-and-topology.md
│   ├── ch-00-agent-conventions-and-shared-contract.md
│   ├── ch-06-config-and-extensibility.md
│   ├── ch-05-data-layer-and-storage.md
│   ├── ch-04-backend-api-and-streaming.md
│   ├── ch-07-auth-security-and-rate-limiting.md
│   ├── ch-08-guardrails-and-moderation.md
│   ├── ch-03-frontend-widget-and-ui.md
│   ├── ch-09-testing-strategy.md
│   ├── ch-10-deployment-and-observability.md
│   └── ch-11-phased-delivery-and-acceptance-criteria.md
├── chapter_dependency_graph.png
├── chapter_dependency_and_write_order.md
└── chapter_lifecycle.png
```

---

## Write Order (Dependency-Resolved)

| # | Layer | Chapter | Depends On |
|---|-------|---------|------------|
| 1 | Foundation | Ch 01: Overview & NFRs | — |
| 2 | Foundation | Ch 02: Architecture & Topology | Ch 01 |
| 3 | Foundation | Ch 00: Agent Conventions & Shared Contract | Ch 02 |
| 4 | Core | Ch 06: Config & Extensibility | Ch 00 |
| 5 | Core | Ch 05: Data Layer & Storage | Ch 00, Ch 06 |
| 6 | Core | Ch 04: Backend: API & Streaming | Ch 06, Ch 05 |
| 7 | Integration | Ch 07: Auth, Security & Rate Limiting | Ch 04 |
| 8 | Integration | Ch 08: Guardrails & Moderation | Ch 04 |
| 9 | Integration | Ch 03: Frontend: Widget & UI | Ch 04, Ch 07 |
| 10 | Delivery | Ch 09: Testing Strategy | Ch 03, Ch 08 |
| 11 | Delivery | Ch 10: Deployment & Observability | Ch 07, Ch 04 |
| 12 | Delivery | Ch 11: Phased Delivery & Acceptance Criteria | Ch 09, Ch 10 |

Chapters 7+8 can run in parallel. Chapters 9+10 can run in parallel.

---

## The Cold-Start Protocol (Resume From Zero)

When you sit down, remember nothing, and open Claude:

### Step 1: Read State
```
Read SPEC_STATE.md
```

### Step 2: Claude Reports
Claude should output:
```
Project: AI Chat Assistant Spec Book
Active Chapter: [Chapter NN: Title] — [STAGE]
Decisions: [X] / [Y] resolved
Open Questions: [Z] remaining
Last Session: [date] — [summary]
NEXT ACTION: [specific recommendation]
BLOCKED ON: [list or "None"]
```

### Step 3: Claude Asks
```
"Ready to [recommended action]? Or review open questions first?"
```

### Step 4: You Decide
- Continue the recommended action, OR
- Review open questions, OR
- Switch to a different chapter (allowed but discouraged)

---

## Chapter Lifecycle Stages

```
CREATED → SCOPED → IN PROGRESS → COMPLETE → REVIEWED → LOCKED
```

| Stage | Meaning | Key Rule |
|-------|---------|----------|
| CREATED | Template exists, no content | Must identify 15-30 questions before advancing |
| SCOPED | Questions identified, deps verified | Ready for first grilling session |
| IN PROGRESS | Active grilling, decisions being made | Sessions happening |
| COMPLETE | All questions resolved, all sections filled | Ready for self-review |
| REVIEWED | Self-review passed, cross-chapter verified | Ready to lock |
| LOCKED | Frozen, implementation-ready | No changes without formal unlock |

---

## Session Types

| Type | Purpose | Typical Duration | Trigger |
|------|---------|------------------|---------|
| Cold-Start | Resume, determine next action | 5 min | Every session start |
| Grilling | Active decision-making | 30-60 min | Chapter in IN PROGRESS |
| Review | Self-review or cross-chapter review | 20-40 min | Chapter in COMPLETE |
| Propagation | Push cross-chapter impacts to dependents | 15-30 min | Pending impacts in queue |

---

## The 15 High-Impact Decisions (Quick Reference)

| ID | Decision | Chapter | Status |
|----|----------|---------|--------|
| D1 | Streaming protocol (AI SDK vs SSE vs custom) | Ch 04 | Open |
| D2 | assistant-ui runtime selection | Ch 03 | Open |
| D3 | Type sharing strategy (shared pkg vs codegen vs path aliases) | Ch 00 | Open |
| D4 | Storage adapter interface contract | Ch 05 | Open |
| D5 | bot.config.ts schema and ownership | Ch 06 | Open |
| D6 | Error contract (shape, codes, mid-stream handling) | Ch 04 | Open |
| D7 | Authentication & token flow | Ch 07 | Open |
| D8 | CORS & origin policy for widget embedding | Ch 07 | Open |
| D9 | Thread sync conflict resolution strategy | Ch 05 | Open |
| D10 | Rate limiting architecture & response contract | Ch 07 | Open |
| D11 | Guardrail pipeline ordering & rejection contract | Ch 08 | Open |
| D12 | State management pattern (runtime vs Zustand vs Context) | Ch 03 | Open |
| D13 | Environment configuration structure | Ch 10 | Open |
| D14 | Widget packaging & host app contract | Ch 02/03 | Open |
| D15 | Testing boundaries & mocking strategy | Ch 09 | Open |

---

## Design Patterns Registry

| Pattern | Where Applied | Chapter |
|---------|--------------|--------|
| Adapter | Storage, Runtime, Model provider | Ch 05, 03, 06 |
| Strategy | Guardrails, Rate limiting | Ch 08, 07 |
| Factory | Runtime creation from config | Ch 06 |
| Dependency Injection | Express routes, config consumption | Ch 04, 06 |
| Facade | Widget public API | Ch 02/03 |
| Composition over Inheritance | assistant-ui components | Ch 03 |
| Convention over Configuration | Folder structure, naming | Ch 00 |
| Interface Segregation | Shared types | Ch 00 |
| Fail Fast | Config validation, guardrails | Ch 06, 08 |
| Single Source of Truth | bot.config.ts, shared types | Ch 00, 06 |

---

## Rules for Multi-Week Continuity

1. **SPEC_STATE.md is sacred.** Update it at the end of every session, no exceptions.
2. **Never start a session without reading SPEC_STATE.md.** Even if you think you remember.
3. **Finish what you start.** Prefer finishing an in-progress chapter over starting a new one.
4. **Propagate impacts immediately.** Flag cross-chapter impacts in the same session.
5. **Lock aggressively.** Once a chapter passes review, lock it.
6. **Respect the dependency graph.** Don't start a chapter whose dependencies aren't at least IN PROGRESS.
7. **Log everything.** Session history is how you reconstruct context weeks later.
8. **The Next-Action Decision Tree is law.** Follow it, don't improvise.
