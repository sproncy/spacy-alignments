# Why this fork exists

`spacy-alignments` is the only package in the spaCy transformer stack without a
CPython 3.14 wheel, and it is a hard blocker rather than an inconvenience:

- There is no cp314 wheel on PyPI, so installers fall back to the sdist.
- The sdist does not build on 3.14 either. Upstream pins `pyo3 ^0.24`, and
  `pyo3-ffi 0.24.2` refuses to compile against 3.14:

  ```
  error: failed to run custom build command for `pyo3-ffi v0.24.2`
  error: the configured Python interpreter version (3.14) is newer than
         [pyo3's maximum supported version]
  ```

So on 3.14 there is no working path at all, not even "install a Rust toolchain".
That holds back `sproncy-ml`, which needs `spacy-transformers` for the
`TransformerModel.v3` and `TransformerListener.v1` architectures in its NER
pipeline, and therefore holds back its `sproncy-schemas` pin — see
[`EXTRACTION_ROADMAP.md` §Version pins](https://github.com/sproncy/sproncy-schemas/blob/main/EXTRACTION_ROADMAP.md#version-pins).

## What is different from upstream

This fork carries [explosion/spacy-alignments#16](https://github.com/explosion/spacy-alignments/pull/16)
by @dsbferris, which bumps `pyo3` to `^0.29` and enables cp314 in the wheel
matrix. The Rust logic is untouched — the only `src/lib.rs` change is the
version string. Plus, in this repo:

- `publish_pypi.yml` is **removed**. This fork must never publish to the
  upstream `spacy-alignments` PyPI project.
- The wheel matrix is trimmed to `ubuntu-latest` and `macos-14`, which is what
  the Sproncy fleet consumes: self-hosted CI is linux/x64, developer machines
  are Apple silicon.
- `CIBW_BUILD` is limited to `cp314-*`. Upstream already ships wheels for every
  earlier version, and duplicating them here risks shadowing upstream's.
- Releases are published rather than drafted, because a draft release is not
  downloadable by CI.

## Consuming the wheels

Releases are cut by tagging `release-vX.Y.Z`, which builds the wheels and
attaches them to a GitHub release. Point `uv` at that release:

```toml
[tool.uv.sources]
spacy-alignments = { url = "https://github.com/sproncy/spacy-alignments/releases/download/0.9.3/spacy_alignments-0.9.3-cp314-cp314-manylinux_2_28_x86_64.whl" }
```

A per-platform URL is awkward across linux and macOS; prefer `--find-links`
against the release page, or a small private index, if more than one platform
needs serving from the same lockfile.

## This fork should be temporary

The upstream PR is small, correct and mergeable. Verified build evidence is
posted at
[explosion/spacy-alignments#16](https://github.com/explosion/spacy-alignments/pull/16#issuecomment-5429597704).
If upstream merges and releases, retire this fork and go back to PyPI — check
that first before spending any time maintaining what is here.
