# Dagster-Delta: deltalake Compatibility and Release Output

**PR**: [ASML-Labs/dagster-delta #54](https://github.com/ASML-Labs/dagster-delta/pull/54)
**Status**: Merged

## Change

- Updated test expectations for newer `deltalake` / Arrow behavior.
- Normalized table comparisons where `string_view` and `string` differences made equality brittle.
- Sorted outputs where row ordering was no longer deterministic.
- Aligned partition columns before comparisons where newer behavior returned a different column order.
- Bumped deltalake constraints in both `pyproject.toml` files.
- Fixed release artifact output by changing the release build to `uv build --out-dir dist`.

## What it enables

- Maintainers can upgrade deltalake without the test suite failing on storage-layer representation changes that do not change the actual data.
- Release publishing can find the artifacts it expects because the build now writes to `dist`.
- The test suite becomes a better signal: it checks data correctness while tolerating row ordering, schema representation, and partition-column differences introduced by newer deltalake behavior.

## Code notes

The heaviest compatibility path had to handle three independent changes at once:

- **Arrow representation**: rebuild tables through `pa.table(table.to_pydict())` before equality checks.
- **Row order**: sort both expected and actual values when storage order is not part of the contract.
- **Partition columns**: select columns in the expected order before asserting equality.

The release workflow fix is intentionally direct:

```yaml
run: uv build --out-dir dist
```

## Links

- PR: https://github.com/ASML-Labs/dagster-delta/pull/54
- Issue: https://github.com/ASML-Labs/dagster-delta/issues/48
