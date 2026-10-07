# Coverage measurement and gate policy

## Measurement scope

Measure every Python production module under `src/longparser`. There are no file omissions for server, pipeline, extractors, OCR helpers or integration adapters. `source` already restricts measurement to production code, so an additional `tests/*` omission is unnecessary.

Optional dependencies and model requirements do not justify hiding whole modules: missing runtime paths should remain visible, and helper behavior can be exercised with boundary mocks. Coverage's existing default exclusions remain for abstract declarations and `TYPE_CHECKING` imports; the Milestone 2 report contains 25 such excluded lines.

Coverage is statement coverage, not a measure of document accuracy, model quality or end-to-end service availability. Executing an import does not establish that a module's runtime behavior has been tested.

## Milestone 2 baseline

Measured on Python 3.13.15 with the existing development/server dependency environment, Ruff 0.15.4, mypy 1.20.0 and coverage.py 7.13.5. The same 74 tests pass under both policies, with zero failures or skips and two existing Pydantic deprecation warnings.

| Policy | Files | Measured statements | Covered statements | Coverage | Minimum |
|---|---:|---:|---:|---:|---:|
| Previous omissions | 14 | 1,207 | 305 | 25.27% | 5% |
| Full production scope | 40 | 4,873 | 924 | 18.96% | **17% provisional** |

All 14 previously measured files have exactly the same measured and covered statement counts under the wider policy. The wider report adds 26 files, 3,666 measured statements and 619 covered statements that the old policy hid. No tests were added in Milestone 2: the newly reported execution comes from the existing suite.

## Coverage by module group

Percentages below are evidence, not additional per-module pass/fail gates.

| Module/group | Measured statements | Covered statements | Coverage | File omitted? | Important remaining paths |
|---|---:|---:|---:|---|---|
| Core schemas and package initializers | 225 | 222 | 98.67% | No | Remaining lazy-import and unknown-export paths. |
| Hybrid chunker | 479 | 42 | 8.77% | No | Packing, hierarchy, overlap, structured tables, lists and equation cases (Milestone 3). |
| Quality scorer and semantic boundaries | 92 | 0 | 0.00% | No | Fallbacks, scoring and semantic split decisions (Milestone 3). |
| Extractors and LaTeX helpers | 1,620 | 94 | 5.80% | No | Mocked format conversion, heading/table helpers and optional-backend guards (Milestone 5); real OCR/model paths remain separate. |
| Integration adapters | 90 | 0 | 0.00% | No | Adapter conversion and optional-library boundaries; currently unexecuted. |
| Pipeline, PII, references and summaries | 481 | 35 | 7.28% | No | Backend selection, redaction, references and summaries with boundary mocks (Milestone 4). |
| Server and chat | 1,760 | 504 | 28.64% | No | Routes, persistence, queues, workers and vector-store paths; existing execution includes imports and chat utility tests. |
| Utilities | 126 | 27 | 21.43% | No | Language detection, OCR routing and RTL edge cases. |
| **Total** | **4,873** | **924** | **18.96%** | **No** | Planned tests and remaining service/model paths. |

## Provisional and final minimum

`fail_under = 17` is an intermediate floor derived from the measured 18.96% baseline. It leaves 1.96 percentage points of headroom for small interpreter/environment differences while providing a stronger check than the previous 5% gate. Local validation covers Python 3.13; the unchanged CI matrix runs Python 3.10–3.13.

Before submitting the combined implementation PR:

1. Complete the planned tests in Milestones 3–5 on this same branch.
2. Re-measure coverage over the same full production scope and update module/group evidence.
3. Raise the committed minimum from these final results, using a documented small margin, and validate it against the available CI matrix results.
4. Run the complete suite with the final committed gate and no diagnostic overrides.
5. If the completed work cannot support a meaningful final gate, resolve DEC-06 from the implementation plan before submission. Do not disable coverage or restore broad omissions to produce a passing result.

The provisional 17% floor is not the completed release gate. A future source-scope change must be explained separately from added executed statements.

## Informational typing check

CI runs `mypy src/longparser/` after installing `.[dev,server]`. Its named step has `continue-on-error: true`; Ruff and pytest/coverage remain blocking checks. Preserve the existing mypy configuration rather than starting a strict-typing cleanup in this release.

The initial baseline is **94 errors in 19 files**, checked across 41 source files before deleting the empty routers initializer. Deleting that initializer removes one empty file, not typing errors. These are existing findings to make visible, not new failures caused by this milestone.

## Reproducing the measurements

From the repository root, with the development and server dependencies installed:

```bash
.venv/bin/python -m pytest tests/ --cov=longparser --cov-report=term-missing --cov-report=json:coverage.json
.venv/bin/ruff check .
.venv/bin/mypy src/longparser/
```

The mypy command exits nonzero while its baseline findings remain; this is expected for the informational step. Review JSON coverage totals and file sets when comparing measurements, not only the headline percentage.
