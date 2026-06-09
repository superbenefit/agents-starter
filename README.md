# SuperBenefit Agents Starter

> A maintained fork of [cloudflare/agents-starter](https://github.com/cloudflare/agents-starter), customized with [Think](https://developers.cloudflare.com/agents/harnesses/think/), Vectorize-backed searchable context, and R2-backed loadable context (Skills).

<a href="https://deploy.workers.cloudflare.com/?url=https://github.com/superbenefit/agents-starter"><img src="https://deploy.workers.cloudflare.com/button" alt="Deploy to Cloudflare"/></a>

## What's different from upstream

This fork extends the official `agents-starter` with the SuperBenefit stack pattern:

| Addition                       | What it provides                                                                                                                                          |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`@cloudflare/think`**        | Opinionated agent harness — minimal subclass (`getModel()` only), tree-structured message history, context blocks, FTS5 search, non-destructive branching |
| **Vectorize binding**          | Searchable context via a custom `search()` provider backed by Workers Vectorize                                                                           |
| **R2 binding**                 | Loadable context (Skills) via `R2SkillProvider` — large reference documents loaded on demand                                                              |
| **`experimental` compat flag** | Required by Think — enables the runtime features Think depends on                                                                                         |

The agent class is `AdvisorAgent extends Think<Env>` — a 3-line subclass that handles the WebSocket protocol, persistence, and the agentic loop automatically.

## Quick start

```bash
npx create-cloudflare@latest my-project --template superbenefit/agents-starter
cd my-project
npm install
```

### Set up infrastructure

```bash
# Create Vectorize index for searchable context (once per project)
npx wrangler vectorize create slc-knowledge --dimensions=768 --metric=cosine

# Create R2 bucket for Skills / loadable context (once per project)
npx wrangler r2 bucket create slc-content
```

### Run locally

```bash
npm run dev
# → http://localhost:5173
```

## Project structure

```
src/
  server.ts    # AdvisorAgent extends Think<Env> — 3-line subclass
  app.tsx      # Chat UI built with Kumo components
  client.tsx   # React entry point
  styles.css   # Tailwind + Kumo styles
```

## Making it your own

### Pick a model

Change the model string in `getModel()` inside `src/server.ts`:

```ts
getModel() {
  return createWorkersAI({ binding: this.env.AI })("your-model-here");
}
```

See the [Workers AI models catalog](https://developers.cloudflare.com/workers-ai/models/) for available models.

### Configure the system prompt

Override `getSystemPrompt()` or `configureSession()` in `AdvisorAgent`:

```ts
getSystemPrompt() {
  return "You are a helpful assistant...";
}

// Or use Sessions for dynamic, LLM-writable context:
configureSession(session: Session) {
  session
    .withContext("personality", {
      provider: {
        get: async () => "You are a helpful advisor...",
      },
    });
  return session;
}
```

See [Conversation state and memory](https://developers.cloudflare.com/agents/concepts/conversation-state-and-memory/) and the [Sessions API reference](https://developers.cloudflare.com/agents/runtime/lifecycle/sessions/).

### Add tools

Use the AI SDK v6 `tool()` function (same pattern as upstream):

```ts
import { tool } from "ai";
import { z } from "zod";

getTools() {
  return {
    // Server-side — auto-execute
    myTool: tool({
      description: "...",
      inputSchema: z.object({ /* ... */ }),
      execute: async (input) => { /* return result */ }
    }),

    // Client-side — browser provides the result via onToolCall
    browserTool: tool({
      description: "...",
      inputSchema: z.object({ /* ... */ })
    }),
  };
}
```

### Add state beyond chat

Use `this.setState()` and `this.state` for real-time state synced to all connected clients. See [Store and sync state](https://developers.cloudflare.com/agents/api-reference/store-and-sync-state/).

### Use a different AI provider

Replace `createWorkersAI` in `getModel()` with any AI SDK v6 provider:

```bash
npm install @ai-sdk/openai
```

```ts
import { openai } from "@ai-sdk/openai";
getModel() { return openai("gpt-5.2"); }
```

See [Think API reference](https://developers.cloudflare.com/agents/harnesses/think/) for all overridable hooks.

## Deploy

```bash
npm run deploy
```

## Keeping the fork in sync with upstream

This fork tracks the official `cloudflare/agents-starter` repo. To pull in upstream changes:

```bash
# Fetch upstream
git fetch upstream

# Review what's changed
git log main..upstream/main --oneline

# Merge upstream into main
git merge upstream/main

# Resolve conflicts, then push
git push origin main
```

**Likely conflict points:**

| File             | Strategy                                                                                               |
| ---------------- | ------------------------------------------------------------------------------------------------------ |
| `wrangler.jsonc` | Upstream may update `compatibility_date` — keep the fork's flags and bindings, take upstream's date    |
| `src/server.ts`  | Upstream changes to the AIChatAgent example are reference-only — the fork uses Think                   |
| `package.json`   | Take upstream dependency updates, keep the fork's additions (`@cloudflare/think`, `@cloudflare/shell`) |

## Learn more

- [Agents SDK documentation](https://developers.cloudflare.com/agents/)
- [Think API reference](https://developers.cloudflare.com/agents/harnesses/think/)
- [Sessions API](https://developers.cloudflare.com/agents/runtime/lifecycle/sessions/)
- [Vectorize get started](https://developers.cloudflare.com/vectorize/get-started/intro/)
- [Workers AI models](https://developers.cloudflare.com/workers-ai/models/)

## License

MIT — same as upstream.
