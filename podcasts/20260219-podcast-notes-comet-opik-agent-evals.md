# Podcast Notes: Agent Evaluation, Optimization & the Future of LLM Development

**Guest:** Gideon Mendels — Co-founder & CEO, Comet
**Host:** Kevin Ball (K-Ball) — VP of Engineering at Mento
**Topic:** Why evals are the missing foundation for AI teams, prompt optimization as a search problem, and continuously improving agents in production

---

## Executive Summary

Gideon Mendels, CEO of Comet (an MLOps platform powering teams at Uber, Netflix, Etsy, Shopify, and Autodesk with ~150K engineers), discusses how building LLM-powered agents sits between traditional software engineering and machine learning. He makes the case that **evaluations (evals) are the single most important missing piece** for most AI teams, that **prompt optimization should be treated as a search/optimization problem** rather than a manual debugging exercise, and that the industry is heading toward **continuous, automated agent improvement loops** modeled after ML retraining pipelines.

---

## Background & Context

### Gideon's Path to Comet

- Started as a software engineer, shifted to ML ~10–12 years ago.
- Worked at Google (2016) on hate speech detection in YouTube comments using pre-transformer models (LSTMs). Despite having data, compute, and smart people, the team struggled — not because of algorithmic difficulty, but because of **process and tooling gaps** in ML workflows.
- Co-founded Comet in 2017–18, starting with **model experiment tracking** (hyperparameters, dataset versioning, results comparison, collaboration).
- Expanded into dataset versioning, model registries, and model monitoring.

### The Shift to LLM Applications

- ~2 years ago, customers began saying: "We're not training models anymore — we're building on top of OpenAI's API, but the workflow is similar."
- Initial approach: bolt features onto the existing platform.
- Realization: enough differences to warrant a **separate product**.
- **September 2024:** Launched **OPIK** — an open-source platform for evaluation, optimization, and observability of LLM agents and applications.

---

## Key Differences: Traditional ML vs. Agent Development

| Dimension | Traditional ML | LLM Agent Development |
|---|---|---|
| **Core asset** | Training data + model weights | System prompt + tool configs + context |
| **Control** | Full control over model weights | No control over weights (API consumer) |
| **Tuning variables** | Hyperparameters, data augmentation, architecture | System prompt, tool descriptions, chunking strategy, vector DB config, LLM selection |
| **Deployment** | Ship model binaries | Ship prompt configurations |
| **Key similarity** | Searching for the best combination of variables to maximize a score | Same |

**Bottom line:** Agent development lives **between software engineering and ML** — it's not purely either, but draws from both paradigms.

---

## The Non-Determinism Challenge

### Multiple Levels of Non-Determinism

- **LLM output:** Even at temperature=0, outputs aren't truly deterministic (due to mixture-of-experts deployment, GPU floating point variations).
- **Semantic equivalence:** Two outputs can be semantically identical but completely different as strings — making traditional assertions (string matching, unit tests) useless.
- **Generalization:** Unlike unit tests that cover specific cases, you need a test suite with enough **coverage and variance** to build confidence the system works broadly.

### What Changes for Software Engineers

- You can't rely on pass/fail binary assertions.
- You need to think about **measuring distance** between expected and actual outputs.
- The concept of **generalization** from ML (does the test suite give confidence beyond the exact examples?) is new for most software engineers.

---

## Evaluations (Evals): The Missing Foundation

### What Evals Are

At the most basic level: a list of inputs, a list of expected outputs, and a way to **measure the distance** between agent output and expected output. This produces a score — pass/fail, percentage correct, etc.

### Types of Evaluation Metrics

- **Deterministic metrics:** BLEU score, schema validation, exact match on structured outputs.
- **LLM-as-judge:** Use another LLM to assess quality, relevance, correctness.

### Why Most Teams Don't Do Evals (Despite Knowing They Should)

1. **Domain knowledge gap:** The engineer building the agent often isn't the domain expert who knows the correct answer (e.g., PTO policy in Norway for a 2.5-year employee).
2. **Complexity of constructing test data:** End-to-end tests are manageable, but testing specific tool selection or graph traversals is extremely hard.
3. **Effort vs. urgency:** Teams feel pressure to ship and default to "vibe checking" — testing a few inputs, seeing if output looks right, pushing to production.

### The Product Solution: Bootstrap Evals from Production Incidents

Rather than trying to build eval datasets upfront, Comet/OPIK focuses on **capturing regressions as they happen**:

1. Agent gives wrong answer in production.
2. Domain expert (e.g., HR person) provides the correct answer in free text.
3. System converts this into a new eval sample (input → expected output with assertion).
4. Next time the agent is modified, the growing eval suite runs automatically.

This is a **product-level solution** — it requires minimal extra work because people fix production issues anyway.

### Levels of Evaluation (Mapping to Software Testing)

| Software Equivalent | Agent Eval Equivalent |
|---|---|
| **System/E2E test** | End-to-end: input → final output. Easiest to build, high value. Start here. |
| **Integration test** | Tool selection: given context + history, did it call the right tool? (A classification problem.) |
| **Unit test** | Individual tool/sub-agent: test a specific LLM call or sub-agent in isolation. |

**Recommended starting point:** Begin with end-to-end system tests. They're the easiest dataset to compile and provide the most value.

### Test-Driven Development for Agents

Conceptually possible but rarely practiced today:

1. Build eval dataset with subject matter expert.
2. One-shot a minimal system prompt.
3. Run against eval dataset.
4. See what fails → add tools, refine prompt, iterate.

The industry is heading this way but isn't there yet.

---

## Prompt Optimization as a Search Problem

### The Core Insight

Everything in agent development is a **search problem**: you have a system with variables (system prompt, tool descriptions, chunking config, number of vector DB results, etc.), and an eval score. You're searching for the **global maximum** of this function.

### Optimization Approaches (Increasing Sophistication)

1. **Brute force:** Try all combinations. Infeasible (prompt space is infinite) but conceptually correct.
2. **Random search:** Randomly sample configurations within defined ranges. Better but still inefficient.
3. **Bayesian search:** More informed search using ML methods.
4. **LLM-assisted optimization:** Use an LLM to analyze failed samples, inspect traces, identify common failure modes, and **suggest new prompt candidates**. Run the candidate, measure improvement, iterate.
5. **Evolutionary approaches (e.g., DSPy's MIPRO/COPRO):** Generate multiple candidates, merge the best ones, evolve over iterations.

### Key Advantages Over Manual Debugging

- **Small datasets work:** Even 20–30 samples can drive significant optimization (unlike ML training which needs thousands).
- **Low cost:** Example — optimizing LangChain's structured output prompt went from **12% pass rate to 96% in two iterations for less than $1** in API costs.
- **Automatable:** Can run as nightly optimization jobs rather than requiring manual trace inspection.

### Real-World Example: LangChain Structured Output

- LangChain had a built-in prompt for LLMs without native structured output support.
- It was only achieving ~12% schema compliance.
- OPIK's optimizer found a dramatically better prompt in 2 iterations.
- PR was accepted same day by LangChain team.

---

## The Vision: Continuous Agent Improvement

### Current State of the Industry

- Most teams don't run optimization regularly (maybe during initial dev, not ongoing).
- Eval suites aren't growing fast enough.
- Agents are **stale after deployment** — they don't improve unless you actively intervene.

### The Future State (6–18 Months)

Modeled after ML retraining pipelines:

1. Agent runs in production, collecting feedback (user feedback, LLM-as-judge evaluators, flagged edge cases).
2. Optional human-in-the-loop to validate/improve data quality.
3. Data feeds into growing eval dataset.
4. Nightly/weekly **automated re-optimization** of agent configuration.
5. New configurations tested via **canary/A/B deployments**.
6. Production performance correlated back to eval suite performance.

### Barriers to Getting There

- **Data quality:** Eval samples must be ground truth — you can't put garbage in.
- **Operational complexity:** Dataset management, user/customer segmentation, deployment models.
- **Algorithm maturity:** Optimization algorithms are promising but still early.
- **Deployment model:** How do you hot-swap prompts without full redeployment?

---

## Prompt & Configuration Management

### The Debate: Prompts as Code vs. Prompts as Data

- **Code argument:** Version control matters, prompts shape core behavior, there's coupling between prompt content and parsing code.
- **Data argument:** Different lifecycle, optimized/changed independently, managed by non-engineers (PMs).
- **Reality:** It's both. The emerging approach is a **configuration registry** (what OPIK calls "blueprints").

### Blueprints: A Configuration Management Approach

- A blueprint contains **all prompts + all configuration variables** for an agent (LLM selection, tool configs, etc.).
- Fetched at runtime via SDK: `opik.getPrompt("prompt_name", "latest")`
- Supports versioning, rollback, access control.
- Enables hot-swapping without full redeployment.
- Allows PMs to manage agent behavior while engineering builds the harness.
- Opens the door to A/B testing, canary deployments, overnight optimization.

### Analogy to ML Model Registries

Just as ML teams use model registries (wrapper around object stores) to decouple model artifacts from application deployment, agent teams can use prompt/config registries to decouple agent behavior from application code.

---

## The Future of Agent UIs

### Intent-Based Interfaces

- LLMs enable moving from **imperative UIs** (click this, drag that) to **intent-based UIs** (express what you want, system figures out the how).
- This doesn't require a chatbot — fuzzy interpretation can enhance any interface.

### Dynamic/Generative UIs

- UIs that change based on the context of the agent session.
- Some interactions are better as text, others as tables/buttons/filters.
- The UI and agent session need to stay in sync.
- Evaluating generated UIs is an open problem (likely DOM-level testing, possibly through sandboxed Redux-style architectures).

### Open Questions

- Will every product expose an MCP and users interact through their own agent?
- Will UIs be generated on-demand to show exactly what's needed?
- What's the role of traditional UX design in an agent-first world?

---

## Practical Advice for Teams Building Agents

### 1. Don't over-invest in frameworks

> "80% of the people I see are not using any [agent framework]. Some of the most successful ones out there are kind of home-brewed, vanilla built."

Don't spend months evaluating frameworks. Start building.

### 2. Build a tiny eval dataset — just 20 samples

> "Spend some time on building a very small evaluation data set, 20 samples, just 20. It will pay off big time."

Even a minimal eval suite dramatically improves your ability to iterate, debug, and optimize. Do this before anything else.

### 3. Use the best model first, optimize costs later

> "Use the frontier model, make it work first, and then you can figure out, can I make this work with a cheaper, smaller model?"

Cost per token drops ~90% year over year. Premature cost optimization is the wrong priority. Get it working, then optimize.

### 4. Don't feel behind — nobody has this figured out

> "Everyone in the industry is trying to figure this out, and it's hard for everyone, including OpenAI."

Even bleeding-edge teams and the frontier labs themselves are still grappling with the same challenges. The field is roughly a year old. There are no established best practices yet.

---

## Key Takeaways

1. **Agent development is a hybrid discipline** — it sits between software engineering and ML, and the best teams draw practices from both worlds.

2. **Evals are the single biggest unlock** — teams that invest in even small eval datasets are significantly more successful at shipping working agents. The challenge isn't algorithmic; it's a product/workflow problem of making eval creation easy.

3. **Prompt optimization is a search problem, not a debugging problem** — the manual inspect-trace-tweak loop is the "manual weight setting" of our era. Automated optimization using LLMs in the loop already produces dramatic improvements with small datasets and minimal cost.

4. **The fundamental equation is broken without feedback loops** — unlike ML models that improve with more data, deployed agents are stale by default. Closing the loop from production feedback → eval dataset → automated optimization is the key architectural challenge.

5. **Configuration management is the emerging infrastructure pattern** — treating prompts, tool configs, and agent parameters as managed configurations (not baked into code) enables A/B testing, canary deployments, hot-swapping, and automated optimization.

6. **Start small, iterate fast** — 20 eval samples, a frontier model, and no framework will get you further than months of infrastructure planning.
