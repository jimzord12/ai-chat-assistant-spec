# AI Frontend Bot Cookbook v2

## Building a Browser-Aware AI Assistant with assistant-ui + Vercel AI SDK

> **Difficulty:** Intermediate  
> **Stack:** Next.js 15+, React, TypeScript, assistant-ui, Vercel AI SDK v6+  
> **Prerequisites:** React hooks, Next.js App Router, Zod, basic AI tool calling

---

## Table of Contents

1. [Overview](#1-overview)
2. [Architecture](#2-architecture)
3. [Installation & Setup](#3-installation--setup)
4. [Defining a Browser Control Toolkit](#4-defining-a-browser-control-toolkit)
5. [Registering Tools on the Client](#5-registering-tools-on-the-client)
6. [Exposing Tools to the Model on the Server](#6-exposing-tools-to-the-model-on-the-server)
7. [Tool Reference: Capabilities & Recipes](#7-tool-reference-capabilities--recipes)
   - 7.1 [Navigate](#71-navigate)
   - 7.2 [Write to an Input](#72-write-to-an-input)
   - 7.3 [Click Element](#73-click-an-element)
   - 7.4 [Highlight Element](#74-highlight-an-element)
8. [Selector Strategy: `data-ai-target`](#8-selector-strategy-data-ai-target)
9. [Human-in-the-Loop & Approvals](#9-human-in-the-loop--approvals)
10. [State-Dependent Tools (`stubTool`)](#10-state-dependent-tools-with-stubtool)
11. [Multi-Step Tool Chains](#11-multi-step-tool-chains)
12. [Key Implementation Notes & Enhancements](#12-key-implementation-notes--enhancements)
13. [Security & Best Practices](#13-security--best-practices)
14. [Troubleshooting](#14-troubleshooting)

---

## 1. Overview

This v2 cookbook shows how to build a **frontend browser agent** that can navigate, fill forms, click elements, and highlight UI in real time inside your Next.js app.

**Use cases:**
- Guided tours / onboarding
- Form autofill & submission
- Visual help ("show me where X is")
- Internal tools & copilots

---

## 2. Architecture

The assistant operates as a frontend bot: the model decides which tool to call → the `execute` function runs **in the browser** (DOM access) → result flows back into the conversation.

**Key principle:** `"use generative"` files are split at build time. The LLM only sees tool *schemas*. Executors stay client-side.

---

## 3. Installation & Setup

### 3.1 Dependencies

```bash
npm install @assistant-ui/react @assistant-ui/react-ai-sdk ai @ai-sdk/openai zod
npm install @assistant-ui/next --save-dev
```

### 3.2 Next.js Config

```ts
// next.config.ts
import { withAui } from "@assistant-ui/next";

export default withAui({
  // your existing config
});
```

### 3.3 Project Structure

```
app/
├── api/
│   └── chat/
│       └── route.ts
├── tools/
│   └── browser-control.ts          # "use generative"
├── providers.tsx                   # Runtime provider
└── page.tsx
```

---

## 4. Defining a Browser Control Toolkit

```ts
// app/tools/browser-control.ts
"use generative";

import { defineToolkit, humanTool, stubTool } from "@assistant-ui/react";
import { z } from "zod";

export default defineToolkit({
  // Tool definitions (see section 7)
});
```

---

## 5. Registering Tools on the Client

```tsx
// app/providers.tsx
"use client";

import {
  AssistantRuntimeProvider,
  Tools,
  useAui,
} from "@assistant-ui/react";
import { useChatRuntime } from "@assistant-ui/react-ai-sdk";
import browserControlTools from "./tools/browser-control";

export function AIProvider({ children }: { children: React.ReactNode }) {
  const runtime = useChatRuntime({ api: "/api/chat" });

  useAui({
    tools: Tools({ toolkit: browserControlTools }),
  });

  return (
    <AssistantRuntimeProvider runtime={runtime}>
      {children}
    </AssistantRuntimeProvider>
  );
}
```

**Layout:**

```tsx
// app/layout.tsx
import { AIProvider } from "./providers";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body><AIProvider>{children}</AIProvider></body>
    </html>
  );
}
```

---

## 6. Exposing Tools to the Model on the Server

```ts
// app/api/chat/route.ts
import { AISDKToolkit } from "@assistant-ui/react-ai-sdk";
import { streamText, convertToModelMessages, stepCountIs } from "ai";
import { openai } from "@ai-sdk/openai";
import browserControlTools from "@/tools/browser-control";

const aiToolkit = new AISDKToolkit({ toolkit: browserControlTools });

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: openai("gpt-4o"),
    messages: await convertToModelMessages(messages),
    tools: await aiToolkit.tools({ frontend: true }),
    stopWhen: stepCountIs(12),
  });

  return result.toUIMessageStreamResponse();
}
```

---

## 7. Tool Reference: Capabilities & Recipes

### 7.1 Navigate

```ts
navigate: {
  description: "Navigate the user to a different page/route within the app",
  parameters: z.object({
    path: z.string().describe("The route path, e.g. /settings"),
  }),
  execute: async ({ path }) => {
    "use client";
    window.location.assign(path);
    return { success: true, navigatedTo: path };
  },
  render: ({ args, status }) => {
    if (status.type === "running") return <div>Navigating to {args.path}…</div>;
    return <div>✅ Navigated to {args.path}</div>;
  },
},
```

**Tip:** For SPA navigation, use `stubTool` + `useRouter`.

### 7.2 Write to an Input

```ts
write_to_input: {
  description: "Write text into an input/textarea/contenteditable",
  parameters: z.object({
    selector: z.string().describe("CSS selector"),
    value: z.string().describe("Text to write"),
    clear: z.boolean().optional().default(true),
  }),
  execute: async ({ selector, value, clear }) => {
    "use client";
    const el = document.querySelector(selector) as HTMLInputElement | HTMLTextAreaElement | null;
    if (!el) throw new Error(`Element not found: ${selector}`);

    const proto = el instanceof HTMLTextAreaElement
      ? HTMLTextAreaElement.prototype
      : HTMLInputElement.prototype;
    const setter = Object.getOwnPropertyDescriptor(proto, "value")?.set;

    if (clear) {
      setter?.call(el, "");
      el.dispatchEvent(new Event("input", { bubbles: true }));
    }

    setter?.call(el, value);
    el.dispatchEvent(new Event("input", { bubbles: true }));
    el.dispatchEvent(new Event("change", { bubbles: true }));
    el.focus();

    return { success: true, selector, value };
  },
  render: ({ args, status }) => {
    if (status.type === "running") return <div>Typing into {args.selector}…</div>;
    return <div>✅ Wrote into {args.selector}</div>;
  },
},
```

### 7.3 Click an Element

```ts
click_element: {
  description: "Click a button/link/element",
  parameters: z.object({
    selector: z.string().describe("CSS selector"),
  }),
  execute: async ({ selector }) => {
    "use client";
    const el = document.querySelector<HTMLElement>(selector);
    if (!el) throw new Error(`Element not found: ${selector}`);

    el.scrollIntoView({ behavior: "smooth", block: "center" });
    await new Promise(r => setTimeout(r, 150));
    el.click();

    return { success: true, selector };
  },
  render: ({ args, status }) => status.type === "running"
    ? <div>Clicking {args.selector}…</div>
    : <div>✅ Clicked {args.selector}</div>,
},
```

### 7.4 Highlight an Element

```ts
highlight_element: {
  description: "Visually highlight an element on the page to draw the user's attention",
  parameters: z.object({
    selector: z.string().describe("CSS selector of the element to highlight"),
    durationMs: z.number().optional().default(2500).describe("How long to show the highlight in ms"),
  }),
  execute: async ({ selector, durationMs }) => {
    "use client";
    const el = document.querySelector<HTMLElement>(selector);
    if (!el) throw new Error(`Element not found: ${selector}`);

    const overlay = document.createElement("div");
    const rect = el.getBoundingClientRect();
    Object.assign(overlay.style, {
      position: "fixed",
      top: `${rect.top - 4}px`,
      left: `${rect.left - 4}px`,
      width: `${rect.width + 8}px`,
      height: `${rect.height + 8}px`,
      border: "2px solid #3b82f6",
      borderRadius: "6px",
      backgroundColor: "rgba(59,130,246,0.15)",
      zIndex: "9999",
      pointerEvents: "none",
      transition: "opacity 0.3s ease",
    });
    document.body.appendChild(overlay);
    el.scrollIntoView({ behavior: "smooth", block: "center" });

    await new Promise(r => setTimeout(r, durationMs));
    overlay.style.opacity = "0";
    await new Promise(r => setTimeout(r, 300));
    overlay.remove();

    return { success: true, selector };
  },
  render: ({ args, status }) => status.type === "running"
    ? <div>Highlighting {args.selector}…</div>
    : <div>✅ Highlighted {args.selector}</div>,
},
```

---

## 8. Selector Strategy: `data-ai-target`

Annotate key elements with a stable, semantic attribute:

```tsx
<button data-ai-target="submit-order">Submit Order</button>
<input data-ai-target="email-field" type="email" />
```

Preferred selector pattern: `[data-ai-target="submit-order"]`

**Why not class or id?** Classes change with refactors and styling updates. `data-ai-target` is an explicit, stable contract between the UI and the agent.

---

## 9. Human-in-the-Loop & Approvals

Wrap sensitive tools in `humanTool()` to require user confirmation before execution:

```ts
import { humanTool } from "@assistant-ui/react";

submit_form: humanTool({
  description: "Submit the current form",
  parameters: z.object({ formSelector: z.string() }),
  render: ({ args, onApprove, onReject }) => (
    <div>
      <p>Submit form <code>{args.formSelector}</code>?</p>
      <button onClick={onApprove}>Yes</button>
      <button onClick={onReject}>No</button>
    </div>
  ),
  execute: async ({ formSelector }) => {
    "use client";
    const form = document.querySelector<HTMLFormElement>(formSelector);
    form?.requestSubmit();
    return { success: true };
  },
}),
```

Alternatively, use the AI SDK `experimental_prepareToolCall` + `needsApproval` pattern for server-side gating.

---

## 10. State-Dependent Tools with `stubTool`

Use `stubTool` to register tools that need React state (router, form context, zustand store) — the executor runs inside a component and has full hook access:

```ts
import { stubTool } from "@assistant-ui/react";
import { useRouter } from "next/navigation";

function NavigationToolProvider() {
  const router = useRouter();

  useAui({
    tools: {
      navigate_spa: stubTool({
        description: "Navigate using Next.js router (no full reload)",
        parameters: z.object({ path: z.string() }),
        execute: async ({ path }) => {
          router.push(path);
          return { success: true, navigatedTo: path };
        },
      }),
    },
  });

  return null;
}
```

---

## 11. Multi-Step Tool Chains

Chain tools sequentially for complex workflows:

```
navigate("/checkout") → write_to_input("#email", "user@example.com") → click_element("[data-ai-target='submit']")
```

**Server config:**
```ts
stopWhen: stepCountIs(12),  // cap agentic loops
```

**Tips:**
- Keep each tool atomic and idempotent where possible
- Return rich result objects so the model can self-correct on failure
- Add a `wait` tool (`setTimeout` wrapper) for animations/transitions between steps

---

## 12. Key Implementation Notes & Enhancements

- **Always use `data-ai-target`** over fragile class/id selectors
- **Dispatch both `input` and `change` events** after writing to inputs (React synthetic event compatibility)
- **Use native prototype setters** to bypass React's controlled component value guards
- **Scroll into view before clicking** to avoid off-screen element failures
- **Always clean up highlight overlays** on unmount or after duration
- **Return structured results** from every `execute` — the model uses them for self-correction
- For **contenteditable** elements, use `execCommand('insertText')` or `InputEvent` instead of value setter

---

## 13. Security & Best Practices

- Never expose destructive tools (delete, payment, auth) without `humanTool()` approval gates
- Validate all `selector` parameters — reject selectors targeting sensitive elements
- Scope `data-ai-target` to safe, non-sensitive UI only
- Rate-limit tool invocations per session server-side
- Log all tool calls for audit trails
- Never leak server-side secrets in tool `render` output (it's client-rendered)
- Use `stopWhen: stepCountIs(N)` to cap agentic loops and prevent runaway execution

---

## 14. Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| Input not updating | React controlled component blocking setter | Use native prototype setter + dispatch `input`/`change` events |
| Element not found | Selector too fragile or element not mounted | Switch to `[data-ai-target="..."]`; add wait step |
| Tool not visible to model | Missing server-side registration | Add to `aiToolkit.tools({ frontend: true })` |
| Infinite tool loop | No stop condition | Add `stopWhen: stepCountIs(N)` |
| Click has no effect | Element off-screen | Add `scrollIntoView` before click |
| SPA navigation not working | `window.location.assign` causes full reload | Use `stubTool` + `useRouter` instead |
| Highlight flickers | Rect computed before scroll | Recompute rect after `scrollIntoView` resolves |
