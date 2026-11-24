# StackGen | AI Agent Evaluation Take-Home

A compact, reproducible **hybrid trace evaluator** for the eight supplied workplace-agent scenarios. Standard-library Python; no credentials, services, or dependencies required.

## Run

```bash
python3 checker.py traces.json --output results.json
python3 -m unittest discover -s tests -v
```

`checker.py` prints JSON with the required `summary.total_traces`, `summary.pass_rate`, `summary.total_release_blockers`, and `traces[]` (`trace_id`, `evaluator_type`, `result`, `reasons`). Additional `findings[]` and `release_blocker` fields expose the audit trail. Nonzero exit on unreadable input/invalid trace schema. Python 3.10+.

## What's inside

- `traces.json` — structured transcription of the eight traces from the exercise.
- `checker.py` — schema checks, deterministic gates, decision policy, aggregation, CLI.
- `judge.py` — **actual** system prompt, user prompt, JSON output schema, API-ready message builder, plus an offline `MockJudge`.
- `rubric.md` — one-page score/severity/ship policy.
- `scores.csv` — each trace's assessment with 2–3 specific facts.
- `improvements.md` — two prioritized changes and checker trade-offs.
- `tests/test_checker.py` — baseline, counterfactual, and fail-closed tests.

## Why this design?

**Deterministic first, semantic second.** Rule checks are reliable for specific IDs, tool failures, confidentiality markers, and side effects. The judge contract evaluates meaning, newer decisions, causal support, and uncertainty; explicit severities prevent a high average from hiding blockers. The output remains explainable and easy to inspect in a live review. `evaluator_type` reports the primary detection path, not exclusivity; both stages run.

## Important limitations (explicit, not hidden)

1. `MockJudge` uses **narrow heuristic stand-ins**, not an LLM. It demonstrates the integration and output contract but cannot interpret arbitrary paraphrases, root causes, or timestamps. The production implementation must call a trusted model using `build_judge_messages(trace)` and enforce `RESPONSE_SCHEMA`, plus calibrate judgments against human labels. In the mock, dates without a year share an arbitrary comparison year; production uses timezone-aware timestamps.
2. Regex-based ID provenance does not establish that a referenced issue has the claimed *status*. Likewise, exact/near-quote privacy checks do not catch all semantic leaks. The real judge and stronger typed evidence/provenance checks are the intended complements.
3. `judge_unavailable` returns **fail** rather than risking an accidental pass because the assignment demands pass/fail. In a production evaluation service this should also be reported as `INCONCLUSIVE` so a judge outage is not confused with a model regression.
4. The rubric is intentionally conservative around externally visible sends and operational rollback advice. An organization's authorization/incident policies should determine final thresholds.

## Discussion examples

- **Timeout vs empty:** A successful empty Linear response can support a scoped no-bugs claim; a timeout cannot.
- **Draft vs send:** Sending after a draft request is a blocker, even with a valid channel.
- **Newer Slack update:** A later decision can change a previously failing answer to pass.
- **Release notes vs launch:** ENG-52 blocks September *release notes*, not proven to block the software launch. The omission is material, but keep the scope precise.
- **Metrics vs cause:** Payment timeouts and co-occurring p99 spikes do not establish a removed cache or justify rollback.
