# AgentFlow Studio

> A browser-based multi-agent research and decision-brief workspace with a visual execution graph, agent inspection, evidence checking, reviewer approval, and live Gemini/Tavily integration.

## Overview

**AgentFlow Studio** is an interactive multi-agent workflow prototype designed to turn an open-ended question into a structured decision brief.

Instead of asking one model to perform every task in a single prompt, the application separates the work into six specialized roles:

```text
                         ┌──────────────┐
                         │  Supervisor  │
                         │  Plans work  │
                         └──────┬───────┘
                                │
                   ┌────────────┴────────────┐
                   │                         │
            ┌──────▼──────┐          ┌──────▼──────┐
            │  Researcher │          │   Analyst   │
            │  Evidence   │          │   Options   │
            └──────┬──────┘          └──────┬──────┘
                   │                         │
                   └────────────┬────────────┘
                                │
                         ┌──────▼───────┐
                         │ Fact Checker │
                         │   Claims     │
                         └──────┬───────┘
                                │
                         ┌──────▼───────┐
                         │    Writer     │
                         │ Decision brief│
                         └──────┬───────┘
                                │
                         ┌──────▼───────┐
                         │   Reviewer    │
                         │ Quality gate  │
                         └───────────────┘
```

The interface makes the workflow observable rather than hiding it behind a single loading spinner. Users can watch agents execute, inspect their outputs and state, follow the event log, and view the final decision brief after review.

The application supports both a **scripted demonstration mode** and a **live mode** using Gemini and Tavily.

---

## Why this project?

Many AI applications produce an answer without showing how the answer was developed.

AgentFlow Studio explores a different approach:

- Break a complex question into specialized tasks.
- Run independent workstreams in parallel where possible.
- Pass structured state between agents.
- Separate research, analysis, verification, writing, and review.
- Make intermediate agent activity visible.
- Attach confidence and caveats to important claims.
- Require a reviewer before releasing the final brief.
- Provide a structured final output instead of returning raw model text.

The goal is not simply to generate text. The goal is to demonstrate a more transparent **agentic workflow for research and decision support**.

---

# Key Features

## 1. Six-agent workflow

The application defines six specialized agents.

| Agent | Responsibility | Main output |
|---|---|---|
| **Supervisor** | Plans the investigation and splits the work | Workflow plan and agent-specific tasks |
| **Researcher** | Collects evidence and market signals | Research notes and sources |
| **Analyst** | Compares options and trade-offs | Strategic analysis |
| **Fact Checker** | Challenges important claims | Confidence labels and caveats |
| **Writer** | Combines the verified work | Decision brief |
| **Reviewer** | Evaluates the final brief | Score, feedback, approval/revision decision |

The roles and handoffs are explicitly represented in the application rather than being only conceptual documentation.

---

## 2. Parallel Researcher + Analyst execution

After the Supervisor creates the plan, the Researcher and Analyst execute concurrently.

Conceptually:

```text
Supervisor
    │
    ├──────────────► Researcher
    │
    └──────────────► Analyst
                       │
          both complete│
                       ▼
                  Fact Checker
```

The implementation uses JavaScript `Promise.all()` for this parallel stage.

This reduces unnecessary sequential waiting and demonstrates how independent agent work can be executed concurrently.

---

## 3. Visual execution graph

The UI contains a live execution graph showing:

- Agent nodes
- Agent status
- Connections between agents
- Running animation
- Completion state
- Failure state
- Execution time
- Current active agent
- Overall progress

The graph is generated dynamically from the agent definitions and workflow edges.

---

## 4. Agent Inspector

Each agent can be selected from the workflow graph or inspector.

The inspector exposes three views:

### Output

Shows the agent's human-readable summary, findings, and handoff.

### State added

Shows the structured state that the agent contributed to the workflow.

### Raw

Shows the underlying node state in JSON form.

This is useful when debugging or demonstrating how information moves through a multi-agent system.

---

## 5. Event log

Every agent start and completion is recorded as an event.

The event log provides:

- Agent name
- Start/end event
- Attempt number
- Execution duration
- Chronological workflow activity

This gives the user visibility into the workflow instead of treating the process as a black box.

---

## 6. Decision brief generation

Once the Writer produces a draft and the Reviewer approves it, the application displays a structured decision brief.

The generated brief is designed around sections such as:

1. Executive Summary
2. Scope and Approach
3. Landscape
4. Options Compared
5. Recommendation
6. Proposed Architecture
7. Evidence and Confidence
8. Sources and Evidence Basis
9. 90-Day Plan
10. Risks and Mitigations
11. Open Questions
12. Next Action

The application also supports structured visual components for:

- Architecture
- Option comparison
- Sources/evidence basis

---

## 7. Reviewer quality gate

The Reviewer acts as the final quality gate.

The live workflow asks the Reviewer to score the brief on:

- Clarity
- Evidence support
- Specificity
- Actionability

The workflow uses an approval threshold of **7.5/10**.

If the score is below the threshold, the Writer receives the Reviewer feedback and can revise the brief.

The implementation allows **one revision cycle**.

```text
Writer
   │
   ▼
Reviewer
   │
   ├── Score >= 7.5 ──► Approved
   │
   └── Score < 7.5 ──► Writer revision
                              │
                              ▼
                           Reviewer
```

---

# Execution Modes

## Scripted Demo Mode

The default mode is a deterministic demonstration.

The agent responses are predefined so the complete workflow can be demonstrated without requiring external API keys.

This makes the project easy to showcase in:

- College demonstrations
- Portfolio presentations
- UI/UX demonstrations
- Agentic AI workflow explanations
- Offline testing

### Important

The scripted mode does **not** perform real research.

The same scripted report can be returned regardless of the question entered by the user.

Therefore, it should be presented as a demonstration of the workflow and interface, not as live research.

---

## Live with Gemini

Live mode uses Google's Gemini API for the model-driven agent steps.

The application currently calls:

```text
Gemini 2.5 Flash
```

through the Google Generative Language API.

The model is instructed to return structured JSON so that the application can pass predictable state between agents.

The Gemini generation configuration uses:

- Temperature: `0.2`
- Top P: `0.9`
- Maximum output tokens: `1500`
- JSON response format

---

## Tavily Web Search

The Researcher uses Tavily for external search grounding in live mode.

The application sends the user question together with the Supervisor's Researcher task to Tavily and retrieves up to five search results.

Those results are then passed into the Researcher prompt as evidence context.

The workflow is designed to discourage unsupported claims and explicitly asks agents to lower confidence when a claim is not supported by available evidence.

---

## Fallback behavior

The application contains multiple fallback paths.

A simplified view is:

```text
                 Gemini
                   │
             failure / unavailable
                   ▼
                 Tavily
                   │
             failure / unavailable
                   ▼
          Public search fallback
                   │
             backup result
```

The browser fallback uses the DuckDuckGo Instant Answer API.

Fallback output is explicitly marked as provisional and should not be treated as equivalent to verified research.

---

# Architecture

AgentFlow Studio is intentionally lightweight.

It is implemented as a browser application with the main UI, styling, orchestration logic, API calls, rendering, and state management contained in the application source.

```text
┌─────────────────────────────────────────────┐
│              Browser / UI                   │
│                                             │
│  Query input                                │
│  Execution graph                            │
│  Agent inspector                            │
│  Event log                                  │
│  Decision brief                             │
│                                             │
├─────────────────────────────────────────────┤
│         Client-side orchestration            │
│                                             │
│  Supervisor                                 │
│  Researcher ───────┐                        │
│  Analyst ──────────┤ Parallel               │
│                    ▼                        │
│  Fact Checker                                │
│  Writer                                      │
│  Reviewer                                    │
│                                             │
├───────────────────────┬─────────────────────┤
│                       │                     │
▼                       ▼                     ▼
Gemini API            Tavily API        Browser fallback
LLM reasoning         Web search        DuckDuckGo
└─────────────────────────────────────────────┘
```

---

# Workflow State

The application maintains a shared in-memory workflow state.

A simplified representation is:

```javascript
{
  mode: "demo" | "live",
  phase: "idle" | "running" | "done",
  nodes: {
    supervisor: {},
    researcher: {},
    analyst: {},
    "fact-checker": {},
    writer: {},
    reviewer: {}
  },
  events: [],
  report: "",
  visuals: {},
  score: null,
  elapsed: null,
  selected: "supervisor",
  tab: "output"
}
```

Each completed agent contributes:

- Status
- Attempt number
- Execution time
- Summary
- Findings
- Handoff
- Structured state

This state is then consumed by downstream agents.

---

# Data Flow

## Step 1 — User question

The user enters a question into the main input.

Example:

```text
Analyze the enterprise AI agent market and recommend a defensible
product wedge for a seed-stage startup.
```

The application requires a minimum query length before execution.

---

## Step 2 — Supervisor

The Supervisor:

1. Understands the question.
2. Defines the decision that needs to be made.
3. Creates a workflow plan.
4. Creates a Researcher task.
5. Creates an Analyst task.

The two tasks are intentionally separated to reduce overlap.

---

## Step 3 — Researcher

The Researcher focuses on:

- Market signals
- Buyer priorities
- Competitive patterns
- Available evidence
- Search results

In live mode, Tavily provides external search results.

---

## Step 4 — Analyst

The Analyst focuses on:

- Strategic options
- Trade-offs
- Defensibility
- Product implications
- Go-to-market implications

The Analyst receives the Supervisor's task and notes.

---

## Step 5 — Fact Checker

The Fact Checker receives the Researcher's and Analyst's work.

It:

- Extracts important claims.
- Challenges the claims.
- Assigns confidence.
- Flags unsupported figures.
- Provides caveats.

Confidence is represented as:

```text
high
medium
low
```

---

## Step 6 — Writer

The Writer receives:

- Original question
- Supervisor plan
- Research
- Analysis
- Verified claims
- Caveats
- Optional Reviewer feedback

It then creates the final Markdown decision brief.

---

## Step 7 — Reviewer

The Reviewer evaluates the draft.

If the result meets the approval threshold:

```text
Reviewer → Approved → Final brief
```

If it does not:

```text
Reviewer → Feedback → Writer → Revised brief → Reviewer
```

Only one revision cycle is allowed by the current implementation.

---

# Structured Visuals

The Writer workflow also generates structured data used by the interface.

## Architecture diagram

The architecture visual is represented as layers containing nodes.

Example:

```javascript
{
  architecture: {
    title: "Governed agent operations platform",
    layers: [
      {
        name: "Customer agents",
        nodes: [
          "Internal copilots",
          "Vendor agent frameworks",
          "Custom workflows"
        ]
      }
    ]
  }
}
```

The UI converts this structured data into an SVG-based architecture diagram.

---

## Option comparison

The comparison visual contains:

- Criteria
- Candidate options
- 1–5 scores
- Verdict
- Recommended option

The scores are explicitly presented as model judgement rather than measured market data.

---

## Sources panel

The source structure identifies:

- Source type or publisher category
- Why that source should be checked

The system intentionally avoids inventing source titles, URLs, or statistics in the structured visual generation step.

---

# Markdown Rendering

The application includes a lightweight Markdown renderer written specifically for the generated brief.

It supports the Markdown structures required by the application, including:

- Headings
- Paragraphs
- Bold text
- Italic text
- Inline code
- Bullet lists
- Numbered lists
- Blockquotes
- Markdown tables
- Custom visualization placeholders

The custom placeholders are:

```text
[[comparison]]
[[architecture]]
[[sources]]
```

These are replaced by interactive visual sections in the rendered brief.

---

# User Interface

The interface is designed around observability.

### Main sections

```text
┌──────────────────────────────────────────────┐
│ AgentFlow Studio                    Mode     │
├──────────────────────────────────────────────┤
│ What should the team investigate?            │
│                                              │
│ [ Question textarea                    ]     │
│ [ Run workflow ]                             │
│                                              │
│ Scripted demo | Live with Gemini             │
│ Gemini key        Tavily key                 │
├──────────────────────────────────────────────┤
│ Execution Graph                              │
│                                              │
│ Supervisor → Researcher ─┐                   │
│             Analyst ─────┤→ Fact Checker     │
│                          │        ↓           │
│                          │      Writer        │
│                          │        ↓           │
│                          └──── Reviewer       │
├──────────────────────┬───────────────────────┤
│ Inspector            │ Decision Brief        │
│                      │                       │
│ Output               │ Executive Summary     │
│ State added          │ Architecture         │
│ Raw                  │ Comparison            │
│ Event log            │ Evidence              │
│                      │ 90-day plan           │
└──────────────────────┴───────────────────────┘
```

---

# Accessibility and UX Details

The implementation includes several accessibility and usability considerations:

- Semantic buttons and labels
- `aria-label` attributes
- `aria-live` for current execution information
- `aria-selected` / `aria-pressed` state handling
- Keyboard shortcut for running the workflow
- Visible focus styling
- Reduced-motion support
- Responsive layout
- Scrollable execution graph on smaller screens
- Dark-mode support through theme variables and system preference

The interface also provides an **Auto-follow** option that moves the graph view to the currently active agent.

---

# Keyboard Shortcut

Run the workflow using:

```text
Ctrl + Enter
```

or:

```text
Cmd + Enter
```

---

# API Keys

The application provides fields for:

```text
Gemini API key
Tavily API key
```

Keys are read from the browser and persisted in `localStorage` for convenience.

### Important security warning

This is a **client-side prototype**.

The API keys are used directly from the browser. That means this architecture should **not** be considered suitable for securely handling production API credentials.

For production deployment, the recommended architecture is:

```text
Browser
   │
   ▼
Your Backend / API Gateway
   │
   ├── Gemini
   └── Tavily
```

The backend should keep provider credentials in environment variables or a proper secret-management system.

Do **not** commit API keys to GitHub.

Do **not** hard-code API keys into the source code.

If a key has accidentally been committed to a public repository, revoke/rotate it immediately.

---

# Running the Project

Because the application is a browser-based HTML/JavaScript project, there is no Node.js build pipeline required by the current source.

## Option 1 — Open locally

Download or clone the repository and open the HTML file in a modern browser.

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
cd YOUR_REPOSITORY
```

Then open the HTML file in your browser.

---

## Option 2 — Run through a local server

Using Python:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

Using VS Code, you can also use a local development server extension if preferred.

A local server is generally preferable when testing browser API requests and deployment behavior.

---

# Suggested Repository Structure

If you keep the current single-file implementation:

```text
agentflow-studio/
│
├── index.html
├── README.md
└── LICENSE
```

A future production-oriented version could be reorganized into:

```text
agentflow-studio/
│
├── frontend/
│   ├── index.html
│   ├── styles/
│   └── src/
│
├── backend/
│   ├── agents/
│   ├── providers/
│   ├── routes/
│   └── server/
│
├── README.md
├── LICENSE
└── .gitignore
```

The second structure is a recommendation for future development, not the current project structure.

---

# Environment Variables for a Future Backend

The current browser version does not use a `.env` file.

If the project is later moved to a backend architecture, credentials should be handled with environment variables such as:

```env
GEMINI_API_KEY=your_gemini_key
TAVILY_API_KEY=your_tavily_key
```

These should never be committed.

Example `.gitignore` entry:

```gitignore
.env
.env.*
!.env.example
```

---

# Deployment

The frontend can be hosted as a static web application.

Possible hosting approaches include:

- GitHub Pages
- Netlify
- Vercel
- Cloudflare Pages
- Any static web server

However, there is an important distinction:

### Demo deployment

The scripted demo can be deployed as a static site without external API credentials.

### Live deployment

The current live architecture exposes API calls from the browser.

For a serious public production deployment, move provider calls behind a backend/API gateway before distributing provider credentials to users.

---

# Error Handling and Resilience

The application includes handling for several provider failure scenarios.

Examples include:

- Missing API key
- Invalid API key
- Unauthorized requests
- Rate limits
- Quota exhaustion
- Network failures
- Provider service errors
- Invalid model JSON
- Empty model responses

The application attempts to recover from certain failures using fallback paths and exposes provider diagnostics to the workflow rather than silently presenting a failure as successful research.

---

# Limitations

AgentFlow Studio is currently a **prototype / demonstration application**.

Important limitations include:

### 1. Browser-side API calls

Live provider requests originate from the browser.

This is convenient for a prototype but is not the preferred production security architecture.

### 2. Demo mode is scripted

The default demo workflow does not perform real research and returns predefined agent outputs.

### 3. Browser fallback is limited

The DuckDuckGo fallback is intended as a backup path, not a replacement for a dedicated research provider.

### 4. Model-generated information still requires verification

Even with a Researcher, Fact Checker, and Reviewer, an LLM-based workflow cannot guarantee factual correctness.

The application explicitly encourages confidence labels and caveats, but users should independently verify high-impact claims.

### 5. In-memory run state

The workflow state is maintained in the current browser session. It is not a persistent database-backed job system.

### 6. No authentication system

The current application does not provide user accounts, roles, access control, or multi-user collaboration.

### 7. No production observability backend

The event log is a UI-level representation of the current run, not a centralized telemetry or audit system.

### 8. No durable agent queue

The workflow is executed in the browser rather than through a persistent server-side queue.

---

# What Makes It an Agentic Workflow?

The project demonstrates several characteristics commonly associated with agentic systems:

- **Task decomposition** — the Supervisor turns a broad question into focused work.
- **Specialized roles** — different agents perform different responsibilities.
- **Tool use** — the Researcher can use external search through Tavily.
- **State passing** — downstream agents consume structured results from previous agents.
- **Parallel execution** — Researcher and Analyst work independently at the same stage.
- **Evaluation** — the Fact Checker evaluates claims and the Reviewer evaluates the final brief.
- **Iterative refinement** — Reviewer feedback can send the Writer through another pass.
- **Structured output** — agent responses are normalized into predictable state.
- **Observable execution** — the UI exposes the workflow rather than hiding it.

The implementation is a custom JavaScript orchestration layer. It does **not** depend on LangGraph or another agent orchestration framework in the current version.

---

# Design Principles

## Transparency over black-box output

The interface exposes intermediate agent activity.

## Specialized agents over one giant prompt

Each role has a focused responsibility.

## Evidence before confidence

Claims are expected to carry confidence and caveats.

## Review before release

The Writer does not represent the final quality gate; the Reviewer does.

## Graceful degradation

The system attempts fallback paths when external providers fail.

## Model/provider neutrality

The conceptual workflow is separated from any single model provider, even though the current live implementation uses Gemini.

---

# Example Use Cases

AgentFlow Studio can be adapted for:

### Market research

```text
Analyze the market for AI-powered customer support tools.
```

### Product strategy

```text
Compare three product directions for a seed-stage startup.
```

### Competitive analysis

```text
Compare the major approaches to enterprise AI agent deployment.
```

### Technology evaluation

```text
Evaluate different architectures for an internal AI assistant.
```

### Business decision support

```text
Compare the options for launching a new SaaS product.
```

The quality of live results depends on the question, available evidence, provider availability, and model output.

---

# Project Goals

The project was built to explore how a multi-agent system can be made:

- Understandable
- Observable
- Modular
- Reviewable
- Evidence-aware
- Interactive
- Easy to demonstrate

Rather than presenting an AI system as a single prompt box, AgentFlow Studio exposes the workflow that produces the final answer.

---

# Future Improvements

Potential next steps include:

- Move API calls to a secure backend.
- Add authentication and user sessions.
- Persist workflow runs in a database.
- Add real-time server-sent events or WebSockets.
- Add more research providers.
- Add source citation extraction and validation.
- Add configurable agent roles.
- Allow users to edit agent instructions.
- Add workflow templates.
- Add run history.
- Add export to PDF/DOCX.
- Add cost and token tracking.
- Add production telemetry.
- Add human approval checkpoints.
- Add retry policies per agent.
- Add configurable review thresholds.
- Add model selection.
- Add automated evaluation datasets.
- Add unit and integration tests.
- Split the single-file application into a maintainable frontend/backend architecture.

---

# Technical Summary

| Area | Current implementation |
|---|---|
| UI | HTML + CSS |
| Application logic | Vanilla JavaScript |
| Agent orchestration | Custom JavaScript |
| AI model | Gemini 2.5 Flash |
| Web research | Tavily |
| Search fallback | DuckDuckGo Instant Answer API |
| Visualization | HTML + CSS + SVG |
| State | In-memory JavaScript state |
| Persistence | API keys in browser `localStorage` |
| Markdown | Custom lightweight renderer |
| Build system | None required |
| Framework | No frontend framework |
| Backend | None |
| Database | None |
| Authentication | None |

---

# Responsible Use

This project is intended for research, experimentation, demonstrations, and decision-support prototyping.

AI-generated recommendations should not automatically be treated as verified facts.

For consequential decisions:

1. Check important claims against primary sources.
2. Verify financial, legal, medical, security, or regulatory information independently.
3. Review the original evidence behind important recommendations.
4. Treat model confidence as an indication of uncertainty, not proof of correctness.
5. Do not expose private information or API credentials in prompts or public repositories.

---

# Contributing

Contributions are welcome.

A typical contribution flow:

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git

cd YOUR_REPOSITORY

git checkout -b feature/your-feature
```

Make your changes, test them locally, then:

```bash
git add .
git commit -m "Add your feature"
git push origin feature/your-feature
```

Open a pull request describing:

- What changed
- Why it changed
- How it was tested
- Any limitations or follow-up work

---

# Author

**Muthamayeen Ahmed**

Built as an exploration of multi-agent AI workflows, browser-based orchestration, research grounding, structured state, and AI-assisted decision support.

---

## Project Status

**Status:** Prototype / Demonstration

The current version successfully demonstrates the end-to-end concept:

```text
Question
   ↓
Supervisor
   ↓
Researcher + Analyst
   ↓
Fact Checker
   ↓
Writer
   ↓
Reviewer
   ↓
Approved Decision Brief
```

The project is intentionally transparent about the difference between its scripted demonstration and live AI workflow.

---

## Acknowledgements

This project integrates external AI/search services through their APIs:

- Google Gemini
- Tavily
- DuckDuckGo Instant Answer API

Refer to the respective providers' current documentation and terms before deploying the project commercially or at scale.
