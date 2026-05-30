# c/c++

c and c++ are complex. Try ensuring the `cc` (c-compiler) that is used is `clang`. Dong this varies depending on the build tooling. Usually `CC=clang` and `CXX=clang++` is enough. Cmake is weird tho.

## Old autotools projects fail on aarch64

**Symptom**

```
configure: error: cannot guess build type; you must specify one
```

…seen on `linux/aarch64`. The tarball's `autotools/config.guess` predates the aarch64 triplet (often stamped 2009 or earlier).

**Root cause**

Vintage autotools projects ship their own `config.guess` / `config.sub` next to `configure`. Those copies were written before `aarch64-unknown-linux-gnu` existed, so the build-type probe falls through. Debian works around this with `dh-autoreconf`; other distros run `autoreconf -fi` to drop fresh GNU scripts in place.

**Fix**

Vendor fresh FSF copies next to the recipe and drop them in before `./configure`:

```yaml
build:
  - cp props/config.guess props/config.sub autotools/
  - ./configure $ARGS
```

Running `autoreconf -fi` achieves the same thing if the project's `configure.ac` is sane and the matching autotools are available at build time.

Surfaced by #13115 (zsync).

## Tests hang when the binary links against ImageMagick

**Symptom**

A binary that links against `libMagickCore` hangs for ~30 s on startup in CI — even for a `--version` invocation — and the test step times out.

**Root cause**

`MagickCoreGenesis()` runs unconditionally at process start, walks the per-user `~/.config/ImageMagick/` tree and `MAGICK_CODER_MODULE_PATH`, and spins waiting for a writable cache directory. Both the darwin builder sandbox and the linux build sandbox deny that write, so the call blocks until an internal timeout.

**Fix**

Use `--help` (which parses `argv` before `MagickCoreGenesis` runs) and wrap the test with `timeout(1)` to catch any future regression:

```yaml
test:
  - |
    out=$(timeout 5 mybinary --help 2>&1 || true)
    echo "$out" | grep -iq mybinary
```

Surfaced by #13111 (autotrace).

## Perl XS-extension ABI drift across perl versions

**Symptom**

A bottle that depends on `perl.org` fails at runtime with:

```
Perl API version v5.40.0 of <module>.c does not match v5.42.0
```

**Root cause**

The bottle's CPAN tree includes compiled XS `.so` files. Each carries a `PERL_API_VERSION_STRING` baked in by `XS_VERSION_BOOTCHECK`. When pantry's `perl.org` recipe bumps its major/minor version, the existing bottle becomes ABI-stale and every XS load aborts.

**Fix**

Re-bottle the dependent recipe whenever `perl.org` moves. Either rev-roll via `versions:` to force a rebuild, or — preferable — stop bundling XS modules altogether and declare them as runtime deps with their own recipes so they track perl independently.

Surfaced by #13111 (autotrace, via the `intltool` dep).
