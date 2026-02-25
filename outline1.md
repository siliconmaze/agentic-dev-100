Agentic Dev 100 outline rewritten cleanly, with:

✅ Repo folder structure

✅ Lab-by-lab objectives

✅ Clear completion criteria

✅ No external course references

✅ Designed for VS Code + Jupyter workflow



---

Agentic Dev 100

Building Production-Ready Agentic Systems with ADK, LangGraph, and AutoGen

Environment:

VS Code

Jupyter Notebooks

Python virtual environment

.env for API keys

Optional: Docker for packaging



---

Repository Structure

agentic-dev-100/
│
├── 00-setup/
│   └── vscode-jupyter-setup.ipynb
│
├── 01-adk-basics/
│   ├── adk-hello-agent.ipynb
│   └── adk-structured-output.ipynb
│
├── 02-react-tools/
│   ├── react-tool-agent.ipynb
│   └── safe-actions.ipynb
│
├── 03-langgraph/
│   └── langgraph-deliberative.ipynb
│
├── 04-memory/
│   └── memory-short-vs-long.ipynb
│
├── 05-observability/
│   └── observability-events-traces.ipynb
│
├── 06-autogen/
│   └── autogen-round-robin.ipynb
│
├── 07-workflows/
│   └── workflow-agent.ipynb
│
├── 08-security/
│   └── security-oversight.ipynb
│
├── 09-eval/
│   └── eval-harness.ipynb
│
├── 10-deploy/
│   └── package-and-run.ipynb
│
├── 11-capstone/
│   └── capstone-agentic-app.ipynb
│
├── requirements.txt
├── .env.example
└── README.md


---

Labs Overview


---

LAB 1 — Environment Setup (VS Code + Jupyter)

Notebook: 00-setup/vscode-jupyter-setup.ipynb

Objective

Set up a reproducible development environment for agentic systems.

You Will:

Create a Python virtual environment

Install ADK, LangGraph, AutoGen

Configure .env for API keys

Run your first test cell successfully


Completion Criteria

Kernel runs successfully

Environment variables load correctly

A simple API test call executes



---

LAB 2 — Creating a Simple ADK Agent

Notebook: 01-adk-basics/adk-hello-agent.ipynb

Objective

Build a minimal working agent.

You Will:

Define an agent configuration

Run an agent loop

Stream output


Completion Criteria

Agent responds to multiple prompts

Agent configuration is modular and reusable



---

LAB 3 — Structured Output with ADK

Notebook: 01-adk-basics/adk-structured-output.ipynb

Objective

Force reliable JSON output.

You Will:

Define schemas (Pydantic/JSON schema)

Validate outputs

Handle invalid responses


Completion Criteria

Agent returns schema-valid JSON

Failures are caught gracefully



---

LAB 4 — ReAct Tool-Using Agent

Notebook: 02-react-tools/react-tool-agent.ipynb

Objective

Implement reasoning + acting behavior.

You Will:

Define tools (calculator, file reader, stub API)

Implement ReAct loop

Log tool decisions


Completion Criteria

Agent selects correct tool for tasks

Tool calls are logged



---

LAB 5 — Safe Action Tools

Notebook: 02-react-tools/safe-actions.ipynb

Objective

Implement controlled side effects.

You Will:

Build tools with confirmations

Implement dry-run mode

Add execution guardrails


Completion Criteria

Destructive operations require approval

Agent cannot bypass safety layer



---

LAB 6 — LangGraph Deliberative Agent

Notebook: 03-langgraph/langgraph-deliberative.ipynb

Objective

Build a deterministic state machine.

You Will:

Create nodes (Plan → Act → Reflect → Finalize)

Define transitions

Add retry logic


Completion Criteria

Graph executes predictably

Errors trigger retries or alternate paths



---

LAB 7 — Memory Systems

Notebook: 04-memory/memory-short-vs-long.ipynb

Objective

Separate short-term state from persistent memory.

You Will:

Implement in-session memory

Add persistent storage (local DB or vector store)

Retrieve context dynamically


Completion Criteria

Agent remembers across runs

Retrieval improves responses



---

LAB 8 — Observability & Tracing

Notebook: 05-observability/observability-events-traces.ipynb

Objective

Make agents debuggable.

You Will:

Log tool calls

Record intermediate states

Create structured trace output


Completion Criteria

Full execution trace is viewable

Sensitive data is redacted



---

LAB 9 — Multi-Agent Collaboration with AutoGen

Notebook: 06-autogen/autogen-round-robin.ipynb

Objective

Create a coordinated multi-agent system.

You Will:

Define roles (Planner, Builder, Reviewer)

Implement round-robin or managed chat

Add termination rules


Completion Criteria

Agents collaborate toward a goal

Conversation ends deterministically



---

LAB 10 — Workflow Agent

Notebook: 07-workflows/workflow-agent.ipynb

Objective

Build a real-world use case.

Example options:

Customer Support Agent

DevOps Assistant

Data Classification Pipeline


You Will:

Build intake → classify → act → respond pipeline

Add escalation logic


Completion Criteria

Workflow handles multiple cases correctly

Escalations trigger properly



---

LAB 11 — Security & Guardrails

Notebook: 08-security/security-oversight.ipynb

Objective

Add policy enforcement.

You Will:

Implement tool allowlists

Detect prompt injection

Enforce output constraints


Completion Criteria

Agent blocks malicious instructions

Tool access is policy-controlled



---

LAB 12 — Evaluation & Regression Testing

Notebook: 09-eval/eval-harness.ipynb

Objective

Build an evaluation harness.

You Will:

Define golden test cases

Add adversarial prompts

Score responses


Completion Criteria

Test suite runs automatically

Failures are measurable



---

LAB 13 — Packaging & Deployment

Notebook: 10-deploy/package-and-run.ipynb

Objective

Turn notebook agent into a runnable application.

You Will:

Refactor into modules

Create CLI entrypoint

Containerize with Docker


Completion Criteria

Agent runs outside notebook

Container builds successfully



---

LAB 14 — Capstone: Production-Ready Agentic System

Notebook: 11-capstone/capstone-agentic-app.ipynb

Objective

Combine everything into a complete system.

Must Include:

ADK runtime layer

LangGraph workflow orchestration

AutoGen multi-agent collaboration

Memory

Guardrails

Evaluation harness

Deployable package


Deliverables

Working demo

README with architecture diagram

Evaluation results summary



---

Course Progression Philosophy

This course moves from:

1. Single-agent basics


2. Structured reliability


3. Tool usage


4. Deterministic workflows


5. Memory


6. Multi-agent systems


7. Security


8. Evaluation


9. Deployment



By the end, learners understand not just how to build agents — but how to build safe, observable, production-ready agentic systems.


