# Jarvis - Architecture Vision

## Purpose

Jarvis is a new project built from scratch, inspired by the lessons learned from Jarvis-Local V1.
The goal is to create a local-first personal assistant that is more modular, more standard, easier to evolve, and better suited to multi-channel usage.

This document is the functional and architectural vision for Jarvis .
It is intended to be self-sufficient so that an agentic coding AI can understand what must be built without needing the full history of prior discussions.

## Product Vision

Jarvis is not just a chat UI connected to a local model.
It is a governed personal assistant platform with:

- a central intelligent orchestrator
- multiple clients and channels
- local-first inference for cost control
- cloud escalation for quality when needed
- a progressive workflow engine for richer capabilities over time
- a persistent memory layer
- a distinct Jarvis persona and voice

The long-term idea is to have one coherent assistant brain that can be reached from multiple interfaces while keeping the same rules, identity, and decision logic.

## Core Principles

- Local-first by default
- Cloud only when it adds real value
- Strong governance over autonomy
- Multi-client architecture from day one
- Standardized tool access through MCP or tools where possible
- n8n as an evolving workflow engine, not the universal entrypoint
- Simplicity over unnecessary infrastructure
- Jarvis persona preserved everywhere

## High-Level Architecture

Jarvis must be built as an API-first backend with a central orchestrator.

Conceptual structure:

```text
Open-WebUI ─┐
Telegram   ─┼──> Jarvis  Orchestrator (Python + FastAPI)
WhatsApp   ─┘             ├── LiteLLM
                          ├── Ollama
                          ├── n8n
                          ├── MCP / Tools
                          ├── SQLite + sqlite-vec memory
                          └── XTTS  TTS
```

## Main Components

### 1. Central Orchestrator

The orchestrator is the core of Jarvis .
It is not a thin proxy.
It is the main product backend.

Required stack:

- Python
- FastAPI

Responsibilities:

- receive requests from all clients
- normalize input across channels
- manage session and user context
- apply security and governance policies
- decide what capabilities are exposed
- decide when to remain local and when to escalate
- coordinate memory access
- coordinate tool access
- call LiteLLM for model routing
- call n8n for complex workflows when needed
- preserve unified behavior across channels

The orchestrator must own the system rules.
Logic must not be spread chaotically across frontend clients, LiteLLM, and n8n.

### 2. LiteLLM

LiteLLM is the selected replacement for the custom router-agent.

Role:

- route requests between local and cloud models according to complexity

Target routing policy:

- SIMPLE -> local
- MEDIUM -> local
- COMPLEX -> cloud
- REASONING -> cloud

In practice:

- local backend = Ollama
- cloud backend = OpenRouter-backed models

LiteLLM is used for model routing, not as the main system brain.

### 3. Ollama

Ollama remains the local LLM runtime.

Role:

- serve low-cost, fast, private, local responses
- handle simple requests
- handle bounded safe tool use where allowed
- preserve the Jarvis persona in local mode

### 4. n8n

n8n is kept, but its role is reframed.

n8n must not be the universal brain or the main user-facing entrypoint.
Instead, it must become the progressive workflow and advanced cognition engine.

Role:

- orchestrate complex workflows
- execute rich multi-step automations
- integrate external systems like email, calendar, and Notion
- support future chaining and composition of workflows
- optionally host more advanced cloud-assisted reasoning paths

Important positioning:

- the orchestrator decides when to escalate to n8n
- n8n executes or coordinates the complex path
- n8n is intended to become more powerful over time because the user wants to keep evolving the intelligence of the system through workflow design

### 5. Memory Layer

Chosen memory architecture:

- SQLite + sqlite-vec

Reasons:

- simpler than a dedicated vector DB service
- embedded and easy to backup
- appropriate for personal assistant scale
- avoids unnecessary infrastructure complexity

Expected properties:

- local embedded memory store
- semantic retrieval plus simple textual lookup if useful
- support for user facts, preferences, and useful context
- no dedicated Qdrant service in 

### 6. TTS Layer

Chosen TTS direction:

- keep XTTS  custom proxy

Reason:

- the custom Jarvis voice and voice cloning are considered real product value

The voice identity is part of the Jarvis experience and should be preserved.

## Client and Channel Strategy

Jarvis  must be designed for multiple clients.

Confirmed channels:

- Open-WebUI
- Telegram
- WhatsApp

Important rule:

- same brain
- same rules
- same policies
- only the connector changes

The first version may start with Open-WebUI, but Open-WebUI is not the final product boundary.
The system must remain frontend-independent.

## Functional Execution Strategy

The system must optimize for cost, speed, and appropriateness.

### Local-first intent

The local model should be used whenever possible for:

- simple requests
- short and direct interactions
- low-risk operations
- bounded read operations
- simple MCP/tool usage
- home automation tasks

### Cloud escalation intent

Cloud models and n8n should be used when the request requires:

- better reasoning
- ambiguity resolution
- longer synthesis
- planning
- prioritization
- writing or rewriting content
- higher confidence for important external actions
- multi-step workflows involving work tools

## Functional Policy Chosen

The agreed functional rule is:

- reading and consultation should remain local when simple and bounded
- modification, redaction, and decision-heavy tasks should go through cloud + n8n

This is the key policy boundary for the system.

## Tool and Permission Boundaries

The assistant must not expose every capability to every model equally.
The local model gets a safe subset.
The more powerful cloud path gets richer capabilities.

### Authorization classes

#### Class A - Home and immediate environment

Examples:

- Home Assistant queries
- read sensor state
- turn lights on/off
- simple home commands

Policy:

- local allowed

#### Class B - Personal tool read access

Examples:

- read calendar
- read emails
- read Notion content

Policy:

- local allowed in read-only mode when the request is well bounded

#### Class C - Personal tool modifications

Examples:

- send an email
- create or modify a calendar event
- create or edit a Notion page

Policy:

- cloud + n8n only

#### Class D - Sensitive system actions

Examples:

- shell commands
- Docker operations
- file deletion
- secrets and credentials
- destructive system tasks

Policy:

- forbidden to local by default

## Special Decision About Home Automation

One explicit decision was made:

- simple real-world actions such as home automation commands may be executed directly by the local model without systematic confirmation

This is a deliberate comfort and cost optimization decision.
It must still be bounded to clearly defined safe action domains.

## MCP and Tooling Philosophy

Tools should use standards whenever possible.

Preferred direction:

- MCP where it is a good fit
- tool/function interfaces where appropriate
- no unnecessary proprietary routing logic if standards already solve the problem

Concrete intention from the discussions:

- local model should be able to use simple MCP-exposed capabilities, especially for home control
- cloud + n8n should own richer tool usage for work-style tasks such as email, calendar, and Notion

## Persona Strategy

Jarvis persona remains a core invariant.

The persona must be preserved across local and cloud paths.
However, the agreed approach is not to reformulate responses with a second LLM call after the fact.

Chosen approach:

- inject Jarvis persona directly through system prompts or equivalent prompt-layer configuration
- local responses and cloud responses should both natively speak in Jarvis style

Expected persona traits:

- premium technical personal assistant feel
- concise by default
- calm, sharp, elegant tone
- consistent identity across channels

## Governance Philosophy

Jarvis  must be a strongly governed assistant, not a fully uncontrolled agent.

This means:

- explicit escalation policies
- explicit permission boundaries
- explicit separation of local vs cloud capabilities
- auditability and predictability matter more than maximum autonomy

The target system should feel well orchestrated and dependable, while still becoming more capable over time.

## Role of n8n Over Time

An important nuance from the discussions:

- the user wants to rely heavily on n8n as the place where the system becomes more powerful over time

This does not contradict the orchestrator-centric architecture.
It means:

- the orchestrator remains the governor and policy owner
- n8n becomes the evolving workflow intelligence layer
- richer multi-workflow chaining is a desired future capability

This is a core strategic decision for .

## Quality and Delivery Strategy

Chosen implementation philosophy:

- build the main flows and system skeleton first
- add automated tests once the first stable shape exists

So the quality strategy is:

- not strict TDD from the first minute
- but definitely not test-free
- the orchestrator must be stabilized first, then verified thoroughly

## What Jarvis  Must Keep From V1

- local-first philosophy
- Jarvis identity and persona
- custom voice through XTTS 
- ability to use a local model for low-cost interactions
- ability to escalate to cloud for more demanding work
- n8n as a meaningful capability layer

## What Jarvis  Must Improve Compared To V1

- replace the custom router with LiteLLM for model routing
- simplify memory architecture
- reduce infrastructure complexity
- reduce unnecessary network hops
- separate concerns more cleanly
- avoid a frontend-dependent architecture
- build the system as a real backend platform, not a UI-centered stack

## What Jarvis  Is Not

Jarvis  is not:

- a simple rewrite of the old router-agent
- a product centered only around Open-WebUI
- a system where n8n does everything
- a fully autonomous uncontrolled agent
- a one-channel chat assistant

## Summary Definition

Jarvis  is a local-first, multi-channel, governed personal assistant platform built around a central intelligent orchestrator.

Its design goals are:

- maximize free and local execution for simple interactions
- selectively escalate to cloud intelligence when complexity justifies it
- use MCP and tools in a standards-friendly way
- use n8n as an evolving advanced workflow engine
- maintain a consistent Jarvis identity across all channels
- keep the architecture modular, auditable, and extensible

## Implementation Orientation For An Agentic Coding AI

If an agentic coding AI starts the project from this document, it should assume the following construction order:

1. Create the new repo for Jarvis 
2. Implement the central orchestrator in Python + FastAPI
3. Integrate LiteLLM for local/cloud routing
4. Connect Ollama as the local backend
5. Define the policy engine for local vs cloud and tool exposure
6. Add SQLite + sqlite-vec memory
7. Integrate n8n as the advanced workflow path
8. Expose a first client through Open-WebUI
9. Preserve Jarvis persona through prompt design
10. Reconnect XTTS  for voice output
11. Add Telegram and WhatsApp connectors later without changing the core brain

## Final Intent

Jarvis  should feel like one coherent assistant system, not a pile of connected services.

The assistant should be:

- cheap when possible
- smart when necessary
- controlled by design
- extensible over time
- consistent in identity
- usable from multiple channels

That is the intended concept of Jarvis .
