# CyberSec AI — Project Brief

> **Version:** 1.1 · **Date:** 2026-10-29 · **Project status:** Phase 1 — Design (Stage 1 of 7)

## 1. Project Vision

**CyberSec AI** is a cybersecurity project that combines deterministic detection, artificial intelligence, and investigation tooling to help security analysts identify, understand, and investigate suspicious activity in cloud environments.

The goal is to evolve from an evidence-based detection engine into an AI-assisted investigation platform, with a professional visual interface, grounded explanations, and controlled investigation workflows.

This is not simply about connecting an LLM to security logs. The project's value lies in combining verifiable detections, traceable evidence, quantitative evaluation, and intelligent assistance.

### Main Objectives

- Detect suspicious behavior from security events.
- Generate reproducible, evidence-backed alerts.
- Use AI to explain findings, separate facts from hypotheses, and suggest investigation steps.
- Progressively add RAG, agents, and security tools with limited permissions.
- Build a cloud-native, secure, scalable application deployable through infrastructure as code.
- Develop a technical portfolio that demonstrates skills in Cloud Engineering, AI Engineering, and Cybersecurity.

### Positioning Relative to Previous Projects

| Project | Focus | Relationship to CyberSec AI |
|---|---|---|
| **Guard-RAG** | *Security of AI*: protecting an AI application | Its prompt-injection defenses and evaluation practices are reused |
| **ThreatScribe** | Detection fundamentals on logs | Paused; its lessons are folded into Phase 1 |
| **CyberSec AI** | *AI for Security*: using AI to investigate incidents in AWS | Current project |

## 2. Target User

The first version targets **security analysts and cybersecurity professionals who need to investigate events and understand alerts faster**.

The platform should help them answer questions such as:

- What happened?
- Which identity performed the action?
- When and from where did it happen?
- Why was this alert generated?
- What evidence supports the detection?
- What should be investigated next?
- How reliable is the conclusion, and what information is missing?

In the future, the project could add an executive view for small and medium-sized business owners, with risk summaries and recommendations that do not require advanced technical knowledge.

## 3. Visual Reference and User Experience

The reference image provided for the project shows a third-party cybersecurity dashboard design. It will be used as **visual and functional inspiration**, not as an interface to be copied literally. The image is kept only as a private reference and **is not included in the public repository**.

The desired visual direction is a modern security application with a dark theme, violet accents, well-organized cards, readable charts, and a clear visual hierarchy.

### Adapting the Reference to CloudTrail

The reference is designed for SaaS applications (Slack, Jira, GitHub). Each component is adapted to what CloudTrail can actually report:

| Reference component | CyberSec AI equivalent | Data source | Available from |
|---|---|---|---|
| Violations Summary | Findings by severity | Detection engine | Phase 1 (initial UI) |
| Event Severity (hourly bars) | Findings by severity over time | Findings + `event_time` | Phase 1 (initial UI) |
| Incident Timeline | Timeline of events behind a finding | Finding evidence | Phase 1 (initial UI) |
| Risk by Application | Findings by AWS service (`event_source`) or by identity | Findings | Phase 1 (initial UI) |
| Event Types | Findings by rule or category (access, evasion, privileges) | Rule catalog | Phase 1 (initial UI) |
| Live Threat Map | Activity by AWS region / source IP | `aws_region`, `source_ip` | Later (requires continuous ingestion) |
| Detection Confidence | Precision and recall **per rule**, measured on labeled data | Evaluation module | Phase 1 (as an evaluation report, not a live metric) |
| Security Query | Conversational querying of findings | AI layer | AI-assistance phase |
| Users / Endpoints / Apps monitored | Observed identities and accounts | Normalized events | Later |

> **Visual honesty rule:** quality metrics (false positives, true positives) only exist when labeled data exists. In production the ground truth is unknown, so the dashboard will never present those metrics as if they were live measurements.

### Desired Visual Elements

- **Overview dashboard:** summary of security posture and alerts.
- **Violations Summary:** number of findings by severity: critical, high, medium, and low.
- **Threat Monitoring:** visualization of suspicious activity and how events evolve over time.
- **Detection Metrics:** detection quality metrics when enough labeled data exists.
- **Incident Timeline:** chronological sequence of the events related to an investigation.
- **Risk by Resource:** distribution of findings by service, identity, or cloud resource.
- **Event Types:** classification of findings by category.
- **Security Query:** a conversational interface to query findings and request help during an investigation.
- **Incident Investigation:** a detailed view with evidence, explanation, hypotheses, confidence level, and recommended actions.

### Design Principles

1. Prioritize readability and analytical usefulness over visual effects.
2. Use color to communicate severity and status, not just as decoration.
3. Keep the interface consistent, with clear navigation and reusable components.
4. Avoid fictional charts, percentages, or indicators that look like real metrics.
5. Always show the evidence behind an alert and allow tracing it back to its source.
6. Design an interface that can grow without rebuilding the whole application.
7. Never rely on color alone to communicate severity: always pair it with text or icons (accessibility).

The visual reference represents the long-term product direction. The MVP does not need to implement all of these components right away. The interface must grow alongside the system's real capabilities.

## 4. Initial Scope: MVP (Phase 1)

The first version focuses on AWS CloudTrail and uses labeled synthetic events to develop and evaluate a reproducible detection engine.

### Initial Features

**1. Event Ingestion**

Read CloudTrail records in JSON format (`{"Records": [...]}` structure, as CloudTrail delivers them to S3). Synthetic datasets will be used initially, without connecting a real AWS account. Invalid records are rejected and reported; they do not stop processing of the rest.

**2. Normalization**

Convert each record into a common `NormalizedEvent` schema, preserving traceability and preventing rules from depending directly on the original JSON structure.

**3. Deterministic Detection**

Implement explicit rules to identify suspicious patterns. The first candidates are:

- Root account usage, with additional evaluation of MFA usage when that information is available.
- Multiple failed console sign-in attempts from the same IP, escalated if a successful sign-in follows.
- Attempts to stop, delete, or modify CloudTrail configuration in ways that could affect monitoring.

Each rule must have documented logic, trigger conditions, severity, known false positives, and automated tests. The rules are based on the monitoring controls of the CIS AWS Foundations Benchmark.

**4. Evidence and Findings**

Each detection must produce a structured finding that includes, at minimum:

- Rule identifier.
- Severity.
- Detection rationale.
- Related events (identifiers).
- Verifiable evidence.
- Timestamp.
- Investigation identifier (optional; to be defined once findings are grouped).

**5. Evaluation**

Create tests and labeled datasets to measure detector quality. Evaluate precision, recall, and false positives per rule. Labels (ground truth) are stored **separately** from events, so the detector can never access them.

**6. Initial Interface**

Once the engine produces reliable results, build a web interface to browse detections, filter findings, and explore their evidence. The design should follow the established visual direction but will only display metrics and features that are actually implemented.

### Out of Scope for Phase 1

- Connections to real AWS accounts.
- Real-time processing.
- AI components (LLM, RAG, agents).
- Multi-tenant support and user roles.
- Automated incident response.

### Definition of Done (Phase 1)

Phase 1 is considered done when:

1. The 3 initial rules are implemented, documented, and tested (positive and negative cases).
2. A labeled, reproducible synthetic dataset exists.
3. Precision and recall are reported per rule.
4. Every finding can be traced back to its source events.
5. The repository includes a README, ADRs, and known limitations.

Anything beyond this point is considered an extension of the project, not a requirement for calling it done.

## 5. Conceptual Architecture

The architecture must separate the responsibilities of processing, detection, explanation, and presentation. Each stage receives and returns data under a defined contract.

```
CloudTrail JSON
      │
Ingestion & Validation
      │
Normalization Layer ──────► NormalizedEvent
      │
Detection Engine (deterministic rules)
      │
Findings & Evidence ──────► Evaluation ◄── Labels (ground truth)
      │
AI Analysis Layer (future: explains, never detects)
      │
Application API
      │
Security Dashboard
```

The initial flow will be implemented locally and in a controlled way. The architecture may evolve toward AWS managed services when there is a concrete need.

The choice of frameworks, services, and architectural patterns must be justified by real requirements, not by the number of technologies that could be added.

## 6. Role of Artificial Intelligence

AI must complement the detection engine, not replace it. **AI never decides whether a detection occurred**: it only works on findings already produced by the engine.

### AI Responsibilities

- Explain why a finding deserves attention.
- Summarize a sequence of related events.
- Separate observed facts from hypotheses.
- Identify missing information.
- Recommend next investigation steps.
- Consult security documentation and procedures in later phases.
- Use authorized investigation tools once the system adds agent capabilities.

### Limits and Guarantees

- The LLM must not invent evidence or present hypotheses as confirmed facts.
- Conclusions must be linked to verifiable events and findings.
- Structured responses must be validated before the application uses them.
- Logs and retrieved documents must be treated as untrusted data, never as privileged instructions. Fields such as `userAgent` or `requestParameters` can be controlled by an attacker.
- Sensitive actions must require human authorization.
- Future agent tools must have minimal permissions and an explicit scope.
- Any AI component must demonstrate, with metrics, an improvement over the non-AI baseline.

## 7. Planned Technology Evolution

Technologies will be added progressively when they solve an identified need.

### Initial Phase: Detection Engine

- Python.
- Pydantic for validation and normalization.
- Pytest for automated testing.
- Ruff for code linting and formatting.
- JSON and synthetic datasets.
- Deterministic rules and evaluation metrics.

### AI-Assistance Phase

- An LLM to generate evidence-based explanations.
- Structured outputs and response validation.
- Evaluations of quality, faithfulness, and unsupported claims (judge from a different model family than the generator).

### Knowledge and Agents Phase

- RAG over security documentation and procedures.
- Read-only investigation tools.
- LangChain, LangGraph, MCP, and LangSmith when justified by integration, orchestration, interoperability, or evaluation needs.

These technologies are not mandatory MVP requirements.

### Cloud Phase

- AWS managed services, selected according to application requirements.
- IAM and the principle of least privilege.
- Encryption, secrets management, logging, and monitoring.
- Infrastructure as code with Terraform.
- Reproducible deployment, tearing down infrastructure when not in use, and cost control.

## 8. Security, Privacy, and Reliability

Security must be a property of the system from the start.

- Use synthetic data during initial development.
- Never store credentials or secrets in the repository.
- Validate records before processing them.
- Limit access to original events.
- Avoid sending full logs to the LLM when not necessary.
- Maintain traceability between findings and original evidence.
- Apply least privilege to services and tools.
- Operate in read-only mode: never execute destructive actions or modify resources autonomously.
- Document limitations, assumptions, and known false positives.

## 9. Known Risks

| Risk | Mitigation |
|---|---|
| A synthetic dataset that is too "easy", producing misleadingly high metrics | Include ambiguous cases and realistic false positives (for example, an administrator mistyping a password) |
| Uncontrolled scope growth | Explicit definition of done for Phase 1 |
| Prompt injection through log fields | Treat logs as untrusted data; send aggregated findings to the LLM |
| An attractive interface that suggests nonexistent capabilities | Visual honesty rule |
| Unexpected AWS costs | Local development first; ephemeral infrastructure with Terraform |

## 10. Success Criteria

The project will not be considered successful just because it produces alerts or shows an attractive dashboard.

Its quality will be evaluated through:

- Correct normalization of the supported events.
- Reproducible deterministic rules.
- Automated tests covering positive and negative cases.
- Precision and recall measured on labeled datasets.
- Full traceability from alerts to their evidence.
- AI explanations faithful to the findings and free of unsupported claims.
- Security controls appropriate to the implemented capabilities.
- An interface consistent with the real data the system can provide.
- Clear documentation of architecture, technical decisions, and limitations.

## 11. Way of Working

Development will be incremental and driven by explicit technical decisions.

Before implementing a new stage:

1. Explain the problem to be solved.
2. Define the expected outcome.
3. Compare the relevant alternatives.
4. Choose the solution and justify it.
5. Record the decision in an ADR (`docs/adr/`).
6. Implement a small, understandable unit.
7. Test it and document its limitations.
8. Move on only when the current stage is sufficiently validated.

Documentation, code, and comments are written in English. Design discussions may happen in Spanish.

AI assistants must act as technical collaborators and mentors, explaining the purpose of generated code and the decisions behind it. They must not produce large amounts of code or add frameworks without a clear objective.

### Phase 1 Design Stages

| # | Stage | Status |
|---|---|---|
| 1 | System boundaries and high-level architecture | ADR-0001 proposed |
| 2 | Data model (`NormalizedEvent`) | Pending (an earlier draft exists, to be reviewed) |
| 3 | Synthetic dataset and ground truth | Pending |
| 4 | Rule engine | Pending |
| 5 | Findings and evidence | Pending |
| 6 | Evaluation | Pending |
| 7 | Where AI fits in | Pending |

## 12. Guiding Principle

**Build a reliable security engine first. Then turn it into an AI-assisted investigation platform.**

First, reliable and measurable detections. Then, grounded explanations. Finally, agents, cloud integrations, and a professional user experience.

CyberSec AI must grow from evidence and verifiable results, not from unnecessary technical complexity.
