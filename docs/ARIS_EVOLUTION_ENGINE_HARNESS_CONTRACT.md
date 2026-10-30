# ARIS Harness: Evolution Engine runtime integration

**Status:** Proposed ARIS-specific integration contract (2026-10-29), not an upstream DeepSeek Harness feature or completed implementation.
**Canonical specification:** [ARIS-EE-BP-001](https://github.com/dylmarriner/ARIS/blob/main/docs/evolution/ARIS_EVOLUTION_ENGINE_TECHNICAL_BLUEPRINT_AND_ROADMAP.md).
**Accepted governance:** [ARIS owner-governed evolution](https://github.com/dylmarriner/ARIS/blob/main/docs/architecture/owner-governed-evolution.md).

## Placement and boundaries

ARIS Harness owns the persistent Executive, model-agnostic adapter dispatch, tool/action orchestration, candidate improvement proposer, runtime loop control and presentation of reviewable proposals. It may evolve prompts, tool contracts, skills, agents, workflow algorithms and its own improvement method by proposing versioned changes. **Proposal is not authority.** ARIS OS validates effectful operations, SCOS persists traceable evidence, Intelligence provides model artifacts, and a protected evaluator decides experimental outcomes. Do not modify general upstream harness guarantees to claim ARIS-specific behavior that is not implemented.

## Evolution lifecycle and interfaces (proposed)

1. **Observe:** record stable task IDs, error classes, tool invocations/results, model, context and skill versions; link raw provenance in SCOS.
2. **Mine weaknesses:** cluster recurring failures and form falsifiable hypotheses tied to actual traces, not self-graded model confidence.
3. **Propose:** generate a minimal patch/diff and immutable candidate manifest with ancestor digest, target surface, expected effect, risk, tests, budget and rollback reference. Preserve rejected candidates.
4. **Review TEST:** present a user-readable multiple-choice decision. Approval is scoped to exact candidate digest, test environment, duration and resources. Harness cannot fabricate owner consent.
5. **Evaluate:** invoke the independent sandbox/evaluator only with valid TEST authorization. Support fixed model and fixture versions, held-out tasks, taint/permission/loop tests and comparable baseline metrics.
6. **Report:** expose wins, losses, costs, confidence and artifacts. No candidate changes its own test score, authority verifier or production environment.
7. **Review DEPLOY:** issue a **separate** question for an exact tested candidate, target and rollback. Owner test approval never implies production installation.
8. **Deploy/recover:** ARIS OS executor independently revalidates DEPLOY and returns observed postcondition evidence; Harness records outcome and can request rollback.

## Optimization architecture

- Start with targeted **Self-Harness** failure-driven edits and **GEPA**-like prompt/skill reflection.
- Use **Meta-Harness**-style selective source/trace exploration and **AlphaEvolve**-style code archives after first reliable evaluator is available.
- Use **AFlow/ADAS** workflow search only when measurable held-out improvements justify the cost.
- Use **CMA-ES/NES** for numerical controls; do not force numerical search onto arbitrary source-code edits.
- Retain model-specific baselines and measure success, context cost, tool retries, crashes, and regression distributions.

## Implementation mapping

| Phase | Harness integration deliverable | Required verification |
|---|---|---|
| EE-P0 | Audit active Harness tree vs legacy/upstream variants | Actual source commit, package paths, test status |
| EE-P1 | Correlated task/candidate telemetry and SCOS provenance | Durable idempotent trace replay |
| EE-P2 | Independent evaluation invocation and artifact checks | Tamper, leakage, hidden-test and score-integrity tests |
| EE-P3 | Two-stage, digest-bound owner review transport | Revocation/expiry/replay/identity mismatch tests |
| EE-P4 | First bounded patch proposal and isolated comparison | Two candidates, honest baseline, held-out regression |
| EE-P5–P6 | Context/meta-skill and Pareto search hooks | Matched-budget evaluation; candidate lineage |
| EE-P7–P9 | Model-specific workflow evolution and release hardening | Compat, crash recovery, no unapproved deployment |

## Compatibility

The ARIS-specific contract should be implemented in owned packages/plugins and verified in EE-P0; it must not claim existing upstream facilities meet unimplemented privacy, authorization or deployment guarantees. Existing [ARIS Harness technical blueprint](ARIS_HARNESS_TECHNICAL_BLUEPRINT_AND_ROADMAP.md) and canonical system map continue to govern placement. This change is documentation-only.

## First-party realization and trainer separation (2026-10-31)

See [ARIS Harness owned optimizer design](ARIS_FIRST_PARTY_EVOLUTION_ENGINE_ROADMAP.md) and [canonical training/evolution blueprint](https://github.com/dylmarriner/ARIS/blob/main/docs/evolution/FIRST_PARTY_EVOLUTION_TRAINING_BLUEPRINT.md). Implement search strategies as native Harness components while SCOS supplies evidence, Intelligence owns offline model training and ARIS OS maintains the independent evaluator/owner grant verifier. Source code from AGPL-licensed Darwinian Evolver is not to be copied without separate legal authorization; other source transplants likewise require recorded license/provenance/security maintenance. No third-party research framework becomes an OS runtime requirement.
