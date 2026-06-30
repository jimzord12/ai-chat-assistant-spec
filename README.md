# AI Chat Assistant — Spec Book

> **Implementation-ready specification for AI agent development teams (Claude Code / Opus).**

This repository contains the complete technical specification for a modern AI Chat Assistant widget built with Next.js 16, Express v5, assistant-ui, Vercel AI SDK, and Gemini 3.1 Flash Lite. The spec is structured so that separate frontend and backend AI agent teams can implement without human guidance.

---

## Quick Start

### If you're a human resuming work:

1. Read [`MASTER_INDEX.md`](MASTER_INDEX.md) — what exists and where
2. Read [`SPEC_STATE.md`](SPEC_STATE.md) — current status and next action
3. Follow the Next-Action Decision Tree in `SPEC_STATE.md`

### If you're Claude Code:

Read [`CLAUDE.md`](CLAUDE.md) first. It will tell you exactly what to do.

---

## Repository Structure

```
spec-book/
├── README.md                  ← You are here
├── CLAUDE.md                  ← Claude Code entry point
├── MASTER_INDEX.md            ← Project map + cold-start protocol
├── SPEC_STATE.md              ← Current status (read this to resume)
├── SPEC_PROTOCOL.md           ← Chapter lifecycle, session rules, decision tree
├── chapter_template.md        ← Template for creating new chapters
├── chapter_dependency_graph.png
├── chapter_lifecycle.png
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
└── .gitignore
```

---

## The Three-File Rule

You only ever need to read two files to resume work at any time:

| Read | File | Why |
|------|------|-----|
| 1st | `MASTER_INDEX.md` | What exists, where everything is, how to resume |
| 2nd | `SPEC_STATE.md` | Where are we right now, what's next |
| (ref) | `SPEC_PROTOCOL.md` | How chapter lifecycle and sessions work (read when needed) |

---

## Technology Stack

| Layer | Technology |
|-------|----------|
| Frontend | Next.js 16, assistant-ui, shadcn/ui, Tailwind v4, TypeScript 5-6 |
| Backend | Express v5 (TypeScript), Vercel AI SDK |
| Model | Gemini 3.1 Flash Lite |
| Storage (Phase 1) | IndexedDB |
| Storage (Phase 2) | OPFS + SQLite (wa-sqlite) |
| Storage (Phase 3) | PostgreSQL + pgvector |

---

## Development Phases

| Phase | Goal |
|-------|------|
| Phase 1 | Basic chat works (IndexedDB, system prompt, input classifier guardrail) |
| Phase 2 | MVP for Production (OPFS SQLite, thread sync, auth, rate limiting) |
| Phase 3 | Decent Product (output guardrail, RAG with pgvector + LangChain, fine-tuned classifier) |

---

## Status

- **Chapters Locked:** 0 / 12
- **Decisions Resolved:** 0 / ~150-200
- **Current Stage:** Initialization — ready to scope Chapter 01
