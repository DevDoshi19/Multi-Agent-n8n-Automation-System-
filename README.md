# Multi-Agent n8n Automation System

> 🚧 **Still building this** — core system works, but I'm actively improving agent logic, adding more data sources, and cleaning up the monitoring layer. Don't be surprised if something looks half-done.

---

## What is this?

I built this because I wanted to see if I could make a system where different AI agents actually *do different jobs* — not just one LLM trying to do everything.

The idea is simple: when a user sends a message, the system figures out what kind of question it is, sends it to the right agent, and returns a clean answer. Under the hood there's also a full observability layer logging every request, error, and response time to Google Sheets.

It's not a toy project — it has guardrails, error handling, latency tracking, and real financial data retrieval. But it's also not "production" in the corporate sense. It's production in the *I built this myself and it actually works* sense.

---

## How it works

When a message comes in, it goes through this flow:

```
User Message
    ↓
Guardrails Check  ← blocks harmful/irrelevant input
    ↓
Request Logger    ← logs to Google Sheets immediately
    ↓
Orchestrator Agent (GPT-4.1)
    ↓         ↓          ↓
Research    Logical    Explainer
 Agent       Agent      Agent
(Groq 8b)  (Qwen 32b) (Groq 70b)
    ↓
Latency Calculator
    ↓
Performance Logger + Response Logger
    ↓
Final Answer to User
```

Each agent has one job:

- **Research Agent** — pulls data from the connected Google Sheet (financial/company data). Doesn't reason, just finds.
- **Logical Agent** — takes what Research found and thinks through it. Structured, analytical, no fluff.
- **Explainer Agent** — takes the logical output and makes it human-readable. Simple language, clear structure.
- **Orchestrator** — manages the whole thing. Decides which agents to call, in what order, and compiles the final response.

---

## The Observability Layer

This part took longer than the agents themselves.

Every request gets:
- A unique `request_id` (timestamp + session id)
- Logged on arrival with status `received`
- Updated after completion with actual `latency_ms`

Errors go to a separate `Errors_Log` sheet with the full stack trace (truncated to 200 chars), the node that failed, and the workflow name.

Performance metrics (execution time per agent run) go into a `Performance_Log` sheet.

So at any point I can open Google Sheets and see exactly what's happening — which queries are slow, which ones are failing, and where.

---

## Tech Stack

| Layer | Tool |
|-------|------|
| Workflow engine | n8n |
| Orchestrator LLM | GPT-4.1 (via Thesys API) |
| Research Agent LLM | Groq — Llama 3.1 8b |
| Logical Agent LLM | Groq — Qwen3 32b |
| Explainer Agent LLM | Groq — Llama 3.3 70b |
| Data source | Google Sheets |
| Observability | Google Sheets (Requests, Performance, Errors tabs) |
| Safety | n8n Guardrails node |

Different models for different agents on purpose — faster/cheaper models for simpler jobs (research retrieval), stronger models for reasoning.

---

## What's actually working

- [x] Multi-agent routing based on intent
- [x] Guardrails blocking harmful input
- [x] Request logging on arrival
- [x] Latency tracking per request
- [x] Error logging with stack trace
- [x] Financial data retrieval from Google Sheets
- [x] Chat UI via n8n webhook

## What I'm still working on

- [ ] Evaluation scoring (auto-grade agent responses for quality)
- [ ] Better intent classification — right now orchestrator decides, want a dedicated classifier
- [ ] More data sources beyond one Google Sheet
- [ ] Moving observability from Sheets to a proper DB eventually

---

## How to run it

1. Import both JSON workflow files into your n8n instance
2. Set up Google Sheets credentials (OAuth2)
3. Connect your own Groq API key and OpenAI/Thesys API key
4. Point the Research Agent to your own data sheet
5. Activate both workflows — the error logger runs separately as a sub-workflow

> The error logging workflow (`error_trigger.json`) needs to be set as the **Error Workflow** in your main workflow settings inside n8n.

---

## Why I built this

I was learning how to build serious AI systems — not just prompt-and-respond chatbots, but something with actual architecture. Multi-agent coordination, observability, safety layers — things that matter when you're building something real.

This is that attempt. Still evolving.

---

*Built by [Dev Doshi](https://github.com/DevDoshi19)*
