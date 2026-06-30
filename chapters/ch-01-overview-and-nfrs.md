# Chapter 01: Overview & NFRs

> **Status:** SCOPED
> **Primary Consumer:** Both Teams
> **Dependencies:** None
> **Dependents:** Ch 02: Architecture & Topology
> **Sessions Contributed:** 1
> **Questions Resolved:** 0 / 8

---

## 1. Context & Scope

### 1.1 What This Chapter Covers
The product identity, target user, core constraints, and non-functional requirements (NFRs) that all other chapters must respect. This is the source of truth for *what* the product is and *what guarantees* it makes.

### 1.2 What This Chapter Does NOT Cover
- Internal architecture decisions (Ch 02)
- API design or streaming protocol (Ch 04)
- UI implementation details (Ch 03)
- Backend deployment setup (Ch 10)

### 1.3 Key Stakeholders
- **Owner:** Full-stack developer (jimzord12)
- **Consumers:** All subsequent chapters inherit constraints defined here

---

## 2. Decision Log

### Decision [D-001]: Product Type
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 02, Ch 03

#### Context
Defines the fundamental delivery mechanism of the product.

#### Decision
This is a **distributable npm React component** — installed via `npm install`, configured via props/config, and embedded into any React/Next.js app. It is NOT a hosted chatbot service, SaaS platform, or standalone page.

#### Implementation Contract
```typescript
// Published as an npm package
// Consumer installs: npm install @your-org/ai-chat-assistant
// Consumer uses:
import { AIChatAssistant } from '@your-org/ai-chat-assistant';
<AIChatAssistant config={...} userId={...} />
```

#### Acceptance Criteria
- [ ] Component is installable via npm/pnpm/yarn
- [ ] Component renders without a host-provided backend (graceful error state)
- [ ] No hosted infrastructure is required to use the component

---

### Decision [D-002]: Target Deployer
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 06

#### Context
Who configures and integrates the component?

#### Decision
**Developers only.** Configuration is done entirely in code via props and/or a config object. No admin UI, dashboard, or non-technical interface is required or planned.

#### Implementation Contract
```typescript
// All configuration is done via TypeScript props/config
interface AIChatAssistantConfig {
  apiUrl: string;        // backend endpoint URL
  userId: string;        // authenticated user ID from host app
  // ...further config defined in Ch 06
}
```

#### Acceptance Criteria
- [ ] All configuration options are TypeScript-typed
- [ ] No runtime config loading from external dashboard or CMS

---

### Decision [D-003]: AI Provider Strategy
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 04, Ch 00

#### Context
Which AI providers does the component support?

#### Decision
**Provider-agnostic**, built on **Vercel AI SDK v7**. The component does not call any AI provider directly. It communicates with a developer-supplied backend URL that must implement the Vercel AI SDK v7 streaming contract (`createUIMessageStreamResponse` + `toUIMessageStream`). The backend may use any provider (OpenAI, Anthropic, Gemini, etc.).

#### Implementation Contract
```typescript
// Component POSTs to developer-supplied URL:
POST {config.apiUrl}
Content-Type: application/json
Body: { messages: UIMessage[] }

// Backend MUST return a Vercel AI SDK v7 compatible stream response.
// See: docs/references/vercel-ai-sdk-v7-chatbot-crash-course.md
```

#### Acceptance Criteria
- [ ] Component works with any backend that implements the Vercel AI SDK v7 contract
- [ ] Component does not import any AI provider SDK directly
- [ ] Swapping backend provider requires zero changes to component config

---

### Decision [D-004]: Scale Target (NFR)
> **Status:** Resolved
> **Phase Impact:** Phase 1
> **Cross-Chapter Links:** Ch 07, Ch 10

#### Context
What concurrent load must the system handle at launch?

#### Decision
**< 100 concurrent users** at Phase 1 launch. No distributed rate-limiting, CDN streaming infrastructure, or horizontal scaling is required for Phase 1. This constraint must be revisited before Phase 2.

#### Acceptance Criteria
- [ ] System handles 100 concurrent chat sessions without degradation
- [ ] Phase 2 planning includes scale re-evaluation

---

### Decision [D-005]: Persistence (NFR)
> **Status:** Resolved
> **Phase Impact:** Phase 1
> **Cross-Chapter Links:** Ch 05

#### Context
Must chat history survive page refresh/close?

#### Decision
**Yes — persistence is required.** Phase 1 uses **IndexedDB** (client-side only). No backend storage is required in Phase 1. The storage layer must be designed via an adapter interface so Phase 2 can swap to server-side storage without breaking the component API.

#### Implementation Contract
```typescript
// Phase 1: IndexedDB adapter
// Phase 2+: Server-side adapter (same interface)
// Adapter interface defined in Ch 05
interface StorageAdapter {
  getThreads(userId: string): Promise<Thread[]>;
  saveMessage(threadId: string, message: UIMessage): Promise<void>;
  clearThread(threadId: string): Promise<void>;
}
```

#### Acceptance Criteria
- [ ] Chat history survives page refresh
- [ ] Chat history is scoped per `userId`
- [ ] Storage implementation is behind an adapter interface (swappable)

---

### Decision [D-006]: Auth Model
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 07

#### Context
Are anonymous users supported?

#### Decision
**Authenticated users only.** The host app owns authentication entirely. The component receives a `userId` prop (string) from the host app and trusts it. The component does not perform any auth, token validation, or login flow.

#### Implementation Contract
```typescript
// Host app is responsible for authentication
// Component receives userId as a required prop
interface AIChatAssistantProps {
  userId: string; // required — component will not render without it
  // ...
}
```

#### Acceptance Criteria
- [ ] Component refuses to render if `userId` is not provided
- [ ] Component does not contain any login UI or auth logic
- [ ] All persisted data is namespaced under `userId`

---

### Decision [D-007]: Backend Contract
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 04

#### Context
What does the component require from the developer's backend?

#### Decision
**Backend-agnostic** — the component only requires a POST endpoint URL. However, the backend **must** implement the **Vercel AI SDK v7 streaming response contract**. Any framework (Next.js, Express, Hono, Elysia, etc.) is valid as long as the response shape is compatible.

#### Acceptance Criteria
- [ ] Component works with Next.js App Router API routes
- [ ] Component works with Express/Elysia standalone servers
- [ ] Documentation clearly states the required backend contract

---

### Decision [D-008]: Multi-Tenancy
> **Status:** Resolved
> **Phase Impact:** All
> **Cross-Chapter Links:** Ch 06

#### Context
How many bots can run per installation?

#### Decision
**One bot/persona per installation.** A single component instance = a single AI persona with a single config. Multiple bots in the same app require multiple component instances with separate configs. No internal routing between personas.

#### Acceptance Criteria
- [ ] A single `<AIChatAssistant />` instance represents exactly one bot persona
- [ ] Multiple instances with different configs can coexist in the same app without conflict

---

## 3. Open Questions

> All 8 scoping questions resolved. No open questions remaining.

---

## 4. Design Patterns & Code Principles

### 4.1 Patterns Applied
| Pattern | Where | How |
|---------|-------|-----|
| Adapter | Storage (Ch 05) | IndexedDB in Phase 1, swappable to server-side in Phase 2 |
| Convention over Configuration | Component API | Sensible defaults, minimal required props |

### 4.2 Principles Enforced
- **Self-contained:** Component has no hard dependency on any specific backend framework or AI provider
- **Host-app trust model:** Component trusts what the host app provides (userId, apiUrl) — no internal validation of auth
- **Phase-aware design:** Phase 1 constraints (IndexedDB, <100 users) must not couple the architecture permanently

### 4.3 Anti-Patterns to Avoid
- **Never** import an AI provider SDK directly into the component
- **Never** render the component without a `userId` prop
- **Never** hardcode a backend URL — always require it via config

---

## 5. Agent Instructions (CLAUDE.md Directives)

### 5.1 Always Do
- [ ] Treat `userId` as a required prop — all data must be namespaced under it
- [ ] Reference `docs/references/vercel-ai-sdk-v7-chatbot-crash-course.md` when making streaming decisions
- [ ] Design storage behind an adapter interface (never couple directly to IndexedDB)

### 5.2 Ask Before Doing
- [ ] Any change that would require anonymous user support
- [ ] Any change that introduces a direct AI provider dependency into the component

### 5.3 Never Do
- [ ] Import `openai`, `anthropic`, or any provider SDK directly in the component package
- [ ] Render without a valid `userId`
- [ ] Couple Phase 1 storage (IndexedDB) permanently into the architecture

---

## 6. Cross-Chapter Impact Register
| Decision ID | Affects Chapter | Impact Description | Propagated? |
|-------------|-----------------|-------------------|------------|
| D-005 | Ch 05 | Storage must be adapter-based; IndexedDB is Phase 1 default | Pending |
| D-003 | Ch 04 | Backend must implement Vercel AI SDK v7 streaming contract | Pending |
| D-006 | Ch 07 | Auth is host-app responsibility; component receives userId prop only | Pending |
| D-008 | Ch 06 | Single config per instance; no internal persona routing needed | Pending |

---

## 7. Session History
| Session # | Date | Questions Resolved | Decisions Added | Notes |
|-----------|------|--------------------|-----------------|-------|
| 1 | 2026-06-30 | 8 / 8 | D-001 to D-008 | Scoping session. All 8 questions answered. Chapter moved to SCOPED. |
