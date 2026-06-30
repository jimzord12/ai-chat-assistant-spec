# AGENTS.md — AI Agent Onboarding

> You are working on a **specification book** for an AI Chat Assistant product.
> This file is your entry point. Do NOT start working until you have read the files listed below in order.

---

## Step 1: Understand the Protocol

Read [`SPEC_PROTOCOL.md`](./SPEC_PROTOCOL.md) first.

It defines:
- The **5-stage chapter lifecycle** (CREATED → SCOPED → IN PROGRESS → COMPLETE → REVIEWED → LOCKED)
- The **4 session types** and how to run each one (Cold-Start, Grilling, Review, Propagation)
- The **Next-Action Decision Tree** — always use this to determine what to do next
- End-of-session obligations (you MUST update `SPEC_STATE.md` at the end of every session)

---

## Step 2: Check Current State

Read [`SPEC_STATE.md`](./SPEC_STATE.md) to get the live status of the project.

It tells you:
- Which chapter is currently active and what stage it's in
- How many decisions have been resolved vs. total estimated
- The **Next Recommended Action** (pre-computed for you)
- The full Chapter Status Board with dependency info
- Any blocked items or pending cross-chapter impacts
- The Recent Sessions log

---

## Step 3: Understand the Structure

Read [`MASTER_INDEX.md`](./MASTER_INDEX.md) for the full chapter list, dependency graph, and write order.

Read [`chapter_template.md`](./chapter_template.md) before creating or editing any chapter file. All chapters must follow this template exactly.

Chapter files live in the [`chapters/`](./chapters/) directory.

---

## Step 4: Reference Materials

Third-party SDK guides and external documentation live in [`docs/references/`](./docs/references/).

Check the [`docs/references/README.md`](./docs/references/README.md) index before making any decisions that involve external SDKs or libraries. Currently available:

| File | Topic | Relevant Chapters |
|------|-------|-------------------|
| [`vercel-ai-sdk-v7-chatbot-crash-course.md`](./docs/references/vercel-ai-sdk-v7-chatbot-crash-course.md) | Vercel AI SDK v7 — core patterns | Ch 03, Ch 04 |
| [`gemini-vercel-chatbot-context-management.md`](./docs/references/gemini-vercel-chatbot-context-management.md) | Gemini + Vercel AI SDK v7 — context management (no RAG) | Ch 04, Ch 08 |

---

## Step 5: Context for Claude

If you are Claude, also read [`CLAUDE.md`](./CLAUDE.md) — it contains model-specific behavioral instructions for this project.

---

## Hard Rules

- **Never skip the Cold-Start sequence.** Always read `SPEC_STATE.md` before doing anything.
- **Never duplicate content.** All decisions, contracts, and acceptance criteria live inside the chapter files.
- **Never modify a LOCKED chapter** without following the Unlock Process defined in `SPEC_PROTOCOL.md §1 Stage 5`.
- **Always update `SPEC_STATE.md` at the end of every session.** If it's not updated, the session didn't happen.
- **Follow the Next-Action Decision Tree** in `SPEC_PROTOCOL.md §3` — do not freelance the next step.

---

## Grilling Session Interaction Rules

- **Always ask questions one at a time.** Never present multiple questions at once and wait for a bulk answer.
- Ask Q1, wait for the user's answer, record it, then ask Q2 — and so on.
- This applies to ALL grilling sessions across ALL chapters, without exception.
- The user drives the pace. Do not rush or batch questions to save turns.

---

## Scoping Depth Calibration

The goal is **thorough but not exhausting** — neither a quick 5-question pass nor a relentless 30-question interrogation.

**The right bar:**
- Cover every decision that would meaningfully block or constrain implementation if left ambiguous
- Skip questions whose answers are already obvious from locked chapters or established conventions
- If a chapter is estimated at 8-12 decisions, aim for the **middle of the range** (10), not the floor (8)
- If a chapter is estimated at 15-25 decisions, aim for **~18-20**, not 15
- Prefer **fewer, higher-signal questions** over many shallow ones
- If you find yourself asking something that has only one reasonable answer given prior decisions, resolve it yourself and note it — don't waste a question on it
- If a question is genuinely important but the user seems fatigued, flag it as "deferred" and add it to Open Questions rather than skipping it silently

**Anti-patterns to avoid:**
- Stopping at the minimum estimate just because you have "enough"
- Asking obvious questions that are already resolved by locked chapters
- Batching 3 sub-questions into one to artificially inflate the count
- Skipping infrastructure/tooling questions because they feel "too technical"
