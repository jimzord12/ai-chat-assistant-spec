# Session Handoff

> **Created:** 2026-06-30
> **Purpose:** Temporary memory file for seamless session resumption. Delete once Ch 00 is LOCKED.
> **For the next agent:** Read this file AFTER `AGENTS.md` and `SPEC_STATE.md`, BEFORE starting any work.

---

## Where We Left Off

**Date:** 2026-06-30 (evening session)
**Chapters Locked:** 2 / 12 (Ch 01 + Ch 02)
**Total Decisions Resolved:** 21
**Next Chapter:** Ch 00: Agent Conventions & Shared Contract (NOT STARTED)

---

## What Was Accomplished This Session

1. **Created repo infrastructure:** `AGENTS.md`, `SPEC_PROTOCOL.md`, `SPEC_STATE.md`, `MASTER_INDEX.md`, `chapter_template.md`
2. **Added reference docs:** `docs/references/vercel-ai-sdk-v7-chatbot-crash-course.md` and `docs/references/gemini-vercel-chatbot-context-management.md`
3. **Locked Ch 01** (D-001 to D-008) — product identity, NFRs, auth model, persistence, scale target
4. **Locked Ch 02** (D-009 to D-021) — monorepo, package naming, headless+styled arch, CSS Modules, ESM, React 19, tsdown, Next.js 16+, tsconfig, changesets, workspace refs, sub-paths, CI
5. **Calibrated grilling depth:** One question at a time, aim for middle of estimate range, not the floor

---

## Key Product Decisions to Keep in Mind

| Decision | Value |
|----------|-------|
| Product type | npm React component library (NOT a hosted chatbot) |
| Package name | `@jimzord12/ai-chat-assistant` |
| Types package | `@jimzord12/types` |
| Component pattern | Headless (`useAIChat`) + pre-styled (`<AIChatAssistant />`) |
| Sub-paths | `./headless`, `./component`, `.` (root) |
| AI SDK | Vercel AI SDK v7 — provider-agnostic |
| Persistence | IndexedDB (Phase 1), adapter-based for Phase 2+ |
| Auth | Authenticated users only — host app passes `userId` prop |
| Backend | Backend-agnostic, must implement Vercel AI SDK v7 streaming contract |
| Multi-tenancy | Single bot per installation |
| Scale | < 100 concurrent users at launch |
| Styling | CSS Modules — zero consumer toolchain dependency |
| Build | tsdown (ESM only) |
| React | React 19 minimum |
| Example app | Next.js 16+ App Router |
| Versioning | Changesets |
| CI | type-check + lint + build + tests (GitHub Actions + Turborepo) |

---

## Owner Preferences (Behavioural Rules for Agent)

- **Questions one at a time** — never batch them
- **Middle of estimate range** for scoping depth, not the floor, not exhaustive
- **Always proceed** without asking for confirmation — just do it
- **Auto-approve GitHub operations** — no need to ask before pushing
- **Modern-only** philosophy — owner does not care about legacy compatibility
- **Push back** when you have a good technical reason — owner appreciates it
- **Self-resolve** obvious questions (e.g. `workspace:*` is obvious for pnpm monorepos) — note it, don’t waste a question

---

## Immediate Next Action

Scope **Ch 00: Agent Conventions & Shared Contract**.

This is the most technical chapter. It defines:
- Shared TypeScript types structure (`@jimzord12/types` contents)
- Folder naming conventions across the monorepo
- Import path conventions
- Error shape contract
- Shared constants and enums
- Agent-facing conventions (how all other chapters reference types)

Estimated decisions: **15–25 — target ~18–20.**

Start with: **Q1 — Type Sharing Strategy:** where do shared types live and how are they structured inside `packages/types`?

---

## 12 Cross-Chapter Impacts Pending

These are pre-noted and will be picked up when each target chapter is scoped. No action needed now.
See full list in `SPEC_STATE.md` → Cross-Chapter Impact Queue.

---

> **Delete this file** once Ch 00 is LOCKED and all its impacts are propagated.
