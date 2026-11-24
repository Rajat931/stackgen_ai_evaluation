# Improvements before the next release

## 1. Enforce policy at the agent's tool boundary (architecture + guardrails)

Introduce typed tool outcomes (`success`, `empty`, `timeout`, `forbidden`), resource-level provenance, and a centralized approval gate for state-changing operations. Require an explicit destination or confirmed approval before `slack_send`; keep `draft` and `send` as different capabilities. Filter confidential fields **before** answer generation, and record policy decisions in an audit trail. In incident workflows, separate *observed telemetry*, *hypotheses*, and *verified root causes*; require deployment evidence or human authorization for rollback actions.

**Why:** Evaluation detects defects after they happen; runtime gates reduce the chance of harmful actions or disclosures in the first place.

## 2. Calibrate the evaluator on counterfactual and human-labeled traces (test set + evaluator)

Add a versioned gold dataset with deliberate perturbations: ticket resolved; newer Slack reversal; identical names; timeout vs empty result; confidential message paraphrased; `draft` changed to `send`; PR approved-but-unmerged; correlated metrics without deployment evidence. Have reviewers label **finding, severity, and supporting evidence** independently, adjudicate disagreements, and track false-negative rates on safety blockers alongside false-positive rates and judge drift. Use structured output validation, sampled human audits, and regression tests on every prompt/rule change.

**Why:** Eight curated traces cannot establish real-world reliability; a judge must itself be evaluated.

## Hybrid trade-offs: cost, latency, accuracy

- **Rules:** low cost/latency and reproducible for explicit states; brittle against paraphrases and unmodeled tool schemas.
- **LLM judge:** flexible for temporal, causal, and language judgments; higher cost/latency, nondeterminism, and possible false confidence.
- **Combined:** run hard gates first, then invoke an approved LLM judge on context-heavy cases; cache by trace/prompt version, sample for audits, and fail closed on judge errors. *This submission's `MockJudge` is an offline test double, not a production semantic evaluator.* Confidential raw tool data must stay within an authorized processing boundary.
