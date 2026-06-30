# Session Handoff

> **Created:** 2026-06-30
> **Purpose:** Temporary memory file for seamless session resumption. Delete once Ch 06 is LOCKED.
> **For the next agent:** Read this file AFTER `AGENTS.md` and `SPEC_STATE.md`, BEFORE starting any work.

---

## Where We Left Off

**Date:** 2026-06-30 (late evening session)
**Chapters Locked:** 3 / 12 (Ch 01 + Ch 02 + Ch 00)
**Total Decisions Resolved:** 41
**Next Chapter:** Ch 06: Config & Extensibility (NOT STARTED)

---

## What Was Accomplished This Session

1. **Locked Ch 00** (D-022 to D-041) — 20 decisions covering:
   - Domain-split type structure for `packages/types`
   - All 7 domain files: `message`, `config`, `user`, `storage`, `streaming`, `knowledge-pool`, `utils`
   - `AppError` plain type with rich shape + `SCREAMING_SNAKE_CASE` error codes
   - `Result<T, E>` pattern for all failable operations
   - kebab-case folders + camelCase files naming convention
   - Package-name-only imports (no relative cross-package)
   - Barrel `index.ts` per subfolder
   - Hook naming freeform after `use` prefix
   - Full strict TypeScript, no exceptions
   - `utils.ts` for shared generic utility types
   - `KnowledgeEntry` shape with `tags` for selective context injection
   - Message types re-exported from Vercel AI SDK v7 (no custom Message)
   - `AIChatAssistantConfig` shape (userId, backendUrl, knowledgePool?, botName?, welcomeMessage?, theme?, maxMessages?)
   - `StorageAdapter` interface (load, save, clear)
   - `ChatUser` minimal shape `{ userId: string }`
   - Enum values in `SCREAMING_SNAKE_CASE`
2. **Added 6 new cross-chapter impacts** (IMP-013 to IMP-018) to `SPEC_STATE.md`

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
| Error shape | `{ code: string; message: string; cause?: unknown; context?: Record<string, unknown> }` |
| Error codes | `SCREAMING_SNAKE_CASE` |
| Error type | Plain `type AppError` (not a class) |
| Failable ops | Always return `Result<T, AppError>` |
| Folder naming | kebab-case |
| File naming | camelCase |
| Cross-package imports | Package name only (`@jimzord12/types`) |
| Barrel pattern | Barrel `index.ts` per subfolder |
| Strict TS | `"strict": true` everywhere, no exceptions |
| Enum values | `SCREAMING_SNAKE_CASE` |
| Message type | Re-exported from Vercel AI SDK v7 — no custom type |
| KnowledgeEntry | `{ id, title, description, source, tags[] }` |
| AIChatAssistantConfig | `{ userId, backendUrl, knowledgePool?, botName?, welcomeMessage?, theme?, maxMessages? }` |
| StorageAdapter | `load / save / clear` returning `Result<T, AppError>` |

---

## Owner Preferences (Behavioural Rules for Agent)

- **Questions one at a time** — never batch them
- **Middle of estimate range** for scoping depth, not the floor, not exhaustive
- **Always proceed** without asking for confirmation — just do it
- **Auto-approve GitHub operations** — no need to ask before pushing
- **Modern-only** philosophy — owner does not care about legacy compatibility
- **Push back** when you have a good technical reason — owner appreciates it
- **Self-resolve** obvious questions — note it, don't waste a question

---

## Immediate Next Action

Scope **Ch 06: Config & Extensibility**.

This chapter defines:
- How `AIChatAssistantConfig` is validated and applied at runtime
- Plugin/extension points for the widget (if any)
- Theme config resolution (`'light' | 'dark' | 'auto'`)
- Knowledge pool injection strategy
- Default value handling
- Config change handling (can config update after mount?)

Estimated decisions: **12–18 — target ~15.**

---

> **Delete this file** once Ch 06 is LOCKED and all its impacts are propagated.
