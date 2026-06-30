# Chapter 02: Architecture & Topology

> **Status:** LOCKED
> **Primary Consumer:** Both Teams
> **Dependencies:** Ch 01 ✅
> **Dependents:** Ch 00: Agent Conventions & Shared Contract
> **Sessions Contributed:** 2
> **Questions Resolved:** 13 / 13

---

## 1. Context & Scope

### 1.1 What This Chapter Covers
The monorepo structure, package topology, component architecture pattern, styling approach, build tooling, output format, TypeScript config strategy, versioning, workspace references, sub-path exports, and CI/CD pipeline.

### 1.2 What This Chapter Does NOT Cover
- Internal component state management (Ch 03)
- API route design and streaming protocol (Ch 04)
- Storage adapter implementation (Ch 05)
- Config schema design (Ch 06)

### 1.3 Key Stakeholders
- **Owner:** jimzord12
- **Consumers:** All chapters — folder paths, package names, and import conventions defined here are referenced everywhere

---

## 2. Decision Log

### Decision [D-009]: Repo Structure
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 00

#### Decision
Monorepo with pnpm workspaces + Turborepo.

#### Implementation Contract
```
ai-chat-assistant/
├── packages/
│   ├── ui/        ← @jimzord12/ai-chat-assistant (published)
│   └── types/     ← @jimzord12/types (published)
├── apps/
│   └── example/   ← Next.js 16+ (not published)
├── turbo.json
└── pnpm-workspace.yaml
```

#### Acceptance Criteria
- [ ] `pnpm install` from root installs all workspace dependencies
- [ ] `turbo build` builds packages in correct order
- [ ] `apps/example` imports from packages via workspace protocol

---

### Decision [D-010]: Package Naming
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 00, Ch 03

#### Decision
- `@jimzord12/ai-chat-assistant` in `packages/ui`
- `@jimzord12/types` in `packages/types`
- `apps/example` — not published

#### Acceptance Criteria
- [ ] Both packages publishable to npm
- [ ] No name collisions

---

### Decision [D-011]: Component Architecture Pattern
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 03

#### Decision
Headless + Styled default.
1. `useAIChat` — headless logic hook, zero UI
2. `<AIChatAssistant />` — pre-styled default built on `useAIChat`

#### Implementation Contract
```typescript
// Headless: packages/ui/src/hooks/useAIChat.ts
export function useAIChat(config: AIChatConfig): AIChatReturn { ... }

// Styled: packages/ui/src/components/AIChatAssistant.tsx
export function AIChatAssistant(props: AIChatAssistantProps): JSX.Element { ... }

// Consumer (headless):
import { useAIChat } from '@jimzord12/ai-chat-assistant/headless';
// Consumer (styled):
import { AIChatAssistant } from '@jimzord12/ai-chat-assistant/component';
```

#### Acceptance Criteria
- [ ] `useAIChat` has no JSX or CSS dependencies
- [ ] `<AIChatAssistant />` built entirely on `useAIChat`
- [ ] Both exports available via sub-paths

---

### Decision [D-012]: Styling Strategy
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 03

#### Decision
CSS Modules. Scoped, zero consumer toolchain dependency.

#### Acceptance Criteria
- [ ] No global CSS leaked into consumer app
- [ ] Renders correctly with zero consumer CSS config
- [ ] CSS Modules bundled by tsdown

---

### Decision [D-013]: Build Output Format
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 00

#### Decision
ESM only. No CJS.

#### Implementation Contract
```json
{
  "type": "module",
  "exports": {
    ".": { "import": "./dist/index.js", "types": "./dist/index.d.ts" },
    "./headless": { "import": "./dist/headless.js", "types": "./dist/headless.d.ts" },
    "./component": { "import": "./dist/component.js", "types": "./dist/component.d.ts" }
  }
}
```

#### Acceptance Criteria
- [ ] ESM output only, `.d.ts` for all sub-paths
- [ ] No `require()` entrypoint

---

### Decision [D-014]: React Version Target
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 03

#### Decision
React 19 minimum.

#### Implementation Contract
```json
{ "peerDependencies": { "react": ">=19.0.0", "react-dom": ">=19.0.0" } }
```

#### Acceptance Criteria
- [ ] `react >=19.0.0` declared as peer dependency
- [ ] No deprecated React 18 patterns used

---

### Decision [D-015]: Build Tooling
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 00

#### Decision
tsdown (Rolldown-based, modern successor to tsup).

#### Implementation Contract
```typescript
// packages/ui/tsdown.config.ts
import { defineConfig } from 'tsdown';
export default defineConfig({
  entry: { index: 'src/index.ts', headless: 'src/headless.ts', component: 'src/component.ts' },
  format: ['esm'],
  dts: true,
  clean: true,
});
```

#### Acceptance Criteria
- [ ] `tsdown build` produces all three entry points with declarations
- [ ] Zero TypeScript errors on build

---

### Decision [D-016]: Example App
> **Status:** Resolved | **Phase Impact:** Phase 1 | **Cross-Chapter Links:** Ch 04

#### Decision
Next.js 16+ App Router. Dev sandbox + canonical Vercel AI SDK v7 backend reference.

#### Implementation Contract
```
apps/example/app/
├── page.tsx             ← demo page
└── api/chat/route.ts    ← Vercel AI SDK v7 reference backend
```

#### Acceptance Criteria
- [ ] Imports via `workspace:*`
- [ ] API route implements Vercel AI SDK v7 streaming contract
- [ ] `pnpm dev` works end-to-end

---

### Decision [D-017]: TypeScript Config Strategy
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 00

#### Decision
Root `tsconfig.base.json` extended by all packages and apps.

#### Implementation Contract
```json
// tsconfig.base.json (root)
{ "compilerOptions": { "strict": true, "target": "ES2022", "module": "ESNext", "moduleResolution": "Bundler", "jsx": "react-jsx", "declaration": true, "skipLibCheck": true } }

// packages/ui/tsconfig.json
{ "extends": "../../tsconfig.base.json", "compilerOptions": { "outDir": "dist" }, "include": ["src"] }
```

#### Acceptance Criteria
- [ ] All packages pass `tsc --noEmit`
- [ ] Base config change propagates to all packages

---

### Decision [D-018]: Versioning & Publishing
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 00

#### Decision
Changesets (`@changesets/cli`). PR → changeset file → CI versions + publishes on merge to main.

#### Acceptance Criteria
- [ ] `pnpm changeset` generates changeset file
- [ ] CI auto-versions and publishes on merge
- [ ] `CHANGELOG.md` auto-generated per package

---

### Decision [D-019]: Workspace Package References
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 00

#### Decision
`workspace:*` for all internal references. Changesets rewrites to exact versions at publish.

#### Implementation Contract
```json
{ "dependencies": { "@jimzord12/types": "workspace:*" } }
```

#### Acceptance Criteria
- [ ] Internal imports resolve correctly in dev
- [ ] Published packages contain exact version numbers

---

### Decision [D-020]: Sub-path Exports
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 03

#### Decision
Three sub-paths: `.` (all), `./headless` (hook only, no CSS), `./component` (styled + CSS).

#### Acceptance Criteria
- [ ] `./headless` import has zero CSS side-effects
- [ ] `./component` import includes styles
- [ ] Root `.` re-exports both without duplication

---

### Decision [D-021]: CI/CD Pipeline
> **Status:** Resolved | **Phase Impact:** All | **Cross-Chapter Links:** Ch 10

#### Decision
Full pipeline on every PR/push: type-check → lint → build → tests via GitHub Actions + Turborepo.

#### Implementation Contract
```yaml
jobs:
  ci:
    steps:
      - pnpm install
      - turbo type-check
      - turbo lint
      - turbo build
      - turbo test
```

#### Acceptance Criteria
- [ ] PRs blocked on any CI failure
- [ ] CI completes under 5 minutes

---

## 3. Open Questions
> All 13 decisions resolved. No open questions.

---

## 4. Design Patterns & Code Principles

### 4.1 Patterns Applied
| Pattern | Where | How |
|---------|-------|-----|
| Facade | `<AIChatAssistant />` | Wraps `useAIChat` into a simple drop-in |
| Separation of Concerns | Headless vs. styled | Logic and UI fully decoupled via sub-path exports |
| Convention over Configuration | Monorepo structure | `packages/` publishable, `apps/` internal |
| Single Source of Truth | `tsconfig.base.json` | One base TS config, all packages extend |

### 4.2 Principles Enforced
- Modern-only: React 19+, ESM, Next.js 16+, tsdown
- Zero consumer tooling requirements
- Headless-first: `useAIChat` is source of truth
- Tree-shakeable by design via sub-path exports

### 4.3 Anti-Patterns to Avoid
- **Never** publish CJS output
- **Never** import Tailwind or CSS-in-JS into the package
- **Never** put shared types inside `packages/ui`
- **Never** use fixed version numbers for internal references during development

---

## 5. Agent Instructions

### 5.1 Always Do
- [ ] Use `@jimzord12/types` for all shared interfaces
- [ ] Keep `useAIChat` free of JSX/CSS
- [ ] Use `workspace:*` for internal references
- [ ] Reference `apps/example/app/api/chat/route.ts` as canonical backend

### 5.2 Ask Before Doing
- [ ] Changes to monorepo folder structure
- [ ] Adding a new publishable package
- [ ] Changes to sub-path export structure

### 5.3 Never Do
- [ ] Add CJS output
- [ ] Add React 18 shims
- [ ] Place shared types in `packages/ui`
- [ ] Use fixed version numbers for `workspace:*` references

---

## 6. Cross-Chapter Impact Register
| Decision ID | Affects Chapter | Impact Description | Propagated? |
|-------------|-----------------|-------------------|------------|
| D-009 | Ch 00 | Monorepo structure defines folder conventions | Pending |
| D-010 | Ch 00, Ch 03 | Package names define all import statements | Pending |
| D-011 | Ch 03 | Headless + styled defines component API surface | Pending |
| D-013 | Ch 00 | ESM-only constrains all import/export conventions | Pending |
| D-016 | Ch 04 | Example API route is canonical backend reference | Pending |
| D-017 | Ch 00 | Root tsconfig.base.json is TS config source of truth | Pending |
| D-020 | Ch 03 | Sub-path exports define headless vs. styled imports | Pending |
| D-021 | Ch 10 | CI pipeline steps and budget defined here | Pending |

---

## 7. Session History
| Session # | Date | Questions Resolved | Decisions Added | Notes |
|-----------|------|--------------------|-----------------|-------|
| 1 | 2026-06-30 | 8 / 13 | D-009 to D-016 | Initial scoping. Prematurely locked at 8. |
| 2 | 2026-06-30 | 13 / 13 | D-017 to D-021 | Reopened. Added tsconfig, changesets, workspace refs, sub-paths, CI. Re-locked properly. |
