## Solution plan

**Issue:** [Add a performance benchmark for the ingestion pipeline to detect regressions (#46)](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/46)

### Understand

**Root cause:** This is a test-coverage gap, not a bug. Nothing in the repo measures how long ingestion takes. `tests/benchmarks/` contains only an empty `__init__.py`, and `test_ingestion_performance.py` (the file the issue names) does not exist. `pytest-benchmark` is already declared in the `dev` extras in `pyproject.toml`, and a `benchmark` marker is already registered under `[tool.pytest.ini_options]`, but no test uses either. A change that makes ingestion much slower (for example, an O(n²) chunker or embedding calls made one at a time) would pass CI unnoticed.

**Expected:** Ingesting a large portfolio (5 repos plus 1 resume) through `IngestionPipeline` completes within 30 seconds. A benchmark test measures this and fails if the mean time exceeds 30s.

**Actual:** No such test exists, so the 30s requirement is unenforced.

**Reproduction:** `ls tests/benchmarks/` shows only `__init__.py`. Running `pytest tests/benchmarks -v` (pytest 9.2.0, pytest-benchmark 5.3.1 loaded) prints `collected 0 items` and `no tests ran in 56.66s`. The 56s is almost entirely startup and import cost, which is why the benchmark must build the pipeline outside the timed function.

### Map

Files I expect to touch:
- `tests/benchmarks/test_ingestion_performance.py` (new): the benchmark itself.
- `tests/conftest.py` or a local fixture in the benchmark file: builds the 5-repo + resume workload.
- `Makefile`: add a `test-benchmark` target (currently only `test-unit`, `test-integration`, and `test-all` exist).
- `.github/workflows/ci.yml`: optional. The `test-unit` job only runs `tests/unit`, so benchmarks will not run in CI unless a step is added.

Code under test, not modified:
- `ingestion/pipeline.py`: `IngestionPipeline.ingest_resume`, `ingest_repo_metadata`, `ingest_readme`.
- `ingestion/embeddings/provider.py`: `MockEmbeddingProvider` (deterministic, 1536-dim, no network).
- `ingestion/embeddings/batch_processor.py`: `BatchEmbeddingProcessor` (calls `vector_db.add` once per chunk).
- `ingestion/chunking/strategy_selector.py`, `ingestion/parsers/*`.
- Reusable inputs: `tests/fixtures/sample_resumes/mid_career.md` and `tests/fixtures/sample_profiles/basic_profile.json` (only 2 repos, so the other 3 must be generated).

### Plan

1. **Build the workload.** Create a fixture that returns 5 repo metadata dicts (modeled on the shape in `basic_profile.json`, each with a realistic `readme_content`) plus the `mid_career.md` resume text.
2. **Build isolated dependencies.** Use `MockEmbeddingProvider`, a lightweight fake vector DB (an object with an `add(...)` method that records calls), and a mocked `db_session` whose `query().filter_by().first()` returns `None`, so nothing is skipped as "already ingested".
3. **Write the benchmark.** In `test_ingestion_performance.py`, mark it `@pytest.mark.benchmark`. Use the `benchmark` fixture to time one function that runs `ingest_resume` plus 5 × `ingest_repo_metadata` (and `ingest_readme` per repo). Use `benchmark.pedantic` with a fresh pipeline per round, so dedupe state and the fake DB do not carry over.
4. **Assert the threshold.** After the run, assert `benchmark.stats["mean"] < 30` (a named constant, `MAX_MEAN_SECONDS = 30`). Also assert the results were not skipped and `chunk_count > 0`, so the test cannot pass by doing no work.
5. **Wire it up.** Add a `make test-benchmark` target (`pytest tests/benchmarks -v -m benchmark`). Optionally add a CI step, and document the command in `docs/CONTRIBUTING.md`. Verify with `ruff check .` and `black --check .`, which CI enforces.

### Inputs & outputs

- **Input:** 5 repo metadata dicts, 1 resume markdown string, and a pipeline built with the mock embedding provider, a fake vector DB, and a mocked DB session.
- **Output:** A pytest-benchmark result (min/mean/max/stddev) printed on each run. The test passes if mean < 30s and fails with a clear message otherwise.
- **Changes:** One new test file, a Makefile target, and possibly one CI step. No production code changes.

### Risks & unknowns

- **Mock session silently skips everything.** `IngestionPipeline._check_skip` calls `db_session.query("IngestedSource").filter_by(...).first()`. A bare `Mock()` returns a truthy object, so every source would be reported as "already ingested" and the benchmark would time nothing. I must configure `.first.return_value = None` and assert `skipped is False`.
- **The mock-only benchmark may be too easy.** With `MockEmbeddingProvider`, ingestion probably finishes in well under a second, so a 30s threshold would only catch dramatic regressions. The real 30s risk is OpenAI network latency, which I cannot test deterministically. I should record the observed mean in the PR and consider whether to ask the maintainer about a tighter threshold or an optional "slow provider" variant.
- **Pre-existing bugs in the ingestion path.** The repo has intentionally seeded bugs and xfail tests (see `pyproject.toml` notes and `tests/unit/`). If a chunker or parser raises on my synthetic input, I need to find out whether it is a seeded bug (do not fix it here) or a problem with my fixture.
- **Slow environment.** Running `pytest` in the venv was very slow on this machine (likely heavy imports such as chromadb). I need to confirm import cost is not what dominates the measured time. The benchmark should construct the pipeline outside the timed function.
- **Flaky timing in CI.** Shared runners are noisy. Using the mean (as the issue asks) and the generous 30s limit should keep this stable.
- **Fake vs. real ChromaDB.** A fake vector DB skips real insert cost. I could use `chromadb.EphemeralClient` for realism, but it adds import time and possible flakiness. I need to decide after measuring both.

### Edge cases

- Mocked DB session returns a truthy value, so sources are skipped (guard against this with assertions).
- Repeated benchmark rounds re-ingest identical content. Use a fresh pipeline and fake DB per round, or the dedupe path will skew timings.
- A repo with an empty or missing `readme_content` (`has_readme` false), so no README ingestion for that repo.
- A very large README or resume that produces more than `BATCH_SIZE` (100) chunks and exercises multiple embedding batches.
- `pytest-benchmark` not installed (the `benchmark` fixture is missing). The error should be obvious, and the `dev` extras already include it.
- Running with `--benchmark-disable` or `-p no:benchmark`: the `benchmark.stats` access must not crash in a confusing way.
- Windows/macOS/Linux path handling when loading fixture files (use `pathlib` relative to the test file).
