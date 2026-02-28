# Dagster-Delta: deltalake >=1.2.1 Compatibility

**PR**: [ASML-Labs/dagster-delta #54](https://github.com/ASML-Labs/dagster-delta/pull/54)
**Status**: Merged
**Reviewer**: ion-elgreco (approved)

## What broke

The dagster-delta project had been stuck on `deltalake<=1.2.0` ([#48](https://github.com/ASML-Labs/dagster-delta/issues/48)) because newer deltalake versions broke the test suite in multiple ways. The release workflow was also broken — builds would succeed but artifacts would land in the wrong directory, so validation and publishing silently failed.

## Root cause

deltalake 1.2.1+ changed how Arrow tables are returned in three distinct ways that I had to identify and handle separately:

1. **Type representation**: columns now come back as `string_view` instead of `string` — so any direct PyArrow table equality assertion fails even though the data is identical
2. **Row ordering**: result sets are no longer deterministic in order, breaking assertions that assumed stable ordering
3. **Partition column positioning**: partition columns can appear at different positions in the schema, so column-order-dependent comparisons break

Each of these needed a different fix strategy across the test files.

## What I did

**Test normalization across 4 test files** — each with a different assertion pattern:
- Round-tripped PyArrow tables through `pa.table(t.to_pydict())` to normalize away `string_view` vs `string` type differences and internal chunk layout changes
- Wrapped `to_pylist()` assertions in `sorted()` for order-independent value comparison
- Used `.select(cols)` to align column ordering where partition columns shifted position

**Dependency constraint update** across both `pyproject.toml` files — flipped from `deltalake<=1.2.0` to `deltalake>=1.2.1` (lockfile resolved to 1.4.2, confirming compatibility across the full range).

**Release workflow fix** in `.github/workflows/template-release.yml` — `uv build` in a workspace defaults to repo-root `dist/`, but `validate-release-version.py` and `uv publish` expect artifacts in `${{ inputs.working_directory }}/dist`. Added `--out-dir dist` so builds, validation, and publishing all agree on the artifact location.

Tested locally before submitting.

## Why it matters

dagster-delta is the Delta Lake integration for Dagster, maintained by ASML Labs. Pinning to an old deltalake version meant users couldn't get upstream performance improvements and bug fixes. The broken release workflow meant even if someone fixed the tests, new versions couldn't actually be published. This PR unblocked both the upgrade path and the release pipeline.

## Links

- PR: https://github.com/ASML-Labs/dagster-delta/pull/54
- Issue: https://github.com/ASML-Labs/dagster-delta/issues/48
