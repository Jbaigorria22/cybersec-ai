# ADR-0001: System Boundaries and High-Level Architecture (Phase 1)

- **Status:** Proposed
- **Date:** 2026-10-29
- **Design stage:** 1 of 7

## Context

CyberSec AI aims to help analyze security events in AWS. Before adding AI, we need
a detector whose results can be **tested and measured**. Without that baseline, there
is no way to tell whether an AI component adds value or only complexity.

The initial data source is **AWS CloudTrail**, because it records AWS control-plane
activity (who did what, from where, and with what result) and aligns with a
Cloud + Security profile.

## Decisions

### D1. Batch processing (files), not real time
The system processes JSON files in the format CloudTrail delivers to S3.
- **Why:** the same input always produces the same output, so tests are repeatable.
- **Rejected alternative:** consuming live events through EventBridge. It adds infrastructure and makes testing harder without adding learning value in this phase.

### D2. Decoupled pipeline stages with contracts
`Ingestion → Normalization → Detection → Findings → Evaluation`.
Each stage receives and returns data with a defined shape and does exactly one thing.
- **Why:** each stage can be tested in isolation and replaced without affecting the others.
- **Rejected alternative:** a monolithic script. Faster to write, but impossible to test in parts and hard to extend.

### D3. Read-only system
The system observes and reports. It does not block users, modify resources, or connect to real accounts in this phase.
- **Why:** a detector that can take action is a valuable target for an attacker. This also satisfies the requirement that consequential actions need human approval.
- **Rejected alternative:** automated response (for example, disabling credentials). Deferred to a future phase, always with human approval.

### D4. Deterministic detection
Rules use no randomness, probabilistic heuristics, or AI models.
- **Why:** metrics (precision, recall) are only meaningful if results are reproducible. This establishes the **baseline** against which any AI component will be compared.
- **Rejected alternative:** LLM-assisted detection. It is neither reproducible nor measurable as a starting point.

## Out of scope (Phase 1)

Connections to real AWS accounts, real-time processing, graphical interface,
AI components, multi-tenant support.

## Consequences

- **Positive:** a testable, measurable system; AI can later be added as one more stage after "Findings" without modifying detection.
- **Negative / accepted:** no real-time detection; reliance on synthetic data until later phases, so metrics reflect the dataset, not a real environment.
- **Risk to watch:** a synthetic dataset that is too "easy" could produce a false sense of quality. Addressed in Stage 3.

## How it connects

- **Stage 2 (Data model):** defines the contract between Normalization and Detection.
- **Stage 3 (Dataset):** feeds Ingestion and Evaluation.
- **Stage 7 (AI):** sits after Findings and never decides whether a detection occurred.
