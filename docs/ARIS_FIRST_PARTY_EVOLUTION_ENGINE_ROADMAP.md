# ARIS Harness: First-Party Candidate Evolution and Search Workplan

**Status:** Proposed, 2026-10-31; not a claim of deployed optimizer code  
**Authority:** [ARIS-owned Evolution and Training Blueprint](https://github.com/dylmarriner/ARIS/blob/main/docs/evolution/FIRST_PARTY_EVOLUTION_TRAINING_BLUEPRINT.md), [Owner TEST/DEPLOY policy](https://github.com/dylmarriner/ARIS/blob/main/docs/adr/0010-owner-governed-evolution.md), [First-party ownership ADR](https://github.com/dylmarriner/ARIS/blob/main/docs/adr/0012-first-party-evolution-and-training-ownership.md), [Project exclusions](https://github.com/dylmarriner/ARIS/blob/main/docs/adr/0011-exclude-independent-repositories.md).

## Design principle

Harness owns the **model-agnostic execution and evolutionary search process**, not model weights, the protected evaluator, SCOS evidence authority or owner-grant signing. Implement ARIS-owned code under the current `packages/aris/runtime` ownership *after EE-P0 confirms canonical active source*. DeepSeek Harness/Cordis-derived packages are real inherited code and require license/update inventories; do not claim the fork is independent because it uses an ARIS label.

## Research translation (not third-party dependencies)

| Inspiration | First-party Harness code | Avoid |
|---|---|---|
| GEPA | Trace-grounded reflection with a stored hypothesis; edit one prompt/skill parameter at a time, track Pareto front of quality/cost/regression | Treating a model-generated score as independently measured success |
| DSPy | Typed, composable model-stage contracts with fixed inputs/outputs and a measurable task harness | Replacing Executive with DSPy Python or requiring it at boot |
| Darwinian Evolver | Population archive, immutable parent digests, mutation operators, failure-case sampling, novelty and persistence across crashes | Copying its AGPL-3.0 implementation into proprietary ARIS without a specific license decision |
| Hermes Self-Evolution | Small candidate batches, unchanged champion, human-reviewable diffs and skill-specific regression fixtures | Its weak keyword-overlap fitness, unverified extraction and claimed-but-unwired test/PR gates |
| Atropos/Axolotl | Use Intelligence-produced rollout/reward/training manifests as evidence about model candidates | Having Harness run or host a full trainer or issue itself TEST/DEPLOY authority |

## First-party contracts

Use existing typed runtime and approval interfaces where possible. Proposed candidate structure (implement and validate only after EE-P0):

```typescript
interface EvolutionCandidateV1 {
  schemaVersion: "aris.candidate/v1";
  candidateId: string;
  parentArtifactSha256: string;
  proposedArtifactSha256: string;
  target: "prompt" | "skill" | "workflow" | "source" | "numeric" | "model-release";
  proposerRunId: string;
  traceEvidenceIds: string[];
  hypothesis: string;
  riskClass: string;
  testPlanDigest: string;
  rollbackArtifactSha256: string;
}
```

Candidate commit, source patch, model checkpoint, test plan and tool/environment revisions are content-addressed and immutable. No candidate controls its own evaluator, hidden test fixture or owner-grant verifier. `model-release` proposals reference Intelligence-produced checkpoint manifests; they do not train weights in Harness.

Proposed interfaces:
- `FailureMiner.analyse(trace_refs) -> EvidenceLinkedHypothesis[]`: separate real verification failure from subjective model preference; identify repeated failures and costs.
- `MutationStrategy.generate(parent, failure_cases, max_candidates) -> Candidate[]`: bounded textual edit, workflow graph edit, source patch or numeric change. Must preserve ancestry, diff and seed.
- `PopulationStore`: lineage by SHA/digest, Pareto archive by success, failure, risk, token cost, wall-clock, novelty and regression; retain negative experiments.
- `ProtectedEvaluatorPort.evaluate(candidate, approvedTestGrant)`: requests, never computes its own grading; independently authenticated scope and result.
- `OwnerDecisionPort`: multiple-choice proposals for TEST only; later separate DEPLOY choice after immutable result, exact digest/target, consent expiry and rollback path.
- `PromotionAdapter`: delegates verified installation to ARIS OS executor; never invokes arbitrary root commands as part of mutation.

## Search staging

1. **Stage H-OWN-0:** baseline sources and upstream-derived code/license inventory; locate existing loop, approval and trace hooks. Establish a real "no evolution" baseline.
2. **H-OWN-1:** extract failure family and source-linked examples from SCOS; freeze evaluator evidence. No mutation until verifier exists.
3. **H-OWN-2:** implement a deterministic one-field text mutator first, plus best-of-N comparison against human-designed baseline; reject reward hacking.
4. **H-OWN-3:** implement reflection from failed cases, constrained candidate mutation, Pareto/archive selection and diversity controls.
5. **H-OWN-4:** introduce independently written population search for code patches in temporary Git worktrees; run compile, unit, integration, compatibility, security and held-out tests.
6. **H-OWN-5:** only after measured reliability, allow search over mutation/search strategy itself under the same owner TEST/DEPLOY model. Internal evaluator/authority code can be *proposed* for modification but changes to those protected roots get independent review and cannot affect their own test outcome.
7. **H-OWN-6:** verify model-specific results for native ≤1B and optional larger providers separately. No universal "improvement" claim based on a single model.

## Acceptance / attack suite

- **Determinism:** replay same seed/trace/candidate yields same tested artifact identity; non-deterministic LLM proposals are recorded rather than claimed deterministic.
- **Resource:** budget cap on mutations, model/API calls and test host effects; cancelled/expired experiment stops without active release change.
- **Evaluation integrity:** a patch editing tests, labels, evaluator, fixtures or granted permissions fails; candidate has no direct access to held-out secrets.
- **Quality:** score held-out task outcomes, not simple keyphrase overlap, and track regression distribution, latency, memory use, context tokens and costs.
- **Consent:** TEST is scoped to candidate and experiment. DEPLOY is separately approved only after results are reviewed; a grant cannot be forged, replayed, broadened or silently inherited.
- **Operability:** interrupted search can restore archive/lineage, and release rollback preserves prior working champion. No third-party optimizer package needed on an ARIS OS install.
- **Architecture:** the three repositories excluded by ARIS ADR 0011 have zero ARIS-owned module references or dependencies.

## Roadmap bridge

Existing **H0–H3**: source inventory, durable runtime and evidence. **H4–H5**: typed model/solver stages, bounded proposals. **H6**: protected verifier and TEST/DEPLOY port. **H7**: SCOS evidence retrieval and reliable lineage. **H8**: actual first-party candidate engine, prompt/skill optimization then code search. **H9**: distributed node workers only through existing ARIS host authorization, never training service dependencies. Cross-repository EE-P0–P9 are not a replacement for these milestones. All status remains unverified until code and CI show otherwise.

## Source provenance

General concepts learned from [GEPA](https://github.com/gepa-ai/gepa), [DSPy](https://github.com/stanfordnlp/dspy), [Darwinian Evolver](https://github.com/imbue-ai/darwinian_evolver), [Hermes Self-Evolution](https://github.com/NousResearch/hermes-agent-self-evolution), [Atropos](https://github.com/NousResearch/atropos) and [Axolotl](https://github.com/axolotl-ai-cloud/axolotl). None of their implementation source is copied into this document or made a required runtime dependency.
