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

This fork carries the `pyo3` → `^0.29` bump and the cp314 wheel matrix, taken
from [explosion/spacy-alignments#16](https://github.com/explosion/spacy-alignments/pull/16)
by @dsbferris. That PR is now closed — superseded by
[#17](https://github.com/explosion/spacy-alignments/pull/17), not rejected — so
**#17 is the one to watch**; see "This fork should be temporary" below. The Rust
logic is untouched either way: the only `src/lib.rs` change is the version
string. Plus, in this repo:

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

Track **[explosion/spacy-alignments#17](https://github.com/explosion/spacy-alignments/pull/17)**
("Python 3.14 support, v0.9.3") — open, mergeable, and opened by a maintainer.

**Retire this fork when two things are true, not one.** A merge alone is not
enough: this repo exists to supply a *wheel*, so the condition is a release on
PyPI carrying cp314. Check both before spending any time maintaining what is
here:

```bash
gh pr view 17 -R explosion/spacy-alignments --json state
curl -s https://pypi.org/pypi/spacy-alignments/json | jq -r '.info.requires_python'
```

The second still reads `<3.14,>=3.9` as of 2026-09-29, which is why this fork
is still load-bearing.

### About the closed #16

Our verified build evidence is posted at
[#16](https://github.com/explosion/spacy-alignments/pull/16#issuecomment-5429597704),
and that PR is **closed**. It was superseded by #17, not refused — the author
closed it saying #17 "contains all of this and more". The link is kept so that
anyone who lands on a closed PR does not read it as either "upstream refused"
or "already merged, delete the fork".
