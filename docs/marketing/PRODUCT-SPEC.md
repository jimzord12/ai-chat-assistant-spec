# Product Spec & Team Alignment – AI Chat Assistant NPM Package

**Document Purpose**: This file acts as the single source of truth and “contract” between Marketing, Development, and Stakeholders. It translates marketing positioning into concrete, prioritized deliverables so everyone stays aligned on what will be shipped.

**Project Owner**: Dimitrios Stamatakis  
**Version**: 1.0 (Initial MVP + Near-term)  
**Target Audience**: Greek & EU web platforms (internal tools + customer-facing)

> ⚠️ **Draft v1** — Not 100% accurate. To be reviewed and refined with the project owner.

---

## Core Value Proposition (Marketing → Dev Translation)

**Marketing Promise**: A modern, embeddable, developer-friendly AI chat assistant that reduces support load, feels native to any design system, and gives full control (no vendor lock-in).

**Dev Commitment**: Deliver a high-quality, reusable npm package that is easy to install, customize, maintain, and extend while meeting performance, security, and privacy standards suitable for production use in Greece/EU.

---

## Prioritized Features

### Phase 1: MVP (Must Ship for Initial Release)
These features directly support the core marketing claims.

- **Installation & Setup**
  - One-command npm install + minimal config
  - Excellent documentation (setup guide, customization examples, troubleshooting)
  - Presets for quick integration + headless mode for full design system control

- **LLM Integration**
  - Full Vercel AI SDK + assistant-ui support
  - Provider agnostic (easy swapping between OpenAI, Anthropic, Google, etc.)

- **Session Management**
  - Local OPFS + SQLite persistence
  - Features: create/delete sessions, clear all, restore previous, export summary

- **Safety & Control**
  - Rate limiting and auth management hooks
  - Pre-defined DOM manipulation tools (highlight elements, navigation, button clicks, form filling)
  - Moderate guardrails (prevent sensitive data exposure and dangerous actions)

- **Customization & UX**
  - Headless + theme/design system compatibility
  - Responsive, accessible (a11y), mobile-first
  - Streaming responses

- **RAG Foundations**
  - Basic Knowledge Pool management structure (expandable)

**Definition of Done for MVP**:
- Package published to npm (private or public)
- Works in a real web app (demo or company integration)
- Docs sufficient for other devs to integrate independently
- Basic tests + security review for DOM tools/guardrails

---

### Phase 2: Near-Term (1–2 Months Post-MVP)
These enhance marketability and user delight.

- **User Experience Enhancements**
  - Quick reply suggestions / prompt chips
  - Feedback buttons (thumbs up/down)
  - Multi-modal support (images, basic file handling)
  - Shareable conversation links

- **Advanced Capabilities**
  - Improved RAG implementation with knowledge ingestion
  - Better guardrails + configurable policies
  - Analytics hooks (usage, popular queries – privacy respecting)

- **Greek/EU Specific**
  - Strong Greek language support + automatic detection
  - GDPR-friendly data handling, consent flows, local storage emphasis
  - Multilingual readiness (Greek + English + major EU)

- **Polish**
  - Voice input basics
  - Theme modes (light/dark/system)
  - Error handling & fallback UX

---

### Future / Nice-to-Haves (Backlog)
- Generative UI / dynamic artifacts
- Advanced agentic tool orchestration
- Backend persistence sync options
- Observability integrations (cost tracking, traces)
- Voice output + more tools
- Community examples & templates

---

## Non-Functional Requirements (Critical for Marketing Claims)

- **Performance**: Lightweight bundle, fast initial load, efficient streaming
- **Security**: No unintended DOM access, input sanitization, guardrails enforcement
- **Privacy**: Local-first by default, clear data export/deletion paths, EU compliance focus
- **Maintainability**: Clean code, TypeScript, good test coverage, versioned docs
- **Accessibility**: WCAG 2.1+ compliance
- **Browser Support**: Modern browsers + reasonable fallback

---

## Success Metrics

- **Dev Team**: Easy to maintain and extend; low bug rate post-launch
- **Marketing**: Positive feedback on ease of setup, customization, and perceived professionalism
- **Business**: Successful internal deployment at ICS Karafyllis with measurable support ticket reduction
- **Adoption**: Other developers can integrate without hand-holding

---

## Scope Boundaries (What is explicitly OUT for now)

- Full omnichannel (WhatsApp, etc.) – web-focused
- No-code admin UI (dev-focused configuration)
- Heavy backend services (keep it lightweight npm package)
- Production hosting / SaaS version

---

## Team Agreement

- **Marketing** agrees to promote only features that are delivered and tested.
- **Development** commits to transparency on timelines and any trade-offs.
- Changes to this spec require discussion and update to this document.
- MVP target delivery: TBD

This document will be updated as the project evolves.

**Approved by**:  
- Developer: __________________  
- Stakeholder / Boss: __________________

---
*Last Updated: 2026-07-01 — Draft v1, pending review*
