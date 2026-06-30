# Chapter 00: Agent Conventions & Shared Contract

> **Status:** LOCKED
> **Primary Consumer:** Both Teams
> **Dependencies:** Ch 01 ✅, Ch 02 ✅
> **Dependents:** Ch 06, Ch 05
> **Sessions Contributed:** 1
> **Questions Resolved:** 20 / 20

---

## 1. Context & Scope

### 1.1 What This Chapter Covers
This chapter defines the shared TypeScript type system, folder/file naming conventions, import path rules, error shape contract, and all agent-facing conventions that every other chapter must follow. It is the source of truth for cross-package contracts.

### 1.2 What This Chapter Does NOT Cover
- Component API surface (Ch 03)
- Storage adapter implementation details (Ch 05)
- Backend streaming implementation (Ch 04)
- CI/build tooling (Ch 02)

### 1.3 Key Stakeholders
- Both frontend and backend agent teams must read and follow this chapter before implementing anything.
- Owner reviews before locking.

---

## 2. Decision Log

### Decision D-022: Type Sharing Strategy
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** IMP-005 (Ch 02 → Ch 00), IMP-008 (Ch 02 → Ch 00), IMP-010 (Ch 02 → Ch 00)

#### Context
Shared TypeScript types must be accessible by all packages without duplication or circular deps.

#### Options Considered
| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| Flat single index.ts | Simple | Gets messy beyond ~50 types | Rejected |
| Domain-split + barrel index.ts | Organised, scalable, easy to navigate | Slightly more files | **Chosen** |
| Sub-path exports per domain | Maximum granularity | Over-engineering for this scale | Rejected |

#### Decision
We will use domain-split files inside `packages/types/src/`, each with their own `index.ts` barrel, and a root `src/index.ts` re-exporting everything.

#### Implementation Contract
```
packages/types/
  src/
    message/
      index.ts
    config/
      index.ts
    user/
      index.ts
    storage/
      index.ts
    streaming/
      index.ts
    knowledge-pool/
      index.ts
    utils/
      index.ts
    index.ts   ← root barrel, re-exports all domains
  package.json
  tsconfig.json
```

#### Acceptance Criteria
- [ ] Each domain folder has its own `index.ts`
- [ ] Root `src/index.ts` re-exports all domain barrels
- [ ] No circular imports between domain files

---

### Decision D-023: Domain Files in `packages/types`
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 05, Ch 06, Ch 03, Ch 04

#### Context
Define which domain files exist inside `packages/types/src/`.

#### Decision
The following domain folders exist:
- `message/` — chat message types (re-exported from Vercel AI SDK v7)
- `config/` — widget configuration and props
- `user/` — userId and auth-related types
- `storage/` — persistence adapter interface
- `streaming/` — streaming response contracts (re-exported from Vercel AI SDK v7)
- `knowledge-pool/` — knowledge entry types for selective context injection
- `utils/` — shared generic utility types

#### Acceptance Criteria
- [ ] All 7 domain folders exist in `packages/types/src/`
- [ ] Each has a barrel `index.ts`

---

### Decision D-024: Error Shape Contract
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 04, Ch 05, Ch 07

#### Context
All failable operations across the library must surface errors in a consistent, typed shape.

#### Options Considered
| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| Simple `{ code, message }` | Minimal | Loses debugging context | Rejected |
| Rich `{ code, message, cause?, context? }` | Full context, debuggable | Slightly more verbose | **Chosen** |

#### Decision
We will use the rich error shape.

#### Implementation Contract
```typescript
// packages/types/src/utils/index.ts
export type AppError = {
  code: string;
  message: string;
  cause?: unknown;
  context?: Record<string, unknown>;
};
```

#### Acceptance Criteria
- [ ] `AppError` is exported from `@jimzord12/types`
- [ ] All packages import `AppError` from `@jimzord12/types`, never redefine it

---

### Decision D-025: Error Code Format
> **Status:** Resolved
> **Phase Impact:** All

#### Context
Error `code` values must follow a consistent casing convention.

#### Decision
Error codes use `SCREAMING_SNAKE_CASE` (e.g. `STREAM_FAILED`, `STORAGE_WRITE_ERROR`, `INVALID_CONFIG`).

#### Acceptance Criteria
- [ ] All `AppError` instances in the codebase use `SCREAMING_SNAKE_CASE` codes
- [ ] A shared enum `ErrorCode` with known codes lives in `packages/types/src/utils/`

---

### Decision D-026: Enums & Constants Location
> **Status:** Resolved
> **Phase Impact:** All

#### Context
Shared enums (e.g. `MessageRole`) and constants (e.g. default config values) need a canonical home.

#### Options Considered
| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| Co-located in `packages/types` | One import, simpler | Mixes runtime values with types | **Chosen** |
| Separate `packages/constants` | Clean boundary | Extra package overhead | Rejected |

#### Decision
Enums and constants live co-located in `packages/types` alongside their related type files.

#### Acceptance Criteria
- [ ] No standalone `packages/constants` package exists
- [ ] Enums are exported from the relevant domain barrel in `packages/types`

---

### Decision D-027: Folder & File Naming Convention
> **Status:** Resolved
> **Phase Impact:** All

#### Context
Consistent naming across the monorepo prevents confusion and merge conflicts.

#### Options Considered
| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| kebab-case everywhere | Most common in JS ecosystem | camelCase hooks look odd as folders | Rejected |
| camelCase everywhere | Consistent with JS identifiers | Unusual for folders | Rejected |
| kebab folders + camelCase files | Best of both worlds | Two rules to remember | **Chosen** |

#### Decision
- **Folders:** `kebab-case` (e.g. `knowledge-pool/`, `chat-message/`)
- **Files:** `camelCase` (e.g. `knowledgePool.ts`, `chatMessage.ts`, `useAIChat.ts`)

#### Acceptance Criteria
- [ ] All folders in `packages/` and `apps/` use kebab-case
- [ ] All `.ts` / `.tsx` files use camelCase
- [ ] ESLint or a naming lint rule enforces this

---

### Decision D-028: Import Path Convention
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** IMP-006 (Ch 02 → Ch 00)

#### Context
Cross-package imports must be deterministic and boundary-safe.

#### Options Considered
| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| Package name only | Clean, enforces boundaries, refactor-safe | Requires proper workspace resolution | **Chosen** |
| Relative cross-package allowed | Flexible | Creates tight coupling, breaks on restructure | Rejected |

#### Decision
All cross-package imports must use the package name (e.g. `import type { AppError } from '@jimzord12/types'`). Relative imports are only allowed within the same package.

#### Acceptance Criteria
- [ ] No `../../` imports crossing package boundaries anywhere in the codebase
- [ ] ESLint `import/no-relative-packages` rule (or equivalent) enforced in CI

---

### Decision D-029: Barrel Pattern Within a Package
> **Status:** Resolved
> **Phase Impact:** All

#### Context
Defines the internal export structure convention for all packages.

#### Decision
Each subfolder inside `src/` exposes a barrel `index.ts`. The root `src/index.ts` re-exports from all subfolder barrels. This is the canonical export surface.

#### Acceptance Criteria
- [ ] Every `src/` subfolder has an `index.ts`
- [ ] Root `src/index.ts` re-exports from all subfolders
- [ ] No deep imports (e.g. `@jimzord12/types/src/config/configTypes`) used anywhere

---

### Decision D-030: Result Type Pattern
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 04, Ch 05

#### Context
Failable operations must surface errors explicitly rather than relying on thrown exceptions.

#### Options Considered
| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| `Result<T, E>` type | Explicit, type-safe, impossible to ignore errors | Slightly more verbose call sites | **Chosen** |
| throw/catch | Familiar, less boilerplate | Silent failure paths, hard to type | Rejected |

#### Decision
All operations that can fail return `Result<T, E>`. `Result` lives in `packages/types/src/utils/`.

#### Implementation Contract
```typescript
// packages/types/src/utils/index.ts
export type Result<T, E = AppError> =
  | { ok: true; value: T }
  | { ok: false; error: E };
```

#### Acceptance Criteria
- [ ] `Result<T, E>` is exported from `@jimzord12/types`
- [ ] All storage adapter methods return `Promise<Result<T, AppError>>`
- [ ] All streaming error paths return `Result<T, AppError>`

---

### Decision D-031: Hook Naming Convention
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 03

#### Context
React hooks need a consistent naming pattern for discoverability.

#### Decision
Hook naming is freeform after the `use` prefix. Both `useVerb` (e.g. `useAIChat`) and `useNoun` (e.g. `useMessages`) are acceptable.

#### Acceptance Criteria
- [ ] All hooks start with `use` (React requirement)
- [ ] Hook file names match the hook name in camelCase (e.g. `useAIChat.ts`)

---

### Decision D-032: TypeScript Strict Mode
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** IMP-010 (Ch 02 → Ch 00)

#### Context
Strict mode prevents entire classes of runtime bugs at compile time.

#### Decision
`"strict": true` is set in `tsconfig.base.json` and inherited by all packages. No flags are relaxed. No exceptions.

#### Acceptance Criteria
- [ ] `tsconfig.base.json` has `"strict": true`
- [ ] No `// @ts-ignore` or `// @ts-nocheck` in the codebase without a documented justification comment
- [ ] CI type-check step fails on any strict violation

---

### Decision D-033: Utility Types Location
> **Status:** Resolved
> **Phase Impact:** All

#### Context
Generic utility types (e.g. `Nullable<T>`, `DeepPartial<T>`) need a canonical home to prevent duplication.

#### Decision
All shared utility types live in `packages/types/src/utils/index.ts` and are exported from the root barrel. Individual packages do not redefine shared utilities.

#### Acceptance Criteria
- [ ] `Nullable<T>`, `Result<T, E>`, `AppError`, `ErrorCode` all exported from `@jimzord12/types`
- [ ] No duplicate utility type definitions in any other package

---

### Decision D-034: KnowledgeEntry Type Shape
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 05, Ch 06

#### Context
The knowledge pool is used for selective context injection — only relevant chunks are passed to the model, not the full pool.

#### Decision
`tags` act as section markers on a parent document, enabling the host app to filter and inject only relevant entries.

#### Implementation Contract
```typescript
// packages/types/src/knowledge-pool/index.ts
export type KnowledgeEntry = {
  id: string;
  title: string;
  description: string;
  source: string;
  tags: string[]; // section markers for selective model context injection
};
```

#### Acceptance Criteria
- [ ] `KnowledgeEntry` exported from `@jimzord12/types`
- [ ] `tags` used to filter entries before injecting into model context (enforced in Ch 06)

---

### Decision D-035: Message Type Strategy
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 03, Ch 04

#### Context
Defining a custom `Message` type would diverge from the Vercel AI SDK v7 and create a maintenance burden.

#### Decision
`packages/types` does NOT define a custom `Message` type. The library re-exports and extends Vercel AI SDK v7 native message types directly. `message/index.ts` is a thin re-export + any extension types.

#### Implementation Contract
```typescript
// packages/types/src/message/index.ts
export type { Message, CoreMessage } from 'ai'; // Vercel AI SDK v7
// Extension types (if needed) defined here
```

#### Acceptance Criteria
- [ ] No custom `Message` type defined in the codebase
- [ ] All packages import message types from `@jimzord12/types` (which proxies the SDK)

---

### Decision D-036: AIChatAssistantConfig Shape
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 03, Ch 06, IMP-004

#### Context
The host app must pass a config object to the widget. This is the top-level public API surface.

#### Implementation Contract
```typescript
// packages/types/src/config/index.ts
export type AIChatAssistantConfig = {
  userId: string;           // required — host app auth
  backendUrl: string;       // required — streaming endpoint URL
  knowledgePool?: KnowledgeEntry[];  // optional — for selective context injection
  botName?: string;         // optional — display name for the assistant
  welcomeMessage?: string;  // optional — first message shown to user
  theme?: 'light' | 'dark' | 'auto'; // optional — default: 'auto'
  maxMessages?: number;     // optional — cap on stored message history
};
```

#### Acceptance Criteria
- [ ] `AIChatAssistantConfig` exported from `@jimzord12/types`
- [ ] Both `<AIChatAssistant />` and `useAIChat` accept/require this config shape
- [ ] `userId` and `backendUrl` are required; all others optional

---

### Decision D-037: StorageAdapter Interface
> **Status:** Resolved
> **Phase Impact:** Phase 2+
> **Cross-Chapter Links:** Ch 05, IMP-001

#### Context
Phase 1 uses IndexedDB directly. Phase 2+ allows host apps to plug in custom storage via an adapter.

#### Implementation Contract
```typescript
// packages/types/src/storage/index.ts
import type { Message } from '../message';
import type { Result, AppError } from '../utils';

export interface StorageAdapter {
  load(userId: string): Promise<Result<Message[], AppError>>;
  save(userId: string, messages: Message[]): Promise<Result<void, AppError>>;
  clear(userId: string): Promise<Result<void, AppError>>;
}
```

#### Acceptance Criteria
- [ ] `StorageAdapter` exported from `@jimzord12/types`
- [ ] Phase 1 IndexedDB implementation satisfies this interface
- [ ] All three methods use `Result<T, AppError>` return types

---

### Decision D-038: AppError as Plain Type
> **Status:** Resolved
> **Phase Impact:** All

#### Context
`AppError` must work cleanly with the `Result<T, E>` pattern. A class adds unnecessary overhead.

#### Options Considered
| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| Plain type | Lightweight, works with Result | No stack trace, no instanceof | **Chosen** |
| Class extending Error | Stack traces, instanceof checks | Heavier, less functional-friendly | Rejected |

#### Decision
`AppError` is a plain `type`, not a class.

#### Acceptance Criteria
- [ ] `AppError` defined as `type`, not `class`, in `packages/types/src/utils/`

---

### Decision D-039: Streaming Type Contract
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 04, IMP-002

#### Context
`streaming.ts` must define what the backend streaming contract looks like without over-engineering.

#### Options Considered
| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| Re-export SDK types with JSDoc | Thin, stays in sync with SDK | No custom abstraction layer | **Chosen** |
| Custom wrapper interface | Decoupled from SDK | Maintenance burden, divergence risk | Rejected |

#### Decision
`streaming/index.ts` re-exports Vercel AI SDK v7 streaming types with JSDoc comments explaining the backend contract requirement.

#### Implementation Contract
```typescript
// packages/types/src/streaming/index.ts
/**
 * The backend endpoint must implement the Vercel AI SDK v7 streaming contract.
 * See: https://sdk.vercel.ai/docs/ai-sdk-ui/stream-protocol
 */
export type { StreamProtocol } from 'ai'; // Vercel AI SDK v7
```

#### Acceptance Criteria
- [ ] `streaming/index.ts` exists and re-exports relevant SDK types
- [ ] JSDoc comment references the Vercel AI SDK streaming protocol docs

---

### Decision D-040: ChatUser Type Shape
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 03, IMP-003

#### Context
Auth is fully the host app's responsibility. The widget only needs the `userId` to scope storage and requests.

#### Decision
`ChatUser` is minimal — `{ userId: string }` only. No display fields (name, avatar) are part of the widget's concern.

#### Implementation Contract
```typescript
// packages/types/src/user/index.ts
export type ChatUser = {
  userId: string;
};
```

#### Acceptance Criteria
- [ ] `ChatUser` exported from `@jimzord12/types`
- [ ] No avatar, displayName, or auth token fields on `ChatUser`

---

### Decision D-041: Enum Value Casing
> **Status:** Resolved
> **Phase Impact:** All

#### Context
Enum values must follow a consistent casing convention, ideally matching the error code convention for uniformity.

#### Decision
All enum values use `SCREAMING_SNAKE_CASE` (e.g. `MessageRole.USER`, `MessageRole.ASSISTANT`, `MessageRole.SYSTEM`).

#### Implementation Contract
```typescript
// packages/types/src/message/index.ts
export enum MessageRole {
  USER = 'USER',
  ASSISTANT = 'ASSISTANT',
  SYSTEM = 'SYSTEM',
}
```

#### Acceptance Criteria
- [ ] All enums in `packages/types` use `SCREAMING_SNAKE_CASE` values
- [ ] No PascalCase or camelCase enum values anywhere in the codebase

---

## 3. Open Questions

_None — all questions resolved._

---

## 4. Design Patterns & Code Principles

### 4.1 Patterns Applied
| Pattern | Where | How |
|---------|-------|-----|
| Result<T, E> | All failable operations | Returns `{ ok: true; value }` or `{ ok: false; error }` instead of throwing |
| Barrel re-exports | Every `src/` subfolder | Each domain folder exposes a clean `index.ts` API |
| SDK type proxying | `message/`, `streaming/` | Re-exports SDK types to keep the rest of the codebase SDK-agnostic at import level |

### 4.2 Principles Enforced
- **Single source of truth:** All shared types live in `@jimzord12/types` — never duplicated
- **Explicit over implicit:** `Result<T, E>` forces error handling at call sites
- **Strict TypeScript:** `"strict": true` everywhere, no escape hatches
- **Modern-only:** ESM only, React 19+, no CJS or legacy compat

### 4.3 Anti-Patterns to Avoid
- Redefining `AppError` or `Result` in individual packages
- Using relative `../../` imports across package boundaries
- Defining custom `Message` types that diverge from Vercel AI SDK v7
- Using `any` or `// @ts-ignore` without documented justification
- Throwing errors where `Result<T, E>` is the expected return type

---

## 5. Agent Instructions

### 5.1 Always Do
- [ ] Import all shared types from `@jimzord12/types`
- [ ] Return `Result<T, AppError>` from any failable async operation
- [ ] Use `SCREAMING_SNAKE_CASE` for all error codes and enum values
- [ ] Name folders in kebab-case and files in camelCase
- [ ] Add a barrel `index.ts` to every new `src/` subfolder

### 5.2 Ask Before Doing
- [ ] Adding a new domain file to `packages/types` (check if it belongs in another chapter first)
- [ ] Extending `AIChatAssistantConfig` with new required fields (breaking change)

### 5.3 Never Do
- [ ] Define a custom `Message` type — always use/extend the Vercel AI SDK v7 type
- [ ] Use relative imports across package boundaries
- [ ] Use `class` for `AppError`
- [ ] Use `throw` where `Result<T, AppError>` is the expected return pattern
- [ ] Relax TypeScript strict mode for any package

---

## 6. Cross-Chapter Impact Register
| Decision ID | Affects Chapter | Impact Description | Propagated? |
|-------------|-----------------|-------------------|------------|
| D-022 | Ch 05, Ch 06 | Domain-split types structure defines how each chapter imports its types | No |
| D-030 | Ch 04, Ch 05 | All failable ops must return `Result<T, AppError>` | No |
| D-034 | Ch 05, Ch 06 | `KnowledgeEntry` shape drives storage and config decisions | No |
| D-036 | Ch 03, Ch 06 | `AIChatAssistantConfig` is the top-level public API — all config decisions cascade from here | No |
| D-037 | Ch 05 | `StorageAdapter` interface must be satisfied by Phase 1 IndexedDB impl | No |
| D-039 | Ch 04 | Backend must implement Vercel AI SDK v7 streaming contract | No |

---

## 7. Session History
| Session # | Date | Questions Resolved | Decisions Added | Notes |
|-----------|------|--------------------|-----------------|-------|
| 1 | 2026-06-30 | 20 | D-022 to D-041 | Full scoping session. Chapter locked in one pass. |
