# ADR-0001: System Boundaries and High-Level Architecture

- **Status:** Proposed (revised)
- **Date:** 2026-10-29
- **Applies to:** Stages A–D (see brief, section 3)

## Context

CyberSec AI is a platform where AI agents help investigate security events in AWS.
Before adding AI, we need a detector whose results can be **tested and measured**.
Without that baseline, there is no way to tell whether an AI component adds value
or only complexity.

The initial data source is **AWS CloudTrail**, because it records AWS control-plane
activity (who did what, from where, and with what result).

**Revision note:** the first version of this ADR placed all AI work at the end of the
project. The roadmap was reordered so the agent arrives in Stage B. The four original
decisions remain; D5 was added to define where the agent sits.

## Decisions

### D1. Batch processing (files), not real time
The system processes JSON files in the format CloudTrail delivers to S3.
- **Why:** the same input always produces the same output, so tests are repeatable.
- **Implication:** the unit of work is the **whole set of input files**, loaded and sorted by event time before rules run. Otherwise a pattern split across two files (for example, a brute-force attempt) would be missed.
- **Rejected alternative:** live events through EventBridge. More infrastructure, harder testing, no learning value at this stage.

### D2. Decoupled pipeline stages with contracts
`Ingestion → Normalization → Detection → Findings → Evaluation`, with the agent consuming Findings (D5).
Each stage receives and returns data with a defined shape and does exactly one thing.
- **Why:** each stage can be tested in isolation and replaced without affecting the others. Rules depend on `NormalizedEvent`, never on raw CloudTrail JSON, so adding a new log source only changes ingestion and normalization.
- **Rejected alternative:** a monolithic script. Faster to write, impossible to test in parts.

### D3. Read-only system
The system observes and reports. It does not block users, modify resources, or act on accounts.
- **Why:** a detector that can take action is a valuable target for an attacker. Read-only limits the damage if it is compromised or fooled.
- **Not a guarantee of safety:** attacker-controlled fields (for example, `userAgent`) and malformed input remain attack vectors. Input is validated, and log content is always treated as data, never as instructions.
- **Rejected alternative:** automated response (for example, disabling credentials). Deferred, and always with human approval.

### D4. Deterministic detection
Rules use no randomness, probabilistic heuristics, or AI models.
- **Why:** precision and recall are only meaningful if results are reproducible. This is the baseline any AI component is compared against.
- **Implication:** events are always processed in a stable order (event time, then event ID), and rules use event time, never the current wall-clock time.
- **Rejected alternative:** LLM-assisted detection. Neither reproducible nor measurable.

### D5. The agent investigates; it never detects
The investigator agent (Stage B) consumes findings and uses read-only tools to explain them and recommend next steps.
- **Why:** detection stays measurable (D4), and the agent's output can be checked against the evidence in each finding.
- **Implication:** the agent receives findings and selected event fields, not raw logs. Its outputs are structured and validated before use.
- **Rejected alternative:** an agent that reads raw logs and decides what is a threat. Unmeasurable, prone to invented evidence, and exposed to prompt injection through log fields.

## Out of scope (until a later ADR changes it)

Connections to real AWS accounts, real-time processing, automated response,
multi-tenant support. The web interface is deferred to Stage E.

## Consequences

- **Positive:** a testable, measurable core; the agent is one more stage after Findings, so it can be added or changed without touching detection.
- **Negative / accepted:** no real-time detection; synthetic data until later stages, so metrics reflect the dataset, not a real environment.
- **Risk to watch:** a synthetic dataset that is too "easy" gives a false sense of quality. Addressed when designing the dataset in Stage A.

## How it connects

- **Data model (`NormalizedEvent`):** the contract between Normalization and Detection (D2).
- **Dataset and labels:** feed Ingestion and Evaluation.
- **Investigator agent (Stage B):** sits after Findings (D5).
- **MCP (Stage C) and AWS (Stage D):** change how tools are exposed and where the system runs, not these boundaries.
