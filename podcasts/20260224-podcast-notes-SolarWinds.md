# Software Engineering Daily — Episode 1899: SolarWinds
## Rethinking Observability in the Age of AI

**Guest:** Krishna Sai, Chief Technology Officer at SolarWinds
**Host:** Sean Falconer
**Source:** Software Engineering Daily

---

## Episode Summary

Krishna Sai, CTO of SolarWinds, discusses how the company is evolving from traditional network monitoring into an AI-driven observability platform. The conversation covers how agentic AI is reshaping incident response and operations, the architectural decisions required to build trustworthy autonomous systems for mission-critical environments, the impact of AI-assisted programming on engineering workflows, and the shifting role of software engineers in a world where context engineering replaces hand-written logic.

---

## SolarWinds Today

SolarWinds' product portfolio spans three core domains: **observability**, **incident response**, and **service management**. IT and ops teams use these tools to detect and remediate issues across network infrastructure, applications, databases, containers, and ML workloads.

The company provides both horizontal coverage (compute, storage, network health) and vertical crosscutting concerns (performance, reliability, cost, security). Customers are ultimately accountable for SLAs and SLOs, but the environments they manage have become so complex that even CIOs struggle to enumerate everything that contributes to a given SLA.

SolarWinds' overarching goal is to **reduce operational toil** — the 3 AM alert storms, war rooms, and manual dashboard triage that still dominate most operations teams' experience.

---

## The Evolution from Monitoring to Agentic AI

Sai outlines a clear generational arc in operational tooling:

1. **Monitoring (Gen 1):** Polling-based systems with manual thresholds. Humans interpret and act.
2. **Observability (Gen 2):** Systems express their own state through metrics, events, logs, and traces (MELT). Correlation across multiple signals becomes possible.
3. **AIOps (Gen 3):** Statistical and ML-based correlation of signals — anomaly detection, pattern recognition. Still largely reactive.
4. **Copilots (Gen 4):** Generative AI as an assistive interface layer. Copilots summarize errors, help write queries, and provide contextual prompts during incidents. Useful, but fundamentally reactive — they wait for a human to engage.
5. **Agentic AI (Gen 5 — emerging):** Autonomous agents that observe, reason, correlate, and act within defined boundaries. They work in the background, loop humans in at critical decision points, and can take action proactively.

The driving force behind this evolution is **exploding complexity** — microservices, distributed systems, tool sprawl, and massive data volumes have reached a point where humans alone cannot maintain system health at scale.

---

## AI-Assisted Programming at SolarWinds

All SolarWinds engineers use AI-assisted coding tools, including both copilots and coding agents. Key observations from their rollout:

- **Commit velocity** has increased 25–30%+.
- **Deployment frequency** has risen and **lead times** have improved.
- **Acceptance rates** for AI-generated code have significantly improved over the past year as models and agents have matured.
- The **bottleneck has shifted to code review**, which is now the constraining factor — a problem they are actively working to address.
- Ongoing concerns remain around **code quality, flaky test generation, and security guideline adherence**.

Sai draws a deliberate parallel between coding agents and operational agents: both involve setting an intent and letting the system decide what actions to take. A coding agent that reads a repo, inspects dependencies, edits multiple files, and runs tests operates on the same mental model as an operational agent that observes error rates, correlates them to a recent deployment, and proposes a rollback.

---

## Architectural Design Principles: "AI by Design"

SolarWinds uses the phrase **"AI by design"** (not "AI first" or "AI everywhere") to describe an approach where the platform is built from the ground up with the assumption that AI-driven components will exist, evolve, and eventually operate autonomously. The key insight: **the model can propose, but the platform must dispose.** Treat the model as a powerful reasoning component, but make the platform the safety boundary.

### Three-Plane Architecture

SolarWinds organizes their platform into three planes, mirroring patterns from systems like Kubernetes:

- **Data Plane:** Handles MELT data ingestion (metrics, events, logs, traces), topology, and normalization.
- **Control Plane:** Manages policies, actions, and execution with enforced least-privilege access.
- **Reasoning Plane:** Where intelligence and agents operate. Explicitly permission-based by design. Models explore hypotheses and propose actions here, but mutations always flow through the control plane.

### Key Design Decisions

**LLM Gateway:** A centralized platform service that handles model selection and abstraction, PII masking, rate limiting, auditability, and policy enforcement. This prevents teams from hardcoding models, avoids vendor lock-in, and provides a single control point for expanding AI use cases.

**Data Preprocessing for AI:** Raw MELT data — especially logs — cannot be thrown directly at LLMs. Logs are noisy and voluminous. SolarWinds compresses, deduplicates, and summarizes log data before exposing it to the reasoning layer, producing compact representations of what changed, what's anomalous, and what's new.

**Tiered Autonomy:** Autonomy is not a binary switch. Every agentic action has an autonomy level that can be tuned by action type, environment, service, or team:
- Level 1 — Recommend only (no mutations)
- Level 2 — Execute with human approval
- Level 3 — Execute autonomously within constraints (for low-risk, high-confidence use cases)

**Transaction Traceability:** First-class audit trails for all agentic actions — what happened, why, and under whose authority. Essential for debugging when things go wrong and for governance at scale.

**Observability for Autonomy:** Making observability of the AI system itself a first-class concern, with activity underway in communities like OpenTelemetry to support this.

---

## The Left-Brain / Right-Brain Analogy

SolarWinds internally uses a brain analogy to frame their system design:

- **Left brain (observability systems):** Optimized for mean time to detect. Ingests signals, identifies anomalies, surfaces what matters.
- **Right brain (action systems):** Runbook automations, workflows, and remediation — optimized for resolution.
- **Subconscious:** Background processing that filters and correlates massive signal volumes.
- **Conscious:** The human decision layer — and increasingly, the layer where autonomous agents operate at varying levels of autonomy.

The analogy mirrors how the human brain processes sensory input: the subconscious filters enormous volumes of data and only surfaces actionable signals to conscious attention.

---

## Real-World Agent Examples

### Configuration Agent
Configuration is one of the highest-leverage, highest-risk surfaces in modern systems. DNS misconfigurations, certificate expirations, and overly permissive security rules often cause subtle degradation rather than outright crashes — the system executes exactly as configured, but behavior degrades. A copilot would discover this after the fact. The configuration agent continuously monitors for service degradation, correlates it to config changes, and can propose rollbacks with a human in the loop.

### ITSM Ticket Summarization Agent
In SolarWinds' service management product, when a support ticket arrives (often forwarded 15+ times), an agent reviews the full incident history, generates a context summary for the human agent (customer sensitivity, prior reviewer notes, previous incident data), and suggests a response that the human can quickly edit and send. This use case saw **instant adoption** and improved mean time to resolution by **30–50% overnight**.

---

## The Shifting Role of Engineers

Sai describes what's happening as more than a tool shift — it's a **responsibility shift**. The role of an engineer is evolving from being the author of deterministic logic to being the engineer of context for probabilistic systems.

Traditionally, responsibility was clear: you wrote the code, it broke, you owned it. With agentic systems, responsibility moves earlier in the process and becomes probabilistic. Engineers must learn to be comfortable with nondeterminism, which is a fundamentally uncomfortable shift for people trained in precise, deterministic thinking.

**Emotional resilience** is an underappreciated factor. The engineers who succeed with these systems are those who treat unexpected AI outputs as feedback ("what context is missing?") rather than failure ("this is garbage, I'll write it myself"). Agentic systems improve through iteration, not one-shot correctness.

### Junior vs. Senior Engineers
Both groups bring distinct strengths. Junior developers tend to adapt faster to new AI tools and get skilled at prompting and steering systems. Senior developers bring the architectural discipline and operational fundamentals that determine whether agents become force multipliers or sources of chaos. Internal AI communities where both groups share learnings have been highly effective at SolarWinds.

---

## Measuring ROI

Justifying AI tool investment remains a common challenge. Sai's approach:

- **Start where metrics are clear.** Customer service is highly metrics-driven — ticket deflection rate, time to first response, and time to resolution are easy to measure and directly tied to business outcomes.
- **Engineering productivity is harder.** Commit velocity and acceptance rates are useful early signals, but tying them to business outcomes remains an unsolved problem for the industry.
- **Focus on DORA-style metrics** like lead time improvement and change failure rate decrease as proxies that connect engineering activity to business value.
- **No matter how powerful the models get**, you still need to do the foundational work of defining what success looks like and putting the right metrics in place to measure it.

---

## Key Takeaways

1. **Agentic AI is the next evolution of observability**, moving beyond dashboards and copilots toward ambient agents that detect, reason, and act proactively — looping humans in only at critical decision points.

2. **Platform architecture matters more than model capability.** Brittle guardrails around powerful models produce disappointing results. Invest in LLM gateways, data preprocessing, tiered autonomy, audit trails, and a clear separation between reasoning and execution.

3. **"The model can propose, but the platform must dispose."** Treat LLMs as reasoning engines. Enforce safety, permissions, and least privilege at the platform level, not the model level.

4. **Data must be refined before it reaches models.** Raw logs and metrics are too noisy and expensive to feed directly into LLMs. Compress, deduplicate, and summarize into compact representations of what changed and what's anomalous.

5. **Autonomy should be graduated, not binary.** Build tiered autonomy levels (recommend → execute with approval → execute autonomously) tunable by action type, environment, service, and team.

6. **The engineer's role is shifting from writing logic to engineering context.** This is a probabilistic discipline, not a deterministic one, and requires both architectural thinking and emotional resilience.

7. **Coding agents are a training ground for operational agents.** The mental model is the same — set intent, let the system decide actions — making coding a useful proxy for building organizational muscle around agentic workflows.

8. **Start ROI measurement where metrics are already clear** (customer service, ITSM) and use DORA-style engineering metrics as proxies while the industry matures its ability to connect productivity gains to business outcomes.

9. **Build internal AI communities.** The learning curve is steep and the landscape changes rapidly. Shared knowledge between junior and senior engineers accelerates adoption and establishes best practices.

10. **Observability of AI systems themselves is a first-class requirement.** If you're deploying agents that act on production systems, you need full traceability of what they did, why, and under whose authority.
