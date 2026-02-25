CKAD-style 2-Day Bootcamp version of Agentic Dev 100, designed to be:

Hands-on

Task-driven

Time-boxed

Minimal theory

“Build under pressure” style

Clear verification steps

Focused on practical competency



---

Agentic Dev 100

2-Day Practical Bootcamp (CKAD Style)

Environment:

VS Code

Jupyter Notebooks

Python venv

.env for API keys

Optional: Docker



---

Bootcamp Structure

Day	Focus	Outcome

Day 1	Core Agent Mechanics	You can build tool-using agents with workflow control
Day 2	Production Readiness	You can secure, evaluate, orchestrate, and package agents



---

Day 1 — Core Agent Engineering


---

Lab 1 — Environment Setup (45 min)

Goal: Get a working agent environment.

Tasks

Create virtual environment

Install ADK, LangGraph, AutoGen

Create .env

Load API key securely

Run a simple test call


Verify

python -m ipykernel works

Notebook runs without errors

Model responds



---

Lab 2 — Build a Minimal ADK Agent (60 min)

Goal: Create a basic functional agent.

Tasks

Define agent configuration

Create a reusable run loop

Stream responses

Handle errors gracefully


Verify

Agent responds consistently

Errors don’t crash kernel



---

Lab 3 — Structured Outputs (45 min)

Goal: Make the agent reliable.

Tasks

Define schema

Enforce JSON output

Validate responses

Retry on failure


Verify

Invalid outputs are caught

Only valid JSON passes



---

Lab 4 — Tool-Using Agent (ReAct Pattern) (90 min)

Goal: Agent chooses and uses tools.

Tasks

Implement calculator tool

Implement file read tool

Build ReAct loop

Log tool decisions


Verify

Agent selects correct tool

Tool output influences final answer

All calls are logged



---

Lab 5 — LangGraph Workflow Agent (90 min)

Goal: Replace free-form loop with deterministic workflow.

Build Graph:

Plan

Act

Reflect

Finalize


Tasks

Define state object

Create graph nodes

Define transitions

Add retry logic


Verify

Graph runs deterministically

Reflection step improves output

Failures trigger retry



---

End of Day 1 Challenge (60 min)

Task: Build a mini research assistant that:

Accepts a question

Uses search tool

Summarizes results

Returns structured output


Constraints:

Must use ReAct

Must use LangGraph

Must validate schema



---

Day 2 — Production-Ready Agentic Systems


---

Lab 6 — Memory Systems (75 min)

Goal: Implement memory.

Tasks

Add session memory

Add persistent store (local JSON / sqlite)

Implement retrieval


Verify

Agent recalls prior session data

Memory affects responses



---

Lab 7 — Multi-Agent Collaboration (AutoGen) (90 min)

Goal: Build a team of agents.

Roles:

Planner

Builder

Reviewer


Tasks

Implement round-robin chat

Add termination condition

Add summary agent


Verify

Agents collaborate

Conversation terminates cleanly

Final answer improves via review



---

Lab 8 — Security & Guardrails (60 min)

Goal: Prevent unsafe behavior.

Tasks

Implement tool allowlist

Detect prompt injection patterns

Block dangerous instructions

Add confirmation layer


Verify

Injection attempts are blocked

Disallowed tools cannot run



---

Lab 9 — Evaluation Harness (60 min)

Goal: Measure agent quality.

Tasks

Define golden test cases

Add adversarial prompts

Score responses

Log failures


Verify

Test suite runs automatically

Failures are reproducible



---

Lab 10 — Packaging & CLI (60 min)

Goal: Run agent outside notebook.

Tasks

Refactor into modules

Create CLI entrypoint

Add config file

Build Docker image (optional)


Verify

Agent runs via CLI

Container builds successfully



---

Final Bootcamp Capstone (2 Hours)

Build a Production-Ready Agent System

Must Include:

ADK runtime agent

LangGraph workflow orchestration

At least 2 tools

Memory

Guardrails

Evaluation test

CLI runner


Bonus:

Multi-agent collaboration

Containerized deployment



---

Practical Exam Style Assessment

Participants must:

1. Debug a broken agent


2. Add a new tool safely


3. Fix schema validation


4. Prevent prompt injection


5. Make evaluation tests pass



Time limit: 90 minutes


---

Competency Outcomes

By the end of the 2 days, participants can:

Build deterministic agent workflows

Add reliable structured outputs

Implement tool-using agents

Design multi-agent collaboration

Secure agents from injection

Add evaluation harness

Package for deployment



---

Bootcamp Philosophy (CKAD Style)

Minimal slides

Heavy hands-on

Task-based

Verification-driven

Real-world constraints

Production mindset

