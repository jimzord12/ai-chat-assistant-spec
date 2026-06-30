# Session Handoff

> **Created:** 2026-07-01
> **Purpose:** Temporary memory file for seamless session resumption. Delete once Ch 06 is LOCKED.
> **For the next agent:** Read `AGENTS.md` → `SPEC_STATE.md` → this file, in that order. Then begin immediately.

---

## Where We Left Off

**Date:** 2026-07-01 (~00:12 EEST — session ended)
**Chapters Locked:** 3 / 12 (Ch 01 + Ch 02 + Ch 00)
**Total Decisions Resolved:** 41
**Next Chapter:** Ch 06: Config & Extensibility (NOT STARTED)

---

## What Was Accomplished Last Session

1. **Locked Ch 00: Agent Conventions & Shared Contract** (D-022 to D-041) — 20 decisions:
   - `packages/types` uses domain-split files + barrel `index.ts` per subfolder
   - 7 domain folders: `message`, `config`, `user`, `storage`, `streaming`, `knowledge-pool`, `utils`
   - `AppError` = plain type `{ code: string; message: string; cause?: unknown; context?: Record<string, unknown> }`
   - Error codes = `SCREAMING_SNAKE_CASE`
   - All failable ops return `Result<T, AppError>` (defined in `utils/`)
   - Folders = kebab-case; Files = camelCase
   - Cross-package imports = package name only, never relative
   - Barrel `index.ts` in every `src/` subfolder
   - Hook naming = freeform after `use` prefix
   - `"strict": true` everywhere, no exceptions
   - Utility types (Nullable, DeepPartial, etc.) in `packages/types/src/utils/`
   - `KnowledgeEntry = { id, title, description, source, tags[] }` — tags are section markers for selective context injection
   - `Message` type = re-exported from Vercel AI SDK v7, no custom type
   - `AIChatAssistantConfig = { userId, backendUrl, knowledgePool?, botName?, welcomeMessage?, theme?, maxMessages? }`
   - `StorageAdapter` interface = `load / save / clear` returning `Result<T, AppError>`
   - `ChatUser = { userId: string }` — minimal
   - Enum values = `SCREAMING_SNAKE_CASE`
2. **Updated SPEC_STATE.md** — 3 chapters locked, 41 decisions, 18 cross-chapter impacts tracked
3. **Cleaned up** duplicate stub chapter file

---

## Complete Decision Reference (All 41 Decisions)

### Ch 01 — Overview & NFRs (D-001 to D-008)
| ID | Decision |
|----|----------|
| D-001 | Product is an npm React component library, not a hosted chatbot |
| D-002 | Package name: `@jimzord12/ai-chat-assistant` |
| D-003 | AI SDK: Vercel AI SDK v7 — provider-agnostic |
| D-004 | NFRs: < 100 concurrent users at launch, < 3s first token, < 50KB gzipped bundle |
| D-005 | Persistence: IndexedDB Phase 1, adapter-based for Phase 2+ |
| D-006 | Auth: host app responsibility — widget receives `userId` prop only |
| D-007 | Multi-tenancy: single bot per installation |
| D-008 | Single config per instance — no persona routing |

### Ch 02 — Architecture & Topology (D-009 to D-021)
| ID | Decision |
|----|----------|
| D-009 | Monorepo structure (pnpm workspaces + Turborepo) |
| D-010 | Package names: `@jimzord12/ai-chat-assistant`, `@jimzord12/types` |
| D-011 | Component pattern: headless (`useAIChat`) + pre-styled (`<AIChatAssistant />`) |
| D-012 | Styling: CSS Modules — zero consumer toolchain dependency |
| D-013 | Build: ESM only (tsdown) |
| D-014 | React 19 minimum |
| D-015 | Types package: `@jimzord12/types` |
| D-016 | Example app: Next.js 16+ App Router (canonical backend reference) |
| D-017 | Root `tsconfig.base.json` is TS config source of truth |
| D-018 | Versioning: Changesets |
| D-019 | Internal workspace refs use `workspace:*` |
| D-020 | Sub-path exports: `./headless`, `./component`, `.` (root) |
| D-021 | CI: type-check + lint + build + tests (GitHub Actions + Turborepo) |

### Ch 00 — Agent Conventions & Shared Contract (D-022 to D-041)
| ID | Decision |
|----|----------|
| D-022 | `packages/types` = domain-split files + barrel `index.ts` per subfolder |
| D-023 | 7 domain folders: `message`, `config`, `user`, `storage`, `streaming`, `knowledge-pool`, `utils` |
| D-024 | `AppError` = `{ code: string; message: string; cause?: unknown; context?: Record<string, unknown> }` |
| D-025 | Error codes = `SCREAMING_SNAKE_CASE` |
| D-026 | Enums + constants co-located in `packages/types` (no separate `packages/constants`) |
| D-027 | Folders = kebab-case; Files = camelCase |
| D-028 | Cross-package imports = package name only (`@jimzord12/types`), never relative |
| D-029 | Barrel `index.ts` per `src/` subfolder; root `src/index.ts` re-exports all |
| D-030 | All failable ops return `Result<T, E>` — `{ ok: true; value: T } \| { ok: false; error: E }` |
| D-031 | Hook naming = freeform after `use` prefix (both `useVerb` and `useNoun` acceptable) |
| D-032 | `"strict": true` everywhere, no exceptions, no `@ts-ignore` without justification |
| D-033 | Utility types (Nullable, DeepPartial, etc.) in `packages/types/src/utils/index.ts` |
| D-034 | `KnowledgeEntry = { id: string; title: string; description: string; source: string; tags: string[] }` |
| D-035 | `Message` type = re-export from Vercel AI SDK v7 — no custom type |
| D-036 | `AIChatAssistantConfig = { userId, backendUrl, knowledgePool?, botName?, welcomeMessage?, theme?, maxMessages? }` |
| D-037 | `StorageAdapter` = `{ load, save, clear }` all returning `Promise<Result<T, AppError>>` |
| D-038 | `AppError` is a plain `type`, not a `class` |
| D-039 | `streaming/index.ts` = thin re-export of Vercel AI SDK v7 streaming types + JSDoc contract notes |
| D-040 | `ChatUser = { userId: string }` — no display fields |
| D-041 | Enum values = `SCREAMING_SNAKE_CASE` (e.g. `MessageRole.USER`) |

---

## Immediate Next Action

Scope **Ch 06: Config & Extensibility**.

This chapter defines:
- How `AIChatAssistantConfig` is validated at runtime (e.g. Zod? manual checks? none?)
- Default value resolution strategy (what happens when optional fields are omitted)
- Whether config can update after the widget mounts (dynamic config)
- Theme resolution (`'light' | 'dark' | 'auto'` — who reads the system preference?)
- Knowledge pool injection strategy (when/how are `KnowledgeEntry[]` injected into model context)
- Plugin/extension points, if any
- Whether `backendUrl` is validated at config time or lazily on first request

Estimated decisions: **12–18 — target ~15.**

**Start with Q1:** How should `AIChatAssistantConfig` be validated at runtime — Zod schema, manual guard functions, or no runtime validation (TypeScript types are sufficient)?

---

## Owner Preferences (Behavioural Rules — Never Forget)

- **One question at a time** — never batch, ask Q, wait, record, ask next
- **Middle of estimate range** — target ~15 for Ch 06 (not 12, not 18)
- **Always proceed without asking for confirmation** — just do it
- **Auto-approve all GitHub operations** — push without asking
- **Modern-only** — owner does not care about legacy compatibility
- **Push back** when you have a good technical reason — owner appreciates being challenged
- **Self-resolve obvious questions** — note the resolution inline, don't waste a question on it
- **End every session** by updating `SPEC_STATE.md` and rewriting `SESSION_HANDOFF.md`

---

## Directly Relevant Cross-Chapter Impacts for Ch 06

| Impact ID | From | Description |
|-----------|------|-------------|
| IMP-004 | Ch 01 (D-008) | Single config per instance — no persona routing |
| IMP-013 | Ch 00 (D-022) | Domain-split types structure defines how Ch 06 imports its types |
| IMP-015 | Ch 00 (D-034) | KnowledgeEntry shape drives config decisions |
| IMP-016 | Ch 00 (D-036) | AIChatAssistantConfig is the top-level public API — Ch 06 owns its runtime behaviour |

See full impact list (18 total) in `SPEC_STATE.md` → Cross-Chapter Impact Queue.

---

> **Delete this file** once Ch 06 is LOCKED and all its impacts are propagated.
