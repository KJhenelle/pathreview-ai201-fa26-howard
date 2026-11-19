## Unit 7 — Issue selection

**Issue link:** [[paste link here](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/46)]

**Issue title:** [Add a performance benchmark for the ingestion pipeline to detect regressions]

**Tier:** [ ] Tier 1  [ ] Tier 2  [x] Tier 3

**Problem summary:**
The repo doesnt currently have a test to verify if large portfolios complete injestion within 30 seconds. A benchmark test should be added to measure the mean injestion time and fail if it doesnt meet the 30 second threshold. The test should be located in tests/benchmarks/test_ingestion_performance.py

**Branch name:** [test/46-ingestion-pipeline-benchmark]

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Unit 8 — Reproduction & solution planning

**Reproduction commit link:** [da5079d](https://github.com/KJhenelle/pathreview-ai201-fa26-howard/commit/da5079dadf846b28b49c22f92efca6fbed5cc4f2)

**Reproduction summary:**
Issue #46 is a missing-test gap, so I reproduced it by confirming that `tests/benchmarks/` contains only `__init__.py` and that `tests/benchmarks/test_ingestion_performance.py` does not exist. Running `pytest tests/benchmarks -v` prints `collected 0 items` / `no tests ran in 56.66s`, so the 30-second ingestion requirement is not enforced anywhere. `pytest-benchmark` 5.3.1 is already installed and the `benchmark` marker is already registered.

**PLAN.md link:** [PLAN.md](https://github.com/KJhenelle/pathreview-ai201-fa26-howard/blob/test/46-ingestion-pipeline-benchmark/PLAN.md)

**Walkthrough video (recommended):** [not recorded]

**Blockers or open questions:**
- Even an empty `pytest` run takes ~57s here (startup/import cost), so I need to make sure the benchmark only times the ingestion calls.
- With the mock embedding provider, ingestion will probably finish far under 30s. I'm unsure whether that is the intended meaning of the threshold or whether the maintainer wants something stricter.
- I need to check whether any seeded bugs in the chunkers or parsers affect my synthetic 5-repo workload.