# Day 3 Study Notes — Kaggle x Google 5-Day AI Agents Intensive
### Topic: Agent Skills

---

## 0. The Big Picture

Day 1 covered the agent itself (Model + Harness). Day 2 covered connecting agents to the outside world (MCP, A2A, A2UI, UCP, AP2). Day 3 tackles a different problem: **how do you give an agent specialized knowledge without overloading it?**

**Core myth being busted:** More context does NOT equal more capability. Dumping a million pages of instructions into an AI doesn't make it your best employee — it makes it perform *worse*, because of something called **attention dilution**.

**The solution:** A lightweight primitive called an **Agent Skill** — which is quietly replacing complex multi-agent "swarm" architectures for most use cases.

---

## 1. What Is an Agent Skill? (The Physical Anatomy)

Despite sounding technical, a skill is structurally very simple:

> **An agent skill is literally just a folder on your file system.**

### Folder Structure

| Component | Type | Purpose |
|---|---|---|
| **SKILL.md** | Mandatory file | The "brain" — contains metadata (name + precise description of *when* to use it) and core instructions |
| **scripts/** | Optional subfolder | Executable code (Python, bash) for deterministic tasks — e.g., calculating a hash. You don't want the LLM "guessing" math; you want it to run a script. |
| **references/** | Optional subfolder | Dense domain context too long for the main prompt — e.g., a 50-page PDF of tax codes |
| **assets/** | Optional subfolder | Supporting files like JSON schemas or email templates |

**Key principle:** This structure separates **"what to do"** (in SKILL.md) from **"how to do it deterministically"** (in scripts/references/assets).

---

## 2. The Restaurant Kitchen Analogy

A simple way to remember how skills work:

- **SKILL.md metadata** = the menu out front. The agent ("chef") reads the menu to know what's available.
- **scripts/ and references/** = the blender and grandma's recipe, sitting in the back kitchen.
- The chef doesn't plug in the blender or pull out the recipe **until a customer actually orders that specific dish.**

This concept has a formal name: **Progressive Disclosure** — you only reveal the complex mechanics of a task when that task is actively triggered.

---

## 3. Two Paths for Creating Skills

| Path | Who Creates It | How |
|---|---|---|
| **Path A** | Subject matter experts (e.g., HR manager, compliance officer) | They translate existing human instructions (onboarding guides, runbooks) directly into SKILL.md format — no coding required. This **democratizes** agent creation. |
| **Path B** | Developers | A developer watches an agent successfully complete a complex multi-step task, then captures that successful "trace" and crystallizes it into a reusable skill folder — so the agent doesn't have to burn compute figuring it out from scratch next time. |

---

## 4. Agent Skills vs. MCP vs. AGENTS.md (Important Distinction)

These three are often confused, but they solve different problems:

| Concept | What It Provides | Analogy |
|---|---|---|
| **MCP** | **Reach** — connects the agent to external systems (e.g., pulling a record from Salesforce, querying BigQuery) | The bridge to the outside world |
| **Agent Skill** | **Know-how** — the step-by-step procedural logic of what to do once you have the data | The recipe / procedural memory |
| **AGENTS.md** | **Global, always-on context** — project-wide rules (e.g., "always write code in TypeScript") | Standing instructions, always loaded |

**They compose together.** Example: A skill might say *"Here are the 5 steps to process a refund,"* and step 2 might be *"Use the MCP tool to check the database."*

---

## 5. The Problem Skills Solve: Context Rot / Context Drought

### The Naive Assumption (and why it's wrong)
The industry assumed: if you have a 1-million-token context window, just dump every tool, instruction, and API spec into the system prompt and let the model figure it out.

### What Actually Happens
- The "Lost in the Middle" study, and a **2025 Chroma Research study across 18 frontier models**, showed that performance degrades **silently** as input grows.
- **Silently** is the key word — there's no error message. The model just gets less accurate.
- This happens even though the context window can physically hold far more — accuracy drops well before the window is full.

### Why: Attention Dilution
- Every extra token added to the context **competes for the model's attention**.
- If your system prompt has instructions for 50 tools and the user just says "hello," the model wastes compute ignoring 49 irrelevant tools.

---

## 6. How Progressive Disclosure Fixes This (With Numbers)

Example given in the source material:

- You have **50 workflows/skills**.
- Instead of loading all of them, you load only the **metadata** (names + trigger descriptions) — roughly **4,000 tokens** total for all 50.
- When the user's request matches one skill, **only that skill's full body loads** — roughly **2,000 tokens**.
- Result: instead of ~15,000 tokens of active heavy context, you're down to about **6,000 tokens**.
- The white paper reports this can achieve **up to a 98% reduction in active context.**

### Scaling Beyond the Menu: RAG for Skills
At enterprise scale (e.g., 40,000 skills), even loading just the *metadata* for everything would overflow the context window.

**Solution:** Use a lightweight retrieval system (a RAG pipeline):
1. User submits a prompt.
2. A fast embeddings model scans all 40,000 skill descriptions.
3. It retrieves only the **top 5 most relevant** skills.
4. Only those 5 are passed into the agent's context window.

This is progressive disclosure taken to enterprise scale.

---

## 7. Are Multi-Agent "Swarm" Systems Dead?

**No — but their role is much narrower now.**

### When You STILL Need Multiple Agents
1. **Genuine asynchronous parallelism** — e.g., three agents researching three different companies at the same time.
2. **Differing security postures** — e.g., an HR agent with clearance to read salaries should stay separate from a marketing agent that scrapes public websites, so the marketing bot can never accidentally leak payroll data.

### When a Single Agent + Skills Is Better
- Example: A logistics company with 100 shipping process variants.
- Maintaining 100 separate sub-agents (each with its own deployment, memory store, routing logic) is an **operational nightmare**.
- One general-purpose agent that dynamically loads from 100 different skill folders is far more elegant and easier to maintain.

---

## 8. The Danger Zone: When Skills Go Wrong

If anyone can write a skill just by typing a markdown file, this can create chaos. Real data from the source material:

| Statistic | Finding |
|---|---|
| **2025 Skills Bench study** | **19%** of poorly designed skills made the agent perform **worse than having no skill at all** — it actively *subtracted* baseline capability. |
| **Vercel study** | **56% non-invocation rate** — meaning more than half the time, the agent simply **ignored** the skill completely. |

### Four Failure Modes (Memorize These for Interviews)

1. **Trigger Failure** — The description is too vague, so the wrong skill fires, or the right one never fires at all.
2. **Execution Failure** — The skill triggers correctly, but its internal instructions are messy, producing wrong tool calls or hallucinated data.
3. **Token Budget Failure** — The author dumps a massive, unoptimized document into `references/`, blowing up the context window and ruining the agent's short-term memory.
4. **Regression** — A *new* skill's trigger phrase is too similar to an *existing* skill's, hijacking the routing and breaking something that used to work fine.

---

## 9. Evaluation-Driven Development (EDD)

Because skills fail in these specific ways, the white paper **mandates** Evaluation-Driven Development.

**Rule:** Before writing a single line of SKILL.md, you must write **three JSON evaluation cases**, each defining:
- The input
- The expected tool(s) to be called
- The expected output format

> "If you don't know what success looks like, you have no business writing the instructions."

**Testing method:** Use the **single-skill sub-agent pattern** — give a blank agent *only* this one skill, then run the JSON evals against it in isolation.

---

## 10. Trajectory Scoring (Why HOW Matters, Not Just the Final Answer)

This is one of the most important — and counterintuitive — concepts from Day 3.

### The Key Insight
> Scoring only the final output can make a broken skill look like it's working.

- A **2026 analysis by Latitude** found that scoring *only the final output* causes you to pass **20–50% more test cases** than scoring the full trajectory (the actual sequence of tool calls).
- Passing more test cases sounds good — but **not** if the agent is "stumbling into the right answer through disastrous methods."

### Concrete Example
A user asks a customer service agent: *"Refund my last order."*
- The agent gets confused, pulls up the user's profile, **accidentally deletes their shipping address**, but then somehow successfully calls the Stripe refund API.
- **Output-only scoring:** ✅ Test passes (the user got their refund).
- **Trajectory scoring:** ❌ Catches the disaster (the deleted shipping address).

### When to Use Which
| Skill Type | Scoring Approach |
|---|---|
| Read-only (e.g., summarizing a public document) | Order of steps may not matter much |
| **Action-allowed** (touches production databases, payments, etc.) | **Must** use exact or in-order trajectory scoring |

---

## 11. The Demo-to-Deploy Gap

> "Confidence always peaks during the local demo, but it completely shatters in production."

- When something fails in production, teams often blame the model for "hallucinating."
- **In reality, it's rarely a model problem — it's an environmental fit problem.**

---

## 12. The Claude Code Infrastructure Stat (Major Talking Point)

Researchers reverse-engineered **Claude Code version 2.1.88** and found:

- **98.4%** of the codebase was purely **operational infrastructure** — permission classifiers, session storage, retry logic, context compaction pipelines.
- Only **1.6%** of the codebase was the actual **prompt/reasoning loop**.

**Why this matters:**
- The foundation model itself is becoming a **commodity** — swappable for a cheaper/better one at any time.
- The **real, durable, proprietary asset** is everything around the model: the skills, the guardrails, the infrastructure. This is "the steering wheel and the transmission," not the engine.

---

## 13. The Graduation Ladder (Skill Deployment Tiers)

Skills don't go straight to production. They move through a structured staging process:

| Tier | What Happens |
|---|---|
| **1. Read-Only** | Evaluated safely, often by an "LLM-as-a-judge" |
| **2. Draft-Only** | Can generate outputs (e.g., draft emails), but a **human must manually review and approve** before anything goes out |
| **3. Action-Allowed** | Full autonomous execution — but only after clearing a strict reliability bar (see Pass^K below) |

### Pass^K: The Reliability Metric for Reaching "Action-Allowed"

**Pass^K measures sustained reliability across repeated runs**, not just a single success.

**Worked example from the source material:**
- A skill succeeds **60%** of the time on a single run (seems okay for a prototype).
- Require it to succeed **8 times in a row** without a critical failure → that's 0.6^8.
- Result: **~1.6% chance** of clearing that hurdle.

**Why this matters:** Pass^K is intentionally ruthless — it exposes inconsistent skills long before they're allowed to touch real production data.

---

## 14. Meta Skills (AI Writing Its Own Instructions)

The frontier concept from Day 3: letting AI create, improve, or evolve skills itself. Four categories:

1. **Authoring** — A tool drafts a skill based on a simple human prompt.
2. **Assisted Authoring from Traces** — This is "Path B" from earlier: capturing a successful agent trace and turning it into a skill.
3. **Improvement** — An agent runs bounded, automated experiments to optimize an *existing* skill.
4. **Library Evolution** — An agent analyzes a month of chat logs, notices a recurring unmet need, and independently *proposes* a brand-new skill.

### The Non-Negotiable Safety Rule for Meta Skills
> **Anything an agent writes must enter the library at the draft tier. AI can never instantly deploy its own updates.**

**Why this matters:** An agent might optimize a skill's description to pass a specific test, but accidentally choose a trigger phrase that steals traffic from another critical skill (a regression). A human reviewer can catch this — an automated system might not.

---

## 15. Orchestration at Scale: DAGs and the File Message Bus

**Problem:** If a complex task uses three skills in sequence, and each skill's output gets dumped back into the context window for the next skill to read, you've recreated the exact context bloat problem skills were meant to solve.

> **Golden rule: You cannot use the context window as a database.**

### The Solution

**DAG (Directed Acyclic Graph) orchestration:**
- "Acyclic" = no cycles, no infinite loops.
- It's a structured map of dependencies between skill steps — a one-way flow, not a back-and-forth loop.

**File Message Bus:**
- Instead of printing a massive (e.g., 10,000-line) JSON output directly into the chat history for the next skill to read...
- ...the output is saved to a **hidden file on disk**, and only the **file path (URI)** is passed to the next skill.
- The LLM's attention is protected — it only ever sees a short reference like "data saved at file X," not the raw bulk data.

### "Shift Intelligence Left" (Important Phrase for Interviews)
> Relying on **deterministic software constraints** rather than **prompt engineering**.

- ❌ Don't write a system prompt yelling in all caps: *"ALWAYS VALIDATE THE ZIP CODE"* — this is just hoping the LLM pays attention.
- ✅ Instead, write a rigid, simple Python script (in the `scripts/` folder) that validates the zip code deterministically, and have the LLM call that tool.

**Summary phrase:** *"You rely on code, not vibes."*

---

## 16. Safely Adopting Skills From the Public Ecosystem

With over **40,000 public skills** available (e.g., Google's official repository at `github.com/google/skills`), the white paper gives three firm heuristics:

1. **Prefer first-party skills** — e.g., use the official Google BigQuery skill rather than a random community-made one.
2. **Pin your versions aggressively** — so an unexpected update doesn't silently break your DAG pipeline.
3. **Audit the code before adopting** — remember, a skill can contain executable Python scripts. **Treat it like a supply chain software dependency.**

---

## 17. Case Study: Home Improvement Retailer (Appendix B)

### The Old (Failed) Approach
A retailer's AI team tried to build **one massive, monolithic retail assistant**. It failed because AI engineers don't actually know the nuances of plumbing codes or lumber grading.

### The Agent Skills Approach
Ownership of knowledge is **distributed to actual domain experts** — each team owns and maintains their own skill folder.

### Example Customer Query: *"I want to tile my shower. What do I need, and can I get it delivered by Tuesday?"*

The agent dynamically loads **three skills in sequence** via the DAG:

1. **Project Guidance Skill** (owned by the retailer's trades team)
   → Loads expert logic on waterproofing substrates and grout types.
   → Result is passed via the file message bus to the next step.

2. **Materials List Skill** (owned by the pro merchandising team)
   → Calculates exact square footage and wastage needed.

3. **Delivery Window Skill** (owned entirely by the fulfillment team)
   → Runs a deterministic script against the inventory database to check Tuesday delivery availability.

### Why This Matters Strategically
- The AI engineering team never needs to become plumbing experts — the actual plumbing experts write a simple markdown file, and the AI reads it when needed.
- **This distributed ownership becomes a company's strategic moat:** a competitor can license the same foundation model, but **cannot buy your codified, version-controlled institutional knowledge.**

---

## 18. Quick Glossary (For Interview Prep)

- **Agent Skill**: A folder containing a SKILL.md file (+ optional scripts/references/assets) that gives an agent on-demand, specialized "know-how."
- **SKILL.md**: The mandatory metadata + instructions file inside a skill folder — the "brain" of the skill.
- **Progressive Disclosure**: Only loading the detailed mechanics of a skill when it's actually triggered, not all the time.
- **Attention Dilution**: Performance degradation that happens when too many irrelevant tokens compete for a model's attention.
- **Context Rot / Context Drought**: The phenomenon where stuffing more context into a model silently reduces its accuracy.
- **MCP vs Skill vs AGENTS.md**: MCP = reach (connects to external systems); Skill = know-how (procedural steps); AGENTS.md = global always-on rules.
- **Path A / Path B**: Two ways skills get authored — by subject matter experts directly (A), or by developers crystallizing a successful agent trace (B).
- **Trigger Failure / Execution Failure / Token Budget Failure / Regression**: The four named failure modes for poorly built skills.
- **Evaluation-Driven Development (EDD)**: Writing JSON eval cases *before* writing the skill instructions.
- **Trajectory Scoring**: Evaluating the *sequence* of actions an agent took, not just the final output — critical for catching dangerous intermediate steps.
- **Pass^K**: A reliability metric requiring K consecutive successful runs, used as the bar for promoting a skill to full autonomy.
- **Graduation Ladder**: The staged rollout of a skill — Read-Only → Draft-Only → Action-Allowed.
- **Meta Skills**: Skills authored, improved, or evolved by AI itself (Authoring, Assisted Authoring from Traces, Improvement, Library Evolution).
- **DAG (Directed Acyclic Graph) Orchestration**: A one-way, no-loop structure for managing dependencies between multiple skill steps.
- **File Message Bus**: Passing a file path/URI between skill steps instead of dumping raw data into the context window.
- **Shift Intelligence Left**: Using deterministic code/scripts instead of relying on prompt instructions for critical logic.

---

## 19. Interview-Style Q&A (Self-Test)

**Q: Why does adding more context to an agent sometimes make it perform worse?**
A: Because of attention dilution — every additional token competes for the model's limited attention. Research (e.g., the 2025 Chroma study) shows accuracy degrades silently as input grows, even well before the context window is full.

**Q: What's the structural difference between an MCP server and an Agent Skill?**
A: MCP gives an agent "reach" — a bridge to external systems/data (e.g., querying a database). A Skill gives an agent "know-how" — the procedural, step-by-step logic for what to do with that data once retrieved. They're complementary, not competing.

**Q: Why is trajectory scoring sometimes more important than output-only scoring?**
A: Because an agent can produce a correct final result through a disastrous internal process (e.g., accidentally deleting data on the way to successfully issuing a refund). Output-only scoring would pass this case; trajectory scoring catches the dangerous intermediate step.

**Q: What does Pass^K measure, and why is it strict?**
A: It measures the probability that a skill succeeds K times in a row. Even a skill with a "decent" 60% single-run success rate has only about a 1.6% chance of succeeding 8 times consecutively — which deliberately filters out unreliable skills before they're allowed full autonomy.

**Q: Why use a "file message bus" instead of just passing data directly between skills in the chat?**
A: To avoid re-creating the context bloat problem skills were designed to solve. Large outputs are saved to disk, and only a short file reference is passed forward, keeping the model's active context clean.

**Q: What is the one non-negotiable safety rule for "meta skills" (AI editing its own skill library)?**
A: Anything an AI writes or modifies must enter the library at the draft tier first — a human must review it before it can go live. AI can never self-deploy its own updates.

---

*Notes compiled from the Day 3 podcast deep-dive discussion (Kaggle x Google 5-Day AI Agents Intensive, Day 3 white paper: Agent Skills).*
