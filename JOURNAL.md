## Unit 7 — Issue selection

**Issue link:** [[paste link here](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/46)]

**Issue title:** [Add a performance benchmark for the ingestion pipeline to detect regressions]

**Tier:** [ ] Tier 1  [ ] Tier 2  [x] Tier 3

**Problem summary:**
The repo doesnt currently have a test to verify if large portfolios complete injestion within 30 seconds. A benchmark test should be added to measure the mean injestion time and fail if it doesnt meet the 30 second threshold. The test should be located in tests/benchmarks/test_ingestion_performance.py

**Branch name:** [test/46-ingestion-pipeline-benchmark]

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger