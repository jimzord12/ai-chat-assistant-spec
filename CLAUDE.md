# CLAUDE.md — Spec Book Agent Instructions

> **You are an AI coding agent (Claude Code / Opus).** This file is your entry point.
> This repository is a SPEC BOOK — it contains technical decisions and implementation contracts, NOT code. Your job here is to help fill, review, and lock chapters.

---

## START HERE — Cold-Start Protocol

When a session begins, before doing anything else:

1. Read `SPEC_STATE.md`
2. Extract: active chapter, stage, decisions resolved/total, open questions, last session summary, next recommended action, blocked items
3. Report to the user:
   ```
   Project: AI Chat Assistant Spec Book
   Active Chapter: [Chapter NN: Title] — [STAGE]
   Decisions: [X] / [Y] resolved
   Open Questions: [Z] remaining
   Last Session: [date] — [summary]
   NEXT ACTION: [recommendation]
   BLOCKED ON: [list or "None"]
   ```
4. Ask: "Ready to [recommended action]? Or review open questions first?"

---

## Next-Action Decision Tree

Evaluate top-to-bottom. First match wins. Do NOT improvise.

1. Pending cross-chapter impacts? → Run a Propagation Session
2. Any chapter COMPLETE but not REVIEWED? → Run a Review Session
3. Any chapter IN PROGRESS? → Continue grilling (next open question)
4. Any chapter REVIEWED? → Lock it, update SPEC_STATE.md, re-evaluate from step 1
5. Any chapter CREATED with deps ≥ IN PROGRESS? → Scope it (identify 15-30 questions)
6. Any chapter SCOPED with deps ≥ IN PROGRESS? → Start grilling (first session)
7. All chapters LOCKED? → Spec book complete. Ready for implementation.
8. All in-progress chapters blocked? → Report blocker chain, find root cause.

---

## Session Types

| Type | What to Do |
|------|-----------|
| Cold-Start | Read SPEC_STATE.md, report status, recommend next action |
| Grilling | Open active chapter, resolve next open question(s), fill decision sub-template |
| Review | Run self-review checklist (SPEC_PROTOCOL.md §2.2), verify cross-chapter consistency |
| Propagation | Read pending impacts, update affected chapters, mark as propagated |

---

## Chapter Lifecycle

```
CREATED → SCOPED → IN PROGRESS → COMPLETE → REVIEWED → LOCKED
```

- **CREATED:** Template exists, no content. Must identify 15-30 questions to advance.
- **SCOPED:** Questions ready. First grilling session moves it to IN PROGRESS.
- **IN PROGRESS:** Active grilling. All questions resolved + all sections filled → COMPLETE.
- **COMPLETE:** Self-review checklist passed → REVIEWED.
- **REVIEWED:** Cross-chapter consistency verified → LOCKED.
- **LOCKED:** Frozen. No changes without formal unlock (change request).

See `SPEC_PROTOCOL.md` for full lifecycle rules and exit criteria.

---

## Decision Sub-Template

When resolving a question, fill this in the chapter file:

```
### Decision [D-NNN]: [Title]
> Status: Resolved | Phase Impact: [1/2/3/All] | Cross-Chapter Links: [D-XXX in Ch YY]

#### Context
[Why this decision exists, what problem it solves]

#### Options Considered
| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| A | ... | ... | Chosen/Rejected |

#### Decision
[Directive: "We will use X because Y"]

#### Implementation Contract
[Exact TypeScript interface, file path, data shape]

#### Acceptance Criteria
- [ ] [Testable condition]

#### Rationale Notes
[Edge cases, gotchas]
```

---

## Always Do
- Read SPEC_STATE.md at the start of every session
- Update SPEC_STATE.md at the end of every session (last session summary, decisions count, open questions count, next recommended action)
- Log every session in the chapter's §7 Session History table
- Check the cross-chapter impact register after every decision — if it ripples, flag it in dependent chapters
- Respect the dependency graph — never start a chapter whose dependencies aren't at least IN PROGRESS
- Prefer finishing in-progress chapters over starting new ones
- Use exact file paths, TypeScript interfaces, and function signatures in implementation contracts — no prose where code belongs
- Make every acceptance criterion binary testable (pass/fail, not subjective)

## Ask Before Doing
- Unlocking a LOCKED chapter
- Starting a new chapter when another is IN PROGRESS (unless the IN PROGRESS chapter is blocked)
- Adding a new question to a chapter that exceeds the 30-question ceiling without justification
- Making a decision that contradicts a dependency chapter's locked decisions

## Never Do
- Never edit SPEC_STATE.md without also updating the Recent Sessions Log
- Never mark a chapter COMPLETE with remaining TODOs, TBDs, or placeholders
- Never advance a chapter stage without meeting ALL exit criteria for the current stage
- Never skip the self-review checklist before marking COMPLETE → REVIEWED
- Never ignore a pending cross-chapter impact — they must be propagated before locking
- Never use `any` in implementation contracts
- Never leave an acceptance criterion untestable (no "works correctly" — specify what "correct" means)

---

## File Reference

| File | Purpose |
|------|--------|
| `SPEC_STATE.md` | Current status — read first after this file |
| `SPEC_PROTOCOL.md` | Full lifecycle rules, session protocols, checklists |
| `MASTER_INDEX.md` | Project map, cold-start protocol, file structure |
| `chapter_template.md` | Template for creating new chapter files |
| `chapters/` | All chapter files (12 total) |
| `chapter_dependency_graph.png` | Visual dependency map |
| `chapter_lifecycle.png` | Visual lifecycle stages |

---

## Technology Context

This spec covers an AI Chat Assistant widget with:
- **Frontend:** Next.js 16, assistant-ui (headless), shadcn/ui, Tailwind v4, TypeScript 5-6
- **Backend:** Express v5 (TypeScript), Vercel AI SDK
- **Model:** Gemini 3.1 Flash Lite
- **Storage:** IndexedDB (Phase 1) → OPFS SQLite (Phase 2) → PostgreSQL + pgvector (Phase 3)
- **Config:** Centralized `bot.config.ts` for model, provider, system prompt, allowed topics
- **Language:** Greek-speaking users (primary), multilingual support (future)

The spec is phased: Phase 1 (basic chat), Phase 2 (MVP for production), Phase 3 (decent product with RAG).
