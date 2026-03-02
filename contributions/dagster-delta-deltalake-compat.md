# Dagster-Delta: deltalake >=1.2.1 Compatibility

**PR**: [ASML-Labs/dagster-delta #54](https://github.com/ASML-Labs/dagster-delta/pull/54)
**Status**: Merged
**Reviewer**: ion-elgreco

## The bug

dagster-delta was pinned to `deltalake<=1.2.0` ([#48](https://github.com/ASML-Labs/dagster-delta/issues/48)) because newer versions broke the test suite. The release workflow was also broken — artifacts landed in the wrong directory so publishing silently failed.

## The fix

deltalake 1.2.1+ changed Arrow table behavior in a few ways:

1. **Types**: columns come back as `string_view` instead of `string`, so direct table equality checks fail even though the data is the same
2. **Row ordering**: results aren't deterministically ordered anymore
3. **Partition columns**: can show up at different positions in the schema

Fixed across 4 test files with different strategies depending on the assertion pattern:
- Round-tripped tables through `pa.table(t.to_pydict())` to normalize away type differences
- Wrapped `to_pylist()` checks in `sorted()` for order-independent comparison
- Used `.select(cols)` to align column ordering

Bumped the dep constraint from `deltalake<=1.2.0` to `deltalake>=1.2.1` in both `pyproject.toml` files (lockfile resolved to 1.4.2).

Also fixed the release workflow — `uv build` defaults to repo-root `dist/` but the validation script and `uv publish` expect `${{ inputs.working_directory }}/dist`. Added `--out-dir dist` so they all agree.

## Links

- PR: https://github.com/ASML-Labs/dagster-delta/pull/54
- Issue: https://github.com/ASML-Labs/dagster-delta/issues/48
