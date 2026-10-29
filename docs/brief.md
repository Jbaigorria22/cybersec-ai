# CyberSec AI — Project Brief

> **Version:** 2.0 · **Date:** 2026-10-29 · **Project status:** Stage A — Design

## Changelog

- **2.0 (2026-10-29):** Roadmap reordered. AI agents, MCP, and AWS deployment now arrive in weeks, not at the end. Phase 1 is replaced by five short stages (A–E). The first user is a technical user; the SMB view moves to a later stage. Only one detection rule is required before AI work starts.
- **1.1 (2026-10-29):** Initial brief.

## 1. Project Vision

**CyberSec AI** is a cloud platform where AI agents, deployed on AWS, help people understand what is happening in their AWS environment: detect possible threats, investigate incidents, and receive recommendations grounded in evidence.

The core idea:

- **Deterministic rules detect.** They are reproducible, testable, and measurable.
- **AI agents investigate and explain.** They work only on findings the rules produced, using read-only tools.

This is not simply about connecting an LLM to security logs. The value lies in combining verifiable detections, traceable evidence, and intelligent assistance.

### Main Objectives

- Detect suspicious behavior in AWS CloudTrail events with explicit rules.
- Build an AI agent that investigates findings, separates facts from hypotheses, and recommends next steps.
- Expose investigation tools through MCP once more than one client needs them.
- Deploy the platform on AWS with infrastructure as code (Terraform).
- Build a portfolio that demonstrates Cloud Engineering, AI Engineering, and Cybersecurity skills.

### Positioning Relative to Previous Projects

| Project | Focus | Relationship to CyberSec AI |
|---|---|---|
| **Guard-RAG** | *Security of AI*: protecting an AI application | Its prompt-injection defenses and evaluation practices are reused |
| **ThreatScribe** | Detection fundamentals on logs | Paused; its lessons are folded into Stage A |
| **CyberSec AI** | *AI for Security*: using AI agents to investigate incidents in AWS | Current project |

## 2. Target User

**First version:** a technical user (the builder) who wants to understand what happened in an AWS account. Building for oneself first keeps feedback fast and requirements concrete.

The platform should help answer:

- What happened, who did it, when, and from where?
- Why was this alert generated, and what evidence supports it?
- What should be investigated next?
- How reliable is the conclusion, and what information is missing?

**Later (Stage E):** a plain-language view for small and medium-sized business owners: "Am I at risk, and what should I do?" It reuses the same findings and investigations, presented without technical jargon.

## 3. Roadmap

Each stage is short, produces something that works, and teaches a specific skill set. A technology enters only when its stage needs it.

| Stage | What gets built | What it teaches | Estimate |
|---|---|---|---|
| **A. Minimal engine** | Ingestion, normalization, **one rule** (root account usage), labeled dataset, tests | Python, Pydantic, testing | 1–2 weeks |
| **B. Investigator agent** | An agent that receives a finding, calls read-only tools, and returns a structured, evidence-linked explanation | LLMs, tool use, structured outputs, evaluation | 2–3 weeks |
| **C. MCP** | The agent's tools exposed as an MCP server | MCP, interoperability | 1–2 weeks |
| **D. AWS deployment** | The platform running on AWS, provisioned with Terraform | Cloud architecture, IAM, cost control | 2–3 weeks |
| **E. Product** | More rules (console brute force, CloudTrail tampering), SMB view, web UI | Product thinking, UX | Later |

Estimates are guides, not deadlines. A stage ends when its definition of done is met.

## 4. Stage A Scope: Minimal Engine

### Features

1. **Ingestion.** Read CloudTrail JSON files (`{"Records": [...]}`, as delivered to S3). Synthetic data only. Invalid records are rejected and reported without stopping the rest.
2. **Normalization.** Convert each record into a `NormalizedEvent`, so rules never depend on raw CloudTrail JSON.
3. **Detection.** One rule: **root account usage**, noting MFA usage when available. Based on the CIS AWS Foundations Benchmark.
4. **Findings.** Each detection produces a structured finding: rule ID, severity, rationale, related event IDs, evidence, and timestamp.
5. **Evaluation.** A small labeled dataset, with labels stored separately from events. Report precision and recall for the rule.

### Out of Scope for Stage A

- AI components (they start in Stage B).
- Connections to real AWS accounts.
- Real-time processing.
- Web interface.

### Definition of Done (Stage A)

1. The root-usage rule is implemented, documented, and tested (positive and negative cases).
2. A small, labeled, reproducible synthetic dataset exists, including realistic false-positive cases.
3. Precision and recall are reported for the rule.
4. Every finding can be traced back to its source events.
5. The README and ADRs describe the decisions and known limitations.

## 5. Conceptual Architecture

```
CloudTrail JSON
      │
Ingestion & Validation
      │
Normalization ──────────► NormalizedEvent
      │
Detection Engine (deterministic rules)
      │
Findings & Evidence ────► Evaluation ◄── Labels (ground truth)
      │
Investigator Agent (Stage B) ──► Read-only tools (MCP in Stage C)
      │
Application API
      │
Interface (Stage E)
```

Each stage receives and returns data under a defined contract. The agent sits **after** Findings: it never decides whether a detection occurred.

Locally first (Stages A–C), then AWS (Stage D). AWS services are chosen in an ADR based on real requirements.

## 6. Role of AI

### What the agent does

- Explains why a finding deserves attention.
- Gathers context with read-only tools (for example, other recent activity by the same identity).
- Separates observed facts from hypotheses.
- Identifies missing information.
- Recommends next investigation steps.

### Limits and Guarantees

- **AI never detects.** It only works on findings the engine produced.
- The agent must not invent evidence or present hypotheses as facts. Every claim links to an event or finding.
- Structured outputs are validated before the application uses them.
- Logs are untrusted data. Fields such as `userAgent` or `requestParameters` can be controlled by an attacker, so they are never treated as instructions.
- Tools are read-only, with minimal permissions and an explicit scope.
- Any AI component must show, with metrics, an improvement over the non-AI baseline.
- Start with **one** agent. Multiple agents are added only if one agent measurably falls short.

## 7. Technology by Stage

| Stage | Technologies |
|---|---|
| A | Python, Pydantic, Pytest, Ruff |
| B | An LLM API, structured outputs, an evaluation harness (judge from a different model family than the generator) |
| C | MCP |
| D | AWS (services decided by ADR), IAM least privilege, Terraform, secrets management |
| E | Web framework (decided by ADR) |

Frameworks such as LangChain or LangGraph are added only when a stage shows a measurable need.

## 8. Visual Reference

A third-party cybersecurity dashboard design is kept as a private reference, **not** included in the public repository. It inspires the long-term look (dark theme, violet accents, cards, clear hierarchy), not a literal copy.

> **Visual honesty rule:** the interface only shows metrics and features that actually exist. Quality metrics (precision, recall) only exist on labeled data, so they are never presented as live measurements.

Design principles: readability over effects; color communicates severity and is always paired with text or icons; evidence is always one click away.

## 9. Security, Privacy, and Reliability

- Synthetic data during development.
- No credentials or secrets in the repository.
- Validate records before processing.
- Do not send full logs to the LLM when not necessary.
- Keep traceability between findings and original events.
- Least privilege for services and tools.
- Read-only: never execute destructive actions or modify resources autonomously.
- Document limitations, assumptions, and known false positives.

## 10. Known Risks

| Risk | Mitigation |
|---|---|
| A synthetic dataset that is too "easy" | Include ambiguous cases and realistic false positives |
| Losing motivation before reaching the interesting parts | Short stages; AI work starts in Stage B |
| The agent invents evidence or claims | Structured outputs, claim-to-evidence links, faithfulness evaluation |
| Prompt injection through log fields | Logs treated as untrusted data; agent receives findings, not raw logs |
| Uncontrolled scope growth | Definition of done per stage |
| Unexpected AWS costs | Local first; ephemeral infrastructure with Terraform |

## 11. Way of Working

Before implementing anything in a stage:

1. Explain the problem.
2. Define the expected outcome.
3. Compare alternatives.
4. Choose and justify.
5. Record the decision in an ADR (`docs/adr/`).
6. Implement a small, understandable unit.
7. Test it and document its limitations.

Documentation and code are in English. Design discussions may happen in Spanish.

AI assistants act as technical mentors: they explain the purpose of the code and the decisions behind it, and they do not add frameworks without a clear objective.

## 12. Guiding Principle

**Rules detect. Agents investigate. Evidence decides.**

Build a small, reliable engine first, then reach AI agents quickly, and grow from verifiable results rather than unnecessary complexity.
