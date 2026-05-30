* [`cmake`](./cmake)

## Pin gcc when upstream is explicit

**Symptom**

Link or compile errors that contradict the visible source — typically `undefined reference to SSL_get_peer_certificate` from bundled C++ code that demonstrably exports the symbol, or compile failures inside `rust-rocksdb`'s `perf_flag.cc`. Identical pinning advice lives in upstream issue trackers (e.g. tikv/tikv#16593) and in Arch's `PKGBUILD` (`gcc12<12.4.0` + `export CC=gcc-12 CXX=g++-12`).

**Root cause**

gcc 13 tightened the default symbol-visibility rules and LTO heuristics. C++ codebases that worked under gcc 11/12 — especially those vendored as `rust-*` crates and not actively maintained against the newest toolchain — silently rely on the older behaviour. When pantry's `gnu.org/gcc` floats to a newer major, those builds break in non-obvious ways.

**Fix** — match what upstream did, pin to the gcc series upstream actually tested:

```yaml
build:
  dependencies:
    gnu.org/gcc: ^12.3 <12.4    # tikv#16593 — breaks on gcc 13+
  env:
    CC: '{{deps.gnu.org/gcc.prefix}}/bin/gcc'
    CXX: '{{deps.gnu.org/gcc.prefix}}/bin/g++'
```

Cross-link the upstream issue tracker in the recipe comment so future bumps can re-check whether the cap is still required.

Surfaced by #13089 (tikv).
