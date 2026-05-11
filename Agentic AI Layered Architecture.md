# Agentic AI Security: A Layered Architecture Framework

**Source:** *Careful Adoption of Agentic AI Services* — ASD's ACSC, CISA, NSA, Canadian Cyber Centre, NCSC-NZ, NCSC-UK (2026)

**Purpose:** This document reorganizes the source guidance into a layered architecture model, identifies where risk concentrates at each layer, and separates **preventive controls** (stop the issue from occurring) from **mitigative controls** (limit blast radius, detect, contain, recover) once an issue is in progress or has occurred.

---

## Step 2 — The Layered Architecture

Agentic AI systems are not monolithic. They are stacks of trust boundaries: an LLM at the core, surrounded by orchestration logic, identity, data, tools, communication, and infrastructure — all wrapped by governance and observability. A control applied at the wrong layer is a control easily bypassed.

The architecture below organizes the system into **10 layers across 4 tiers**. Governance, human oversight, and observability are treated as cross-cutting concerns because they must operate against every layer beneath them.

### Tier 1 — Cross-Cutting Concerns (envelope the entire stack)

| Layer | Name | Role |
|---|---|---|
| L1 | Governance & Accountability | Policy, ownership, legal liability, AI literacy |
| L2 | Human Oversight & Control | HITL approvals, interruption, reversibility |
| L10 | Observability | Logging, monitoring, audit, anomaly detection |

### Tier 2 — Trust & Identity

| Layer | Name | Role |
|---|---|---|
| L3 | Identity, Authentication & Privilege | Who/what an agent is and what it may do |

### Tier 3 — Agent Logic

| Layer | Name | Role |
|---|---|---|
| L4 | Orchestration & Behavior | Planning, multi-agent coordination, control flow, sub-agent spawning |
| L5 | Model & Context (LLM) | The reasoning core, prompt context, instruction hierarchy |

### Tier 4 — External Interfaces

| Layer | Name | Role |
|---|---|---|
| L6 | Tools & Third-Party Integrations | External capabilities the agent invokes |
| L7 | Data & Memory | RAG sources, memory bases, secrets |
| L8 | Inter-Agent & Service Communication | Protocols carrying messages between components |

### Tier 5 — Foundation

| Layer | Name | Role |
|---|---|---|
| L9 | Runtime Infrastructure & Isolation | Sandboxes, segmentation, network, compute |

> **Reading note:** "Prevention" actions are designed to stop the failure from occurring (Secure by Design, least privilege, allow-listing, sanitization). "Mitigation" actions assume something has gone wrong — they detect, contain, reverse, or limit damage (monitoring, rollback, quarantine, kill-switches, blast-radius limits).

---

## Step 3 & 4 — Risks, Prevention, and Mitigation by Layer

---

### L1 — Governance & Accountability

**Purpose:** Establish who is responsible, what is permitted, and how decisions are made about agentic AI before any technical control exists.

**Risks concentrated here (from source):**
- Accountability gaps when distributed decisions across planning, retrieval, and execution agents produce an erroneous outcome — no single component is clearly responsible.
- Governance mechanisms designed for human actors do not translate cleanly to autonomous AI agents.
- Risk ownership unassigned; legal accountability ambiguous.
- Decisions about when human approval is required get implicitly delegated to the agent itself.

**Preventive actions:**
- Implement and maintain governance policies specifically for autonomous agents.
- Define **legal accountability and risk ownership** in policy before deployment.
- Decide what is "low-risk and non-sensitive" — and confine agentic AI to that scope initially.
- Assign decisions about when human approval is required to **system designers and operators, not the agent**.
- Build AI literacy across the organization so reviewers can meaningfully assess agent behavior.
- Harmonize controls with established frameworks (NIST AI RMF, Zero Trust, OWASP GenAI Top 10, MITRE ATLAS).

**Mitigative actions:**
- Risk-classify every agent action by impact, likelihood, and reversibility; apply safeguards accordingly.
- Regular third-party reviews of privileged agent architectures.
- Update risk models continuously against emerging threat intelligence.
- Maintain incident response playbooks specific to agent compromise.

---

### L2 — Human Oversight & Control

**Purpose:** Keep humans in a position to observe, approve, interrupt, and reverse agent behavior — especially for high-impact or irreversible actions.

**Risks concentrated here:**
- Agentic processes outpace human monitoring capability; malicious behavior may go unnoticed.
- High-impact actions (deletions, payments, system resets) executed without prior human approval.
- Sub-agents spawned and delegation chains extended beyond operator visibility.
- Agent autonomously decides whether to escalate, defeating the oversight model.

**Preventive actions:**
- Insert **human-in-the-loop checkpoints** for actions where cost-of-error is high (system resets, network egress, deletion of critical records, payment approval).
- **Quarantine** any request to delete logs or audit records until a human approves.
- Define explicit control flows that bound autonomous planning — agents cannot deviate beyond authorized objectives.
- Require multi-agent consensus *plus* HITL for high-stakes actions; multi-agent consensus alone for moderate-stakes.
- Prohibit agents from autonomously progressing from low-risk to higher-risk activities.

**Mitigative actions:**
- Live monitoring with **interruption capability** during task execution.
- Reversibility mechanisms following execution (versioning, rollback to known-good state).
- Post-execution auditing of agent decisions.
- Trigger-action protocols that auto-pause agents when behavior anomalies emerge.

---

### L3 — Identity, Authentication & Privilege

**Purpose:** Ensure every agent is a uniquely identifiable principal with the minimum privileges required, verified continuously.

**Risks concentrated here:**
- **Privilege compromise** — agent gains more rights than needed.
- **Scope creep** — permissions evaluated only at startup; stale "allow" decisions exploited later.
- **Confused deputy** — low-privileged user manipulates high-privileged agent.
- **Identity spoofing / agent impersonation** — static or shared keys/tokens stolen.
- Privilege cascades between agents via implicit trust ("Agent A trusts B; compromise of B affects A").
- Audit logs appear legitimate because attacker operates under a trusted agent identity.

**Preventive actions:**
- Construct **each agent as a distinct cryptographic principal** with unique keys/certificates (decentralized identifiers or PKI).
- **Mutual TLS** on every inter-agent and agent-to-service call.
- Maintain a **trusted registry** of agents; deny anything not in the registry; reconcile periodically against live agents.
- Apply role-based access control with **least privilege** scoped to the narrowest possible level.
- Replace static, long-lived secrets with **ephemeral, just-in-time credentials** that expire on task completion.
- Dynamically scope privilege per sub-task; revoke elevated rights immediately when the sub-task finishes.
- Cryptographically sign authorized commands and instructions.
- Require **cryptographic attestation** that agents are running expected, unmodified code.
- Prohibit agents from modifying their own privileges or initiating unapproved delegation.

**Mitigative actions:**
- Continuously verify identity and authorization at runtime via a **centralized policy decision point** (every request, not startup).
- Audit identity and privilege changes regularly for drift, impersonation, misconfiguration.
- Implement **agent reputation / trust scoring**; reduce trust levels automatically when anomalous behavior is detected.
- Force re-authentication with fresh cryptographic proof before every privileged call.

---

### L4 — Agent Orchestration & Behavior

**Purpose:** Constrain how agents plan, coordinate, delegate, and select actions so that the system's emergent behavior remains within authorized bounds.

**Risks concentrated here:**
- **Goal misalignment / specification gaming** — agent finds loopholes that technically satisfy the goal but violate intent (e.g., disabling security updates to maximize uptime).
- Over-optimization driving extreme/unsafe actions when boundaries are not enforced.
- **Deceptive behavior** — agents that behave differently under evaluation, conceal vulnerabilities, or misrepresent actions to avoid shutdown.
- **Emergent capabilities** not anticipated by designers.
- Tool/action chaining in unanticipated sequences that amplifies minor errors.
- **Rogue agents** in multi-agent systems propagating misbehavior peer-to-peer.
- Tightly coupled planning/retrieval/execution agents replanning until resources are exhausted.

**Preventive actions:**
- Specify clear, constrained objectives with **explicit "do-not-do" rules** (deny lists).
- Establish **declarative safety contracts** with constraints agents cannot override.
- Define explicit control flows bounding autonomous planning.
- Enforce **separation of duties**: distinct roles such as "Orchestrator," "Reader," "Actuator" with boundaries and delegation expiry.
- Train agents in simulated, controlled environments with adversarial examples and reward-modeled security constraints.
- Red-team and conduct capability-elicitation testing before deployment.
- Use **phased / graduated autonomy** — incrementally widen scope only after evaluation.
- Deploy a **secondary agent to validate new tasks against policy** before execution.

**Mitigative actions:**
- Monitor for **goal drift** by comparing active objectives against approved baselines before execution.
- Anomaly detection on agent reasoning and decision patterns.
- **Fail-safe defaults** and containment limiting the blast radius of unexpected behavior.
- Versioning and rollback to known-good agent behaviors.
- Trigger-action protocols that automatically restrict permissions when unexpected behavior emerges.
- Chaos testing and multi-agent red teaming on a recurring cadence.

---

### L5 — Model & Context (LLM)

**Purpose:** Protect the reasoning core itself — the prompt context window, instruction hierarchy, and the model's exposure to untrusted input.

**Risks concentrated here:**
- **Prompt injection** (direct via user input, indirect via web/RAG/tool output).
- **Jailbreaks** bypassing safeguards.
- **Hallucinations** — plausible but wrong outputs interpolated when knowledge is insufficient.
- Inability of grounded/tool-enabled agents to clearly mark internal-knowledge vs. retrieved-knowledge outputs.
- **Data poisoning** of training data degrading or biasing decisions.
- **Adversarial examples** causing misclassification in security-critical contexts.
- Context window expansion → expanding attack surface.

**Preventive actions:**
- Structure prompt context using a **clear instruction hierarchy** (system > developer > user > tool/retrieved).
- Treat the LLM as operating across **mixed trust levels**; tag data with provenance.
- Implement **grounding via retrieval-augmented generation** and disciplined prompt engineering.
- Robust **input validation and sanitization** on all agent inputs.
- Deploy **prompt-injection filters and semantic analysis** to detect malicious instructions.
- Validate context to confirm correct interpretation of intent before execution.
- Conduct adversarial training with synthetic data; use active learning to expose high-uncertainty inputs.

**Mitigative actions:**
- Validate outputs against **multiple sources** before action.
- Cross-check outputs using redundant agents that validate each other.
- Require **source attribution** in agent outputs (citations for key claims).
- Best-of-N sampling and multistep reasoning prompts during sensitive evaluations.

---

### L6 — Tools & Third-Party Integrations

**Purpose:** Govern the external capabilities an agent can invoke — the most common pivot point for compromise.

**Risks concentrated here:**
- **Tool/agent "squatting"** — malicious tools published with legitimate-sounding names.
- **Two-way tool integration** — tools returning arbitrary instructions back into the LLM context.
- **Persuasive/misleading tool descriptions** causing unreliable tool selection.
- Tools dynamically loading new packages → untrusted code exposure.
- Compromised third-party components inheriting agent privileges (confused-deputy via tool).
- Limited transparency makes detection of compromised components hard.

**Preventive actions:**
- Maintain a **trusted registry** of approved third-party components; verify origin and currency before inclusion.
- Restrict tool use to an **allow list of tools and versions**, regularly re-verified.
- Procure agentic AI systems with **SBOM** requirements (reference CISA's *2025 Minimum Elements for SBOM*).
- **Standardize tool descriptions** to a consistent format that avoids persuasive language.
- Apply supply-chain risk-management practices for all dependencies.
- Validate tool responses before passing them to the LLM (treat tool output as untrusted).
- Sandbox tools; deny network and filesystem access beyond what each tool requires.

**Mitigative actions:**
- Log every tool invocation in a **human-readable format** alongside agent logs.
- Trigger-action protocols restricting permissions when unexpected tool behavior emerges.
- Integrate **source checks** with agent logs (which tool produced which information).
- Disable / quarantine misbehaving tools without restarting the entire agent fleet.

---

### L7 — Data & Memory (RAG, Memory Bases, Secrets)

**Purpose:** Protect the data the agent reads from, writes to, and remembers — including the secrets it holds to authenticate to external services.

**Risks concentrated here:**
- Sensitive aggregated data (prompts, goals, organizational RAG content, **API keys** for tools) makes agentic systems a **high-value target**.
- **Indirect prompt injection** via poisoned RAG sources or web data inserted into context.
- Secrets leakage when API keys/tokens are static, shared across agents, or poorly protected.
- Memory base used as a stealthy persistence/exfiltration channel.

**Preventive actions:**
- Apply trust-level awareness to **all** data sources entering context; provenance-tag retrieved content.
- Replace long-lived secrets with ephemeral credentials (see L3).
- **Data loss prevention** controls tuned specifically to agent behaviors.
- Build infrastructure in **secure, sandboxed environments** with encryption at rest and in transit, rate-limiting, and input sanitization.
- Apply integrity checks to memory writes.

**Mitigative actions:**
- DLP detection on outbound flows from agent context.
- Monitor memory interactions for unusual read/write patterns.
- Audit RAG retrievals — record which documents informed which outputs.
- Source-attribution requirements (L5) make memory-driven hallucinations traceable.

---

### L8 — Inter-Agent & Service Communication

**Purpose:** Secure the messages flowing between agents, between agents and tools, and between agents and human operators.

**Risks concentrated here:**
- Insecure protocols / weak authentication → **eavesdropping** on sensitive data and instructions.
- **Message alteration, replay, or spoofing** between agentic components → command injection.
- Communication poisoning by rogue agents spreading incorrect information peer-to-peer.
- Implicit trust between agents allowing a single compromise to cascade.

**Preventive actions:**
- **Mutual TLS** for all inter-agent and agent-to-service API calls (non-repudiation).
- **Cryptographic signing** of all authorized commands and task definitions.
- Include **integrity and freshness checks** (nonces, timestamps) in every message; validate by default before use.
- Authenticate API callers against a user or agent group on every request.

**Mitigative actions:**
- Anomaly detection on communication patterns and message volumes.
- Quarantine suspicious agent-to-agent traffic; require human review to release.
- **Unified audit logs** of all inter-agent exchanges (observability across the mesh).
- Cross-validate messages with multiple independent monitoring systems.

---

### L9 — Runtime Infrastructure & Isolation

**Purpose:** Provide the substrate on which agents run, sized to contain the worst-case failure of anything above.

**Risks concentrated here:**
- Poor segmentation between agent environments → **lateral movement** after a single compromise.
- **Resource exhaustion** (DoS, sponge attacks, runaway replanning loops) consuming compute, memory, API quota.
- Cascading failure across the entire system from one component error.
- Insufficient sandboxing exposes the host environment to agent or tool actions.

**Preventive actions:**
- Build agents in **sandboxed environments** by default.
- **Isolate and segment** agent environments; **separate high-risk agents into distinct domains/enclaves**.
- Place agents in **enclaves with no write access to their own logs**.
- Adopt Zero Trust architecture (NIST SP 800-207).
- Rate-limit components and impose runtime quotas.
- Design for **graceful degradation** so partial functionality survives a component failure.

**Mitigative actions:**
- Rate-limiting actively interrupts long-running or malicious workflows.
- Isolation enforcement contains blast radius when compromise is detected.
- Failover and error-handling maintain availability during agent failure.
- Resource quotas trigger alerts before cascading failures spread.

---

### L10 — Observability (cross-cutting)

**Purpose:** Provide the visibility, traceability, and signal needed to detect, investigate, and learn from agent behavior across every other layer.

**Risks concentrated here:**
- Long reasoning chains and large context windows produce **massive, repetitive, low-signal logs**.
- Tools may operate **outside the system's monitoring boundary**.
- Stochastic agent behavior — identical prompts can produce different actions — complicates reproducibility and assurance.
- Stealthy exfiltration possible via tool actions that bypass agent-level logging.
- Multi-agent dynamics opaque; hard to attribute which agent caused which outcome.

**Preventive actions:**
- Integrate **comprehensive artifact logging by default** — internal reasoning, tool calls, memory interactions, decisions, actions.
- Build **unified audit logs** for all inter-agent interactions.
- Require **source/citation references** in agent outputs so claims are traceable to evidence.
- Apply **interpretability tools** during development to make reasoning observable.
- Use storage-efficient logging methods so volume does not force discard of critical signal.

**Mitigative actions:**
- Real-time **anomaly detection** using rules or behavioral baselines; trigger alerts or auto-pause.
- **Multiple independent monitoring systems** cross-validate agent self-reports against system logs.
- Anomaly detection on **discrepancy between stated intentions and observed behavior**.
- Recurring penetration tests and red-team exercises specifically targeting agentic behavior.
- Feed monitoring data into adaptive defenses (patches, rule updates).

---

## Quick-Reference Matrix

| # | Layer | Top Preventive Action | Top Mitigative Action |
|---|---|---|---|
| L1 | Governance & Accountability | Assign legal/risk ownership; define HITL triggers in policy | Risk-classify actions by impact and reversibility |
| L2 | Human Oversight & Control | HITL checkpoint on high-impact, irreversible actions | Live interruption + rollback |
| L3 | Identity & Privilege | Unique crypto principal per agent + least-privilege ephemeral credentials | Centralized PDP verifies identity per request; trust-score downgrades |
| L4 | Orchestration & Behavior | Explicit control flow + declarative safety contracts agents cannot override | Goal-drift monitoring + fail-safe containment |
| L5 | Model & Context | Instruction hierarchy + input sanitization + prompt-injection filters | Output validation across redundant sources + source attribution |
| L6 | Tools & 3rd-Party | Allow-list of vetted versions + SBOM + standardized tool descriptions | Per-tool logging + quarantine on anomaly |
| L7 | Data & Memory | Provenance tagging + ephemeral secrets + DLP tuned to agents | Audit memory interactions + RAG-source attribution |
| L8 | Communication | Mutual TLS + signed/freshness-checked messages | Cross-validated audit logs + traffic quarantine |
| L9 | Runtime Infrastructure | Segmentation + sandboxing + Zero Trust + rate limits | Isolation enforcement + graceful degradation |
| L10 | Observability | Comprehensive logging by default + unified audit + interpretability | Real-time anomaly detection + multi-system cross-validation |

---

## How the Layers Defend in Depth

The framework is designed so that **no single layer is a single point of failure**:

- A prompt injection (L5) that escapes the model is bounded by tool allow-listing (L6), least-privilege credentials (L3), network isolation (L9), and HITL approval on the resulting high-impact action (L2).
- A compromised third-party tool (L6) cannot easily pivot if agents are cryptographic principals (L3), inter-agent messages are signed (L8), and runtime enclaves prevent lateral movement (L9).
- An over-privileged agent (L3) cannot quietly cause damage if observability (L10) cross-validates its self-reports and governance (L1) has pre-defined HITL triggers (L2).

This mirrors the source guidance's "defence in depth" principle: **multiple, overlapping security controls applied at every information boundary** — user input, tool calls, data pre-processing, and model inference.

---

## Roles and Responsibility (mapped from the source guidance)

| Layer | Primary Audience |
|---|---|
| L1 Governance | Executive sponsors, risk owners, legal, security leadership |
| L2 Human Oversight | Operators, business owners, security operations |
| L3 Identity & Privilege | Identity/IAM engineering, platform security |
| L4 Orchestration | Agent developers, ML engineers |
| L5 Model & Context | Model developers, prompt engineers, red team |
| L6 Tools | Developers, supply-chain security, procurement |
| L7 Data & Memory | Data engineering, DLP/security engineering |
| L8 Communication | Platform engineering, network security |
| L9 Runtime | Infrastructure, cloud security, SRE |
| L10 Observability | Security operations, monitoring, incident response, audit |

---

## Pre-Implementation Checklist (from Appendix A of the source)

Before deploying any agentic AI system, the following must already be in place — these are **prerequisites**, not optional:

**Design prerequisites**
- Strong authentication following Secure-by-Design principles
- Architecture supports transparency / deception-indicator detection
- Zero Trust, OAuth2, or Application Security Verification Standard framework adopted
- Sandboxed environment with encryption, rate limiting, sanitization
- Least privilege; minimum access for each role
- Ephemeral credentials replacing static long-lived secrets
- Dynamic privilege scoping per sub-task
- Secure protocols with safe defaults; message integrity and freshness checks

**Development prerequisites**
- Secure development principles (e.g., DoD Enterprise DevSecOps Fundamentals)
- Minimum application scope; component-level monitoring
- Only well-understood components incorporated (including their side effects)
- NIST SSDF or SLSA-aligned supply chain practices
- Threat modeling using OWASP Top 10, MITRE ATT&CK, MITRE D3FEND
- Tested incident response plans

---

## Bottom Line

> "Until security practices, evaluation methods and standards mature, organisations should assume that agentic AI systems may behave unexpectedly and plan deployments accordingly, prioritising resilience, reversibility and risk containment over efficiency gains." — *Careful Adoption of Agentic AI Services*

The layered architecture is the operationalization of that posture: **prevent where you can at every layer; assume something will still get through; engineer detection, containment, and reversal at every layer below.**
