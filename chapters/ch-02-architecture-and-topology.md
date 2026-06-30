# Chapter 02: Architecture & Topology

> **Status:** LOCKED
> **Primary Consumer:** Both Teams
> **Dependencies:** Ch 01 ✅
> **Dependents:** Ch 00: Agent Conventions & Shared Contract
> **Sessions Contributed:** 1
> **Questions Resolved:** 8 / 8

---

## 1. Context & Scope

### 1.1 What This Chapter Covers
The monorepo structure, package topology, component architecture pattern, styling approach, build tooling, output format, and minimum runtime requirements. This chapter defines the physical layout of the codebase and the technical constraints every other chapter must respect.

### 1.2 What This Chapter Does NOT Cover
- Internal component state management (Ch 03)
- API route design and streaming protocol (Ch 04)
- Storage adapter implementation (Ch 05)
- Config schema design (Ch 06)

### 1.3 Key Stakeholders
- **Owner:** jimzord12 (full-stack developer)
- **Consumers:** All chapters — folder paths, package names, and import conventions defined here are referenced everywhere

---

## 2. Decision Log

### Decision [D-009]: Repo Structure
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 00

#### Context
How is the codebase organised across packages and apps?

#### Decision
**Monorepo** managed with **pnpm workspaces** and **Turborepo**. All packages and apps live in a single repository.

#### Implementation Contract
```
ai-chat-assistant/           ← root monorepo
├── packages/
│   ├── ui/                  ← @jimzord12/ai-chat-assistant (published)
│   └── types/               ← @jimzord12/types (published)
├── apps/
│   └── example/             ← Next.js 16+ dev/demo app (not published)
├── turbo.json
├── pnpm-workspace.yaml
└── package.json
```

#### Acceptance Criteria
- [ ] `pnpm install` from root installs all workspace dependencies
- [ ] `turbo build` builds all packages in correct dependency order
- [ ] `apps/example` can import from `packages/ui` and `packages/types` via workspace protocol

---

### Decision [D-010]: Package Naming
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 00, Ch 03

#### Context
What are the published npm package names?

#### Decision
- Component package: **`@jimzord12/ai-chat-assistant`** (in `packages/ui`)
- Shared types package: **`@jimzord12/types`** (in `packages/types`)
- Example app: not published, internal only

#### Implementation Contract
```json
// packages/ui/package.json
{ "name": "@jimzord12/ai-chat-assistant" }

// packages/types/package.json
{ "name": "@jimzord12/types" }

// apps/example/package.json
{ "name": "example" }
```

#### Acceptance Criteria
- [ ] `@jimzord12/ai-chat-assistant` is publishable to npm
- [ ] `@jimzord12/types` is publishable to npm as a peer dependency
- [ ] No name collisions with existing npm packages

---

### Decision [D-011]: Component Architecture Pattern
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 03

#### Context
How is the component structured for consumers?

#### Decision
**Headless + Styled default.** Two layers:
1. `useAIChat` — headless logic hook, zero UI, full control for custom implementations
2. `<AIChatAssistant />` — pre-styled default component built on top of `useAIChat`

#### Implementation Contract
```typescript
// Layer 1: Headless hook (packages/ui/src/hooks/useAIChat.ts)
export function useAIChat(config: AIChatConfig): AIChatReturn { ... }

// Layer 2: Pre-styled default (packages/ui/src/components/AIChatAssistant.tsx)
export function AIChatAssistant(props: AIChatAssistantProps): JSX.Element { ... }

// Consumer using headless:
import { useAIChat } from '@jimzord12/ai-chat-assistant';

// Consumer using default styled:
import { AIChatAssistant } from '@jimzord12/ai-chat-assistant';
```

#### Acceptance Criteria
- [ ] `useAIChat` has no JSX or styling dependencies
- [ ] `<AIChatAssistant />` is built entirely using `useAIChat` internally
- [ ] Both exports are available from the package root

---

### Decision [D-012]: Styling Strategy
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 03

#### Context
How are styles scoped and delivered in the published package?

#### Decision
**CSS Modules.** All styles are scoped via CSS Modules. Zero dependency on the consumer's CSS toolchain (no Tailwind required, no runtime CSS-in-JS).

#### Implementation Contract
```
packages/ui/src/components/
├── AIChatAssistant.tsx
├── AIChatAssistant.module.css
├── MessageList.tsx
├── MessageList.module.css
└── ...
```

#### Acceptance Criteria
- [ ] No global CSS classes leaked into consumer app
- [ ] Component renders correctly with zero CSS configuration from consumer
- [ ] CSS Modules are bundled into the output by tsdown

---

### Decision [D-013]: Build Output Format
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 00

#### Context
What module format does the published package expose?

#### Decision
**ESM only.** No CJS output. Targets modern bundlers (Next.js 16+, Vite, etc.).

#### Implementation Contract
```json
// packages/ui/package.json
{
  "type": "module",
  "exports": {
    ".": {
      "import": "./dist/index.js",
      "types": "./dist/index.d.ts"
    }
  }
}
```

#### Acceptance Criteria
- [ ] Published package contains only ESM output (`dist/index.js`)
- [ ] TypeScript declarations (`.d.ts`) are included in the publish
- [ ] No `require()` entrypoint exposed

---

### Decision [D-014]: React Version Target
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 03

#### Context
What is the minimum supported React version?

#### Decision
**React 19 minimum.** No legacy React support. Enables use of `useOptimistic`, `useFormStatus`, Actions, and the `use()` hook where appropriate.

#### Implementation Contract
```json
// packages/ui/package.json
{
  "peerDependencies": {
    "react": ">=19.0.0",
    "react-dom": ">=19.0.0"
  }
}
```

#### Acceptance Criteria
- [ ] Package declares `react >=19.0.0` as a peer dependency
- [ ] No React 18-only APIs or deprecated patterns used
- [ ] CI runs tests against React 19

---

### Decision [D-015]: Build Tooling
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 00

#### Context
What tool bundles the component package for publishing?

#### Decision
**tsdown** — modern successor to tsup, built on Rolldown (Rust-based). Handles ESM output + `.d.ts` generation.

#### Implementation Contract
```typescript
// packages/ui/tsdown.config.ts
import { defineConfig } from 'tsdown';

export default defineConfig({
  entry: ['src/index.ts'],
  format: ['esm'],
  dts: true,
  clean: true,
});
```

#### Acceptance Criteria
- [ ] `tsdown build` produces `dist/index.js` and `dist/index.d.ts`
- [ ] CSS Modules are handled and bundled correctly
- [ ] Build completes with zero TypeScript errors

---

### Decision [D-016]: Example App
> **Status:** Resolved
> **Phase Impact:** Phase 1
> **Cross-Chapter Links:** Ch 04

#### Context
What framework powers the development and demo app?

#### Decision
**Next.js 16+ (App Router).** Serves dual purpose: development sandbox for the component and canonical reference implementation of the Vercel AI SDK v7 backend contract.

#### Implementation Contract
```
apps/example/
├── app/
│   ├── page.tsx              ← demo page using <AIChatAssistant />
│   └── api/
│       └── chat/
│           └── route.ts      ← reference backend (Vercel AI SDK v7)
├── package.json
└── next.config.ts
```

#### Acceptance Criteria
- [ ] Example app imports `@jimzord12/ai-chat-assistant` via workspace protocol
- [ ] `apps/example/app/api/chat/route.ts` implements the Vercel AI SDK v7 streaming contract
- [ ] `pnpm dev` from `apps/example` starts a working demo

---

## 3. Open Questions

> All 8 scoping questions resolved. No open questions remaining.

---

## 4. Design Patterns & Code Principles

### 4.1 Patterns Applied
| Pattern | Where | How |
|---------|-------|-----|
| Facade | `<AIChatAssistant />` | Wraps `useAIChat` into a simple one-prop-drop-in |
| Separation of Concerns | Headless hook vs. styled component | Logic and UI are fully decoupled |
| Convention over Configuration | Monorepo folder structure | `packages/` for publishable, `apps/` for internal |

### 4.2 Principles Enforced
- **Modern-only:** React 19+, ESM, Next.js 16+, tsdown — no legacy support
- **Zero consumer tooling requirements:** CSS Modules, no Tailwind dependency
- **Headless-first:** `useAIChat` is the source of truth; the styled component is a convenience layer

### 4.3 Anti-Patterns to Avoid
- **Never** publish CJS output or add a `require()` entrypoint
- **Never** import consumer-side tooling (Tailwind, styled-components) into the package
- **Never** put shared types directly in `packages/ui` — they belong in `packages/types`

---

## 5. Agent Instructions

### 5.1 Always Do
- [ ] Use `@jimzord12/types` for all shared TypeScript interfaces
- [ ] Keep `useAIChat` free of any JSX or CSS imports
- [ ] Reference `apps/example/app/api/chat/route.ts` as the canonical backend implementation

### 5.2 Ask Before Doing
- [ ] Any change to the monorepo folder structure
- [ ] Any addition of a new publishable package

### 5.3 Never Do
- [ ] Add CJS output to any package
- [ ] Add React 18 compatibility shims
- [ ] Place shared types inside `packages/ui`

---

## 6. Cross-Chapter Impact Register
| Decision ID | Affects Chapter | Impact Description | Propagated? |
|-------------|-----------------|-------------------|------------|
| D-009 | Ch 00 | Monorepo structure defines folder conventions and import paths | Pending |
| D-010 | Ch 00, Ch 03 | Package names define all import statements across the spec | Pending |
| D-011 | Ch 03 | Headless + styled pattern defines component API surface | Pending |
| D-013 | Ch 00 | ESM-only output constrains all import/export conventions | Pending |
| D-016 | Ch 04 | Example app API route is the canonical backend contract reference | Pending |

---

## 7. Session History
| Session # | Date | Questions Resolved | Decisions Added | Notes |
|-----------|------|--------------------|-----------------|-------|
| 1 | 2026-06-30 | 8 / 8 | D-009 to D-016 | Scoping session. All 8 questions answered. Self-review passed. Chapter fast-tracked to LOCKED. |
