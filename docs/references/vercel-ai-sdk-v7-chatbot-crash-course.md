# Vercel AI SDK v7 Crash Course: Super Simple Chatbot Component in Next.js

**AI SDK v7** (latest) is a unified TypeScript toolkit for building AI apps. It includes:

- **AI SDK Core** (`ai`): `streamText`, providers, tools, streaming.
- **AI SDK UI** (`@ai-sdk/react`): `useChat` hook for React/Next.js.

Perfect for Next.js App Router.

## 1. Project Setup

```bash
pnpm create next-app@latest my-chatbot --typescript --tailwind --eslint --app
cd my-chatbot
pnpm add ai @ai-sdk/react
# Optional specific provider:
# pnpm add @ai-sdk/openai
```

Add API keys to `.env.local`:

```env
AI_GATEWAY_API_KEY=your_key
# or OPENAI_API_KEY=...
```

## 2. Backend Route (`app/api/chat/route.ts`)

```ts
import {
  streamText,
  UIMessage,
  convertToModelMessages,
  createUIMessageStreamResponse,
  toUIMessageStream,
} from 'ai';

export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: 'xai/grok-build-0.1', // swap as needed: 'openai/gpt-4o', etc.
    instructions: 'You are a helpful, concise assistant.',
    messages: await convertToModelMessages(messages),
  });

  return createUIMessageStreamResponse({
    stream: toUIMessageStream({ stream: result.stream }),
  });
}
```

## 3. Frontend Chat Component (`app/page.tsx`)

```tsx
'use client';

import { useChat } from '@ai-sdk/react';
import { useState } from 'react';

export default function Chatbot() {
  const [input, setInput] = useState('');
  const { messages, sendMessage, status, stop, error } = useChat();

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (input.trim()) {
      sendMessage({ text: input });
      setInput('');
    }
  };

  return (
    <div className="max-w-2xl mx-auto p-4 h-screen flex flex-col">
      <div className="flex-1 overflow-y-auto space-y-4 mb-4">
        {messages.map((message) => (
          <div
            key={message.id}
            className={`flex ${message.role === 'user' ? 'justify-end' : 'justify-start'}`}
          >
            <div
              className={`max-w-[80%] p-3 rounded-lg ${
                message.role === 'user'
                  ? 'bg-blue-600 text-white'
                  : 'bg-gray-200 dark:bg-gray-800'
              }`}
            >
              {message.parts.map((part, i) => 
                part.type === 'text' ? <div key={i}>{part.text}</div> : null
              )}
            </div>
          </div>
        ))}

        {(status === 'submitted' || status === 'streaming') && (
          <div className="text-center text-sm text-gray-500">AI is thinking...</div>
        )}
      </div>

      {error && <div className="text-red-500 mb-2">Error: Something went wrong.</div>}

      <form onSubmit={handleSubmit} className="flex gap-2">
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          disabled={status !== 'ready'}
          placeholder="Type your message..."
          className="flex-1 p-3 border rounded-lg focus:outline-none focus:ring-2"
        />
        <button
          type="submit"
          disabled={status !== 'ready' || !input.trim()}
          className="px-6 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50"
        >
          Send
        </button>
        {(status === 'submitted' || status === 'streaming') && (
          <button
            type="button"
            onClick={stop}
            className="px-4 bg-red-500 text-white rounded-lg"
          >
            Stop
          </button>
        )}
      </form>
    </div>
  );
}
```

## 4. Run It

```bash
pnpm run dev
```

## Key v7 Concepts

- **Streaming**: Real-time via `useChat` + `streamText`.
- **Messages**: Use `parts[]` (text, tools, etc.).
- **Status**: `'ready' | 'submitted' | 'streaming' | 'error'`.
- **Extensibility**: Add tools, persistence (`chatId`), multi-modal, etc.
- **Providers**: Easy swap (Gateway, OpenAI, Anthropic, etc.).

## Next Steps

- Tools / agents: See official chatbot-tool-usage docs.
- Persistence: Vercel KV or DB.
- Polish: Markdown, scroll, error retry (`regenerate`).

Full docs: [https://ai-sdk.dev](https://ai-sdk.dev/docs)

---
*Generated from Vercel AI SDK v7 documentation.*
