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
## Unit 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
All five sub-tasks from PLAN.md are done. I wrote `tests/benchmarks/test_ingestion_performance.py`, which ingests a resume plus 5 repos (metadata and README) through `IngestionPipeline` with the mock embedding provider and fails if the mean time exceeds 30s. I added `make test-benchmark` and a CI step that runs the benchmark. The mean is about 1.07s locally, and the test fails as expected when I set a tiny threshold.

**Next steps:**
Get a classmate or mentor to review the draft PR in Slack, wait for all five CI jobs to pass, record my own review in the PR's AI disclosure section, then mark the PR ready for review.

**Blockers:**
CI has not run on the PR yet, so the "all five jobs green" box is still unchecked.

---

### Check-in 2 (end of week)

**PR link:** [codepath/pathreview-ai201-fa26-howard#101](https://github.com/codepath/pathreview-ai201-fa26-howard/pull/101)

**Branch:** `test/46-ingestion-benchmark-pr` (the PR branch, containing only the code commits). My notes branch is `test/46-ingestion-pipeline-benchmark`, which also holds PLAN.md and this journal.

**What you built:**
A pytest-benchmark test that ingests a 5-repo + resume portfolio through the real ingestion pipeline (mock embeddings, in-memory fake vector store) and fails if the mean time is over 30 seconds. It also asserts that nothing was skipped and that chunks were stored, so it cannot pass by doing no work.

**Tests added or updated:**
`tests/benchmarks/test_ingestion_performance.py` (new). No existing tests changed, and there was no xfail marker to remove for this issue.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes

**Draft PR feedback received from:** none
