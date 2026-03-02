# Dagster-Delta: deltalake >=1.2.1 Compatibility

**PR**: [ASML-Labs/dagster-delta #54](https://github.com/ASML-Labs/dagster-delta/pull/54)
**Status**: Merged
**Reviewer**: ion-elgreco

## The bug

dagster-delta was pinned to `deltalake<=1.2.0` ([#48](https://github.com/ASML-Labs/dagster-delta/issues/48)) because newer versions broke the test suite. The release workflow was also broken: `uv build` writes artifacts to repo-root `dist/` but validation and publishing expected them in the workspace subdirectory, so publishing silently failed.

## The fix

deltalake 1.2.1+ changed how Arrow tables come back in three ways, each needing a different fix:

**`string_view` vs `string` types** - columns now come back as `string_view`, so direct PyArrow table equality fails even though the data is identical. Fixed by round-tripping tables through `pa.table(table.to_pydict())` to normalize the types.

**Nondeterministic row ordering** - result sets aren't ordered the same way anymore. Wrapped list assertions in `sorted()`.

**Partition column positioning** - partition columns can show up at different positions in the schema. Used `.select(cols)` to align column order before comparison.

The heaviest test file needed all three strategies plus explicit `.sort_by()` on both sides of comparisons. The others mostly just needed `sorted()` wrappers.

Bumped the dep from `<=1.2.0` to `>=1.2.1` in both `pyproject.toml` files (lockfile resolved to 1.4.2). Fixed the release workflow by adding `--out-dir dist` to `uv build`.

## Links

- PR: https://github.com/ASML-Labs/dagster-delta/pull/54
- Issue: https://github.com/ASML-Labs/dagster-delta/issues/48
