# LangChain Rejectionism Lab

A hands-on, Vercel-hosted route to learning LangChain. Build one small capability at a time; each milestone should leave the app deployable.

## How to use this lab

1. Complete the goal and acceptance checks for one milestone.
2. Use the implementation prompt as a starting brief for an AI coding assistant—review and adapt its output.
3. Commit when the checks pass; deploy the branch to Vercel.
4. Record what surprised you in the milestone notes before continuing.

**Recommended stack:** Next.js (App Router), TypeScript, LangChain.js, Vercel AI Gateway, and Vercel AI SDK for the streaming UI. Keep model calls in server-side route handlers; never expose provider credentials to the browser.

## Milestone 0 — Foundations

**Goal:** Understand the moving parts and make the empty app deploy.

- Create a Next.js App Router app with TypeScript.
- Deploy it to Vercel from this repository.
- Add a short `docs/decisions.md`: model provider, what data may be sent to it, and a no-secrets-in-client rule.

**Prompt:**

> Set up a minimal Next.js App Router TypeScript application for a LangChain learning lab. Include a home page explaining the lab, preserve the existing README, and do not add API keys or call a model yet. Keep dependencies minimal and make the project deployable on Vercel.

**Done when:** a production deployment loads and the repository documents its model/data boundary.

## Milestone 1 — One model call

**Goal:** Call a chat model from a server route and display a response.

- Configure `ChatOpenAI` through Vercel AI Gateway.
- Add a route handler accepting a single prompt.
- Validate input and return friendly errors without exposing provider details.

**Prompt:**

> Add a server-only Next.js route that accepts `{ prompt: string }`, validates it, then calls a LangChain `ChatOpenAI` model configured for Vercel AI Gateway. Add a simple form on the home page to submit a prompt and render the response. Keep credentials server-side and document the required environment variable names without printing their values.

**Done when:** a user can submit a prompt, receive text, and see a safe validation error for empty input.

## Milestone 2 — Streaming chat

**Goal:** Learn the chat message model and token streaming.

- Replace the one-shot UI with a multi-turn conversation.
- Use the AI SDK LangChain adapter to stream LangChain output to the client.
- Support cancellation from the browser and pass the request abort signal to the server-side model call.

**Prompt:**

> Convert the single-prompt prototype into a streaming multi-turn chat. Use `@ai-sdk/langchain` to adapt a LangChain stream for an AI SDK chat UI. Preserve message roles, validate payloads, and wire client cancellation to the route request signal. Use the Node.js runtime and keep the model call in the route handler.

**Done when:** text appears incrementally, follow-up questions include prior context, and Stop halts the request.

## Milestone 3 — Structured output and tools

**Goal:** Make the model produce reliable data and use a narrowly scoped tool.

- Define a Zod schema for a `RejectionAnalysis` response: summary, assumptions, risks, and questions.
- Add one deterministic local tool, such as counting words or checking whether a quoted claim appears in supplied text.
- Render tool progress and the structured result separately from normal chat text.

**Prompt:**

> Add a structured `RejectionAnalysis` workflow using a Zod schema with `summary`, `assumptions`, `risks`, and `questions`. Add one side-effect-free local tool that works only on text supplied by the user. Show tool activity in the UI, validate tool input, and never let the model choose arbitrary code or network access.

**Done when:** malformed structured output is handled safely and the tool cannot access data outside the request.

## Milestone 4 — Retrieval-augmented generation (RAG)

**Goal:** Answer from a small, cited knowledge base rather than improvising.

- Start with a handful of Markdown documents you own.
- Split, embed, and store chunks in a vector database selected through a Vercel Marketplace integration.
- Retrieve the top relevant chunks, include source labels, and instruct the model to say when evidence is insufficient.

**Prompt:**

> Add a small RAG pipeline for Markdown documents owned by this project. Implement ingestion separately from the chat route, retrieve a bounded number of relevant chunks, and make answers cite their source document and section. If no source supports the answer, return an explicit insufficient-evidence response rather than guessing.

**Done when:** each knowledge-grounded answer shows sources, and an unsupported question is declined with an explanation.

## Milestone 5 — Agent workflow with LangGraph

**Goal:** Understand controlled multi-step execution.

- Model the workflow as explicit graph nodes: classify request, retrieve evidence, draft, review, finalize.
- Persist only the state you need; define a maximum step count and a human-approval branch for consequential actions.
- Stream graph events to the UI.

**Prompt:**

> Implement a LangGraph workflow with explicit nodes for request classification, retrieval, drafting, review, and finalization. Set a strict recursion/step limit. The graph must require human approval before any future side effect, and it must emit progress events suitable for the existing chat UI. Keep the first version read-only.

**Done when:** you can trace which nodes ran, why the graph stopped, and no path can perform a side effect.

## Milestone 6 — Evaluation and observability

**Goal:** Measure quality and diagnose production behavior.

- Create a small evaluation set: 10 representative prompts with expected properties, not necessarily exact wording.
- Track latency, failures, retrieval quality, tool calls, and user cancellation.
- Add feedback controls and capture no sensitive prompt content without an explicit retention decision.

**Prompt:**

> Add a lightweight evaluation suite with ten representative prompts and assertions about response properties, citations, and refusal behavior. Add server-side structured logging for latency, failures, retrieval count, and tool usage. Avoid logging secrets or full user content by default. Document how to run the evaluation locally and how to interpret failures.

**Done when:** regressions are detectable before deployment and production failures are diagnosable from safe telemetry.

## Milestone 7 — Production hardening

**Goal:** Make the app safe to share.

- Add authentication before exposing private data or costly actions.
- Rate-limit requests and set per-user/model cost boundaries.
- Add prompt-injection defenses: isolate retrieved content, constrain tools, and treat tool output as untrusted.
- Test failure modes: model timeout, malformed tool arguments, missing retrieval results, and cancellation.

**Prompt:**

> Harden this LangChain application for a limited beta. Add authentication-aware authorization boundaries, request rate limiting, sensible input and output limits, and safe failure behavior. Treat retrieved content and tool output as untrusted. Add tests for model failure, tool validation failure, no retrieval results, and user cancellation. Do not add destructive tools.

**Done when:** the app fails closed around protected data and has clear, user-safe error states.

## Reference shelf

- [LangChain with Vercel AI Gateway](https://vercel.com/docs/ai-gateway/ecosystem/framework-integrations/langchain)
- [AI SDK LangChain adapter](https://ai-sdk.dev/providers/adapters/langchain)
- [Deploying AI apps on Vercel](https://ai-sdk.dev/docs/advanced/vercel-deployment-guide)
- [LangChain.js documentation](https://js.langchain.com/docs/introduction/)
- [LangGraph.js documentation](https://langchain-ai.github.io/langgraphjs/)
- [Vercel AI Gateway documentation](https://vercel.com/docs/ai-gateway)
- [Vercel AI templates](https://vercel.com/templates/ai)

## Practical rules

- Start with one model call before adding agents or RAG.
- Use real evaluation prompts before optimizing a demo.
- Prefer explicit, typed tools over open-ended agent autonomy.
- Keep secrets in Vercel environment variables; never in browser code or commits.
- Deploy each milestone to a preview URL before merging it.
