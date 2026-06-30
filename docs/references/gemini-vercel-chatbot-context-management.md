# Building a Chatbot with Vercel AI SDK v7 + Gemini 3.1 Flash-Lite: Context Management for Large Knowledge Pools (v1 Approach)

## Goal
Avoid full RAG for v1 by leveraging Gemini's massive context window and smart prompt/context engineering.

## Key Advantage: Gemini 3.1 Flash-Lite
- **1M token context window** — Stuff a huge amount of knowledge directly.
- Supports **Context Caching** for static knowledge (big cost/latency savings on repeated use).

## Core Strategies (No Full RAG)

### 1. Direct Knowledge Injection + Context Caching
- Pre-process knowledge into structured markdown/text.
- Cache large static portions via Gemini API (explicit or implicit).
- Include relevant modules via simple heuristics (keywords/topics).

### 2. Conversation Management
- **Sliding Window**: Keep recent messages only.
- **Running Summary**: Periodically summarize history + key knowledge into a compact "memory" block.
- Message structure:
  1. System prompt (instructions + core knowledge).
  2. Persistent summary/memory.
  3. Recent conversation turns.

### 3. Lightweight Modular Retrieval
- Break knowledge into sections.
- Simple router (keywords or tiny LLM call) to select relevant sections per query.

### 4. Agentic Tools (Optional)
- Use Vercel AI SDK v7 agents with tools for:
  - Fetching specific knowledge by key.
  - Summarization.
- Minimal overhead.

## Implementation Sketch (Vercel AI SDK)

```typescript
import { streamText } from 'ai';
import { google } from '@ai-sdk/google';

async function generateResponse(messages: CoreMessage[], knowledgeContext: string) {
  const trimmed = manageContext(messages); // your sliding window + summarization logic

  return streamText({
    model: google('gemini-3.1-flash-lite'),
    messages: [
      {
        role: 'system',
        content: `You are a helpful assistant. Knowledge: ${knowledgeContext}`
      },
      ...trimmed
    ],
  });
}
```

## Best Practices
- Monitor token usage (SDK exposes it).
- Implement proactive compression when nearing limits.
- Use Vercel KV for per-user session state.
- Test with real knowledge volume early.

## When to Add RAG (v2)
- Knowledge too large/dynamic.
- Need high-precision retrieval.
- Hallucination issues on niche facts.

This approach ships fast, stays cheap, and scales surprisingly well thanks to Gemini's context capabilities.

---

**Sources / Further Reading**
- Gemini Context Caching docs
- Vercel AI SDK v7 agent & memory features
- Community patterns for context middleware
