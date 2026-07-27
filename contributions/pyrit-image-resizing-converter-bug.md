# PyRIT: ImageResizingConverter Zero-Dimension Bug Report

**Issue**: [microsoft/PyRIT #2162](https://github.com/microsoft/PyRIT/issues/2162)
**Status**: Closed; fixed in [#2169](https://github.com/microsoft/PyRIT/pull/2169) based on my finding

## The bug

PyRIT is Microsoft's AI red-teaming toolkit; converters transform attack prompts and payloads (here, adversarial images) before they're sent to a target. `ImageResizingConverter` downscales images by a `scale_factor` but only validated `scale_factor > 0`, leaving two holes:

- A positive fractional factor on a tiny image truncates a dimension to zero: `ImageResizingConverter(scale_factor=0.5)` on a 1×1 image computes `int(1 * 0.5) = 0`, and Pillow raises `ValueError: height and width must be > 0` mid-pipeline.
- Non-finite values (`nan`, `inf`) passed validation entirely.

For automated red-team runs the failure mode is an opaque Pillow crash in the middle of a batch image-attack workflow instead of a clear validation error at construction time.

## Report and resolution

The report included the exact repro, both holes, and a fix design: reject non-finite scale factors at construction and guard computed dimensions before calling Pillow. The merged fix ([#2169](https://github.com/microsoft/PyRIT/pull/2169)) implements the finite check as proposed and clamps output dimensions to a 1px minimum, with tests for `nan`/`±inf` and 1×1, 1×8, 8×1 images.

## Links

- Issue #2162: https://github.com/microsoft/PyRIT/issues/2162
- Fix PR #2169: https://github.com/microsoft/PyRIT/pull/2169
