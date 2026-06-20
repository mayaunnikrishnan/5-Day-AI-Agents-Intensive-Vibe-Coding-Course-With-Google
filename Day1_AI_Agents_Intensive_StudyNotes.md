# Day 1 Study Notes — Kaggle x Google 5-Day AI Agents Intensive
### Topic: Introduction to Agents and Vibe Coding

---

## 1. Background: Why This Course Exists

The way software is built has changed a lot in a short time.

- As of early 2026, **85% of professional developers** regularly use AI coding agents.
- **41% of all new code** is AI-generated.
- This is not a future trend — it is already the **current baseline**.
- The real challenge today is the gap between *prompting a model* and *building something you can actually deploy to production*. This gap is what most teams struggle with.

---

## 2. From Syntax to Intent

This is described as one of the biggest shifts in computing history.

- **Old way:** Developers write exact syntax (code rules) to tell the computer what to do.
- **New way:** Developers express **intent** (what they want) in natural language, and AI translates that into code.

This shift changes what skills matter most: knowing the *syntax* of a language is now less important than knowing how to *clearly describe the problem* and *verify the result*.

---

## 3. The Spectrum: Vibe Coding vs Agentic Engineering

Think of this as a spectrum with two ends:

| | **Vibe Coding** (casual) | **Agentic Engineering** (disciplined) |
|---|---|---|
| Process | Prompt the AI, copy-paste errors back, repeat | AI works inside structured, deterministic boundaries |
| Review | Minimal codebase understanding, spot-checking | Systematic testing, CI/CD gating, evaluation checks |
| Best for | Quick prototypes, experiments | Production systems, real businesses |

**Key idea:** Vibe coding is fine for trying things out fast. Agentic engineering is needed when you want something reliable enough to run in production.

---

## 4. Context Engineering

Context engineering = the skill of managing *what information the AI agent sees* at any given time. This is called "the real skill of modern engineering."

Two types of context:

1. **Static context** — Example: system instructions. This is loaded every time, so it is "expensive" (uses up context window space constantly).
2. **Dynamic context** — Example: **agent skills**, loaded only when needed, on demand. This is "cost-efficient" because it doesn't bloat the context unless required.

*(Note: Agent Skills are covered in more depth on Day 3 of this course.)*

---

## 5. How the Software Development Life Cycle (SDLC) Is Changing

In the traditional SDLC, writing the code (implementation) used to take the most time.

**In the new AI-driven SDLC:**
- The implementation phase shrinks from **weeks to minutes**.
- The new bottlenecks become:
  - **Specification quality** — how clearly you describe what you want built.
  - **Verification** — checking that what was built is correct.

This was also confirmed in the live quiz: *"Specification quality"* is the primary new bottleneck in a compressed AI-driven SDLC.

---

## 6. The Factory Model of Software Development

A useful mental model from the white paper:

> As a developer, your output is no longer just the code itself. Your real output is **the system that produces the code**.

This means developers are shifting from "writing lines of code" to "designing and managing the factory (system) that writes code."

---

## 7. The Core Formula: Agent = Model + Harness

This is one of the most important concepts from Day 1.

```
Agent = Model + Harness
```

- **Model** → contributes roughly **10%** of what makes an agent work well.
- **Harness** → contributes roughly **90%**. The harness includes:
  - Sandboxes (safe environments to run code)
  - Tools (things the agent can use, e.g., search, code execution)
  - Orchestration (coordinating multiple steps/agents)
  - Guardrails (safety and correctness checks)

**Interview tip:** If asked "what makes an AI agent reliable?" — the answer is **not just the model**, it's the harness around it.

---

## 8. Two Modes a Developer Works In

Developers now move between two modes:

1. **Conductor Mode** — Actively directing real-time edits inside the IDE (hands-on, moment-to-moment control).
2. **Orchestration Mode** — Asynchronously delegating complex tasks to autonomous agent networks or "swarms" of agents, then checking back later.

---

## 9. Five Parts of Every AI Agent

From the live quiz, every AI agent is built from **five parts**. The most important one to remember:

- **The Model** = the **reasoning engine**. It reads the context and decides what should happen next. (This was the correct quiz answer, not memory, tools, or orchestration.)

*(Other parts mentioned across the discussion: memory, tools, orchestration, and the harness/guardrails layer.)*

---

## 10. Key Risks of an AI-Driven SDLC

Even though there's a lot of optimism, three main long-term risks were discussed:

1. **Erosion of human expertise** — As AI writes and manages more code, developers may lose deep understanding of their own codebase.
2. **Accountability gaps** — If AI is responsible for most decisions, it becomes harder to know who is accountable when something goes wrong.
3. **Lost opportunities for innovation** — Many good ideas come from developers deeply understanding their product. If that understanding fades, fewer human-driven improvements may happen.

**Added risk:** Security gaps become more dangerous if teams lose technical understanding of their own systems.

---

## 11. Closing the "Last 20%" Gap in Agent Tasks

Getting an agent from 80% done to 100% done (production-ready) is the hardest part. Patterns that help:

- **Build → Evaluate → Deploy → Observe → Optimize loop** (a continuous improvement cycle for agents).
- Giving agents a **sandbox to write their own code/tools**, instead of just calling the LLM with a fixed prompt.
- Using **sub-agents** that check/evaluate the main agent's work.
- **Human-in-the-loop checkpoints** — flagging certain situations for a human to review.
- Building a **"golden dataset"** from human reviews, which helps the system self-improve over time.

---

## 12. Open Knowledge Format + Graph RAG (Advanced Concept)

A community question explored combining two ideas:

- **Open Knowledge Format**: A simple system of linked markdown files, where each file represents one "thing" (a service, database, contract, etc.), and files link to each other — like index cards connected with strings.
- **Graph RAG (Retrieval-Augmented Generation)**: Follows those "strings" (connections) to understand relationships — e.g., "If I change X, what else gets affected?"

**Why this matters:** Normally, an agent dropped into a large codebase starts writing code without understanding the full system. Combining these two ideas gives the agent a **map of the whole system** before it starts coding — similar to how a human architect thinks before making a change.

This is essentially an advanced form of **context engineering** — giving the agent a compact, structured view of the system instead of dumping the entire repository into its context window.

---

## 13. Long-Running Agents — When Are They Useful?

Good use cases for agents that run for a long time without constant supervision:

- **Deep Research** — Google's first major long-running consumer agent; goes off, researches, and returns with a structured answer.
- **Coding agents** — work well because their output can be **continuously tested and verified** (so the agent doesn't waste time/money going down a wrong path).
- **Dynamic, multi-step processes**, such as:
  - Bank loan processing (can take weeks, with back-and-forth for more information)
  - Insurance claims
  - Legal case work
  - Scientific research (e.g., AlphaEvolve, co-scientist systems)
  - Longer AI-generated videos/movies

**General rule of thumb:** Tasks that normally take humans a long time to do (and involve changing inputs over time) are good candidates for long-running agents.

**Watch out for:** As agents run longer, new bottlenecks appear — often in the **external tools** the agent calls, since many of those tools were originally built for humans to use (with expected latency/lag), not for fast, repeated agent calls.

---

## 14. AlphaEvolve (Self-Evolving AI for Coding)

- AlphaEvolve is described as an **evolutionary algorithmic agent**.
- It uses a large language model (like Gemini) plus an **evaluator function** to find optimized algorithms for a given problem.
- Past examples: solving long-standing math problems (e.g., matrix multiplication optimization), designing new hardware, optimizing schedulers.
- Newer use cases: DNA sequencing, molecular simulation, financial and retail optimization, infrastructure design.
- **Mental model:** Think of AlphaEvolve as "an expert optimization agent" you can use as an additional tool/skill during your vibe-coding or agentic engineering work — you don't have to manually figure out how to improve performance yourself.

---

## 15. The "I-U-S" Framework (Impressive → Useful → Sustainable)

A simple framework for evaluating any AI/agent project as it matures:

1. **I — Impressive**: The first demo. Solves one specific use case and looks great.
2. **U — Useful**: Can it work for everyone, not just your one example? Is it generally applicable?
3. **S — Sustainable**: Is it scalable, secure, and cost-effective long term? (Many AI solutions look great but cost far more than the traditional approach — sustainability means fixing that.)

**Interview tip:** This is a nice framework to mention when asked "how do you evaluate if an AI/agent solution is production-ready?"

---

## 16. The "Three H's" of AI Safety

When discussing safety for AI/agent systems, keep these three in mind:

1. **Hate**
2. **Harm**
3. **Hallucinations**

This covers grounding the model properly, checking for bias in training data, and verifying outputs before trusting them.

---

## 17. Tools Introduced in Code Labs

Two hands-on tools introduced on Day 1:

1. **Google Antigravity**
   - Described as a "central command center" for managing agents, workspaces, and code.
   - Visual interface — shows implementation plans, lets you track agent tasks, and view generated artifacts.
   - You can switch between different models (Gemini, Claude, GPT) inside it if you run low on token quota for one model.

2. **Google AI Studio**
   - Lets you build an app by describing it in plain English (or any language) — i.e., "vibe coding."
   - You can deploy directly to the cloud (e.g., Cloud Run) with just a few clicks.
   - Goes from "idea" to "deployed URL" in minutes — something that used to take multiple days of manual setup.

**Logan's framing (from Q&A):** The progression is moving from **"prompt to prototype"** → **"prompt to production"** → **"prompt to profitable company."** The long-term vision is that vibe coding could let many more people build software businesses, similar to how YouTube let many more people become content creators.

---

## 18. Quick Recap — Pop Quiz Answers (Good for Self-Testing)

| Question | Correct Answer |
|---|---|
| Which part of an agent is the "reasoning engine" that decides what happens next? | **The Model** |
| Key differentiator of agentic engineering vs vibe coding? | **Systematic testing, CI/CD gating, and evaluation judges** |
| Primary new bottleneck in the AI-driven SDLC? | **Specification quality** |
| Missing piece in "Agent = Model + Harness"? | **The surrounding scaffolding (the harness)** |
| Financial/operational tradeoff of agentic engineering? | **High CapEx, Low OpEx** (higher upfront investment in training/tools/GPUs, but lower ongoing developer time/effort) |

---

## 19. Quick Glossary (For Interview Prep)

- **Vibe Coding**: Casual, prompt-driven coding with manual review and iteration.
- **Agentic Engineering**: Disciplined, systematic AI-assisted development with automated testing and guardrails.
- **Context Engineering**: Managing what information/context an AI agent has access to, and when.
- **Harness**: The surrounding infrastructure (sandboxes, tools, orchestration, guardrails) that makes an AI model usable and reliable as an agent.
- **Orchestration**: Coordinating multiple steps or multiple agents to complete a task.
- **Conductor Mode**: Developer actively directs the agent in real time.
- **Orchestration Mode**: Developer delegates tasks asynchronously to autonomous agents.
- **Agent Skills**: On-demand, loadable pieces of context/capability for an agent (covered Day 3).
- **Graph RAG**: Retrieval method that uses relationships/connections between pieces of information, not just plain text similarity.
- **CapEx / OpEx**: Capital Expenditure (upfront investment) vs Operational Expenditure (ongoing running cost).

---

## 20. Looking Ahead: Day 2 Preview

- **Topic:** Agent Tools and Interoperability.
- Will cover **MCP (Model Context Protocol)** and **A2A (Agent-to-Agent)** — how agents connect to outside tools and to each other.

---

*Notes compiled from the Day 1 livestream Q&A and white paper discussion (Kaggle x Google 5-Day AI Agents Intensive, 4th iteration).*
