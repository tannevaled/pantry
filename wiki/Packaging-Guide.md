# Tips

- Consider the build a black box (as much as possible)
    - Only change inputs (configure args)
    - If it does things wrong that can’t be changed with inputs modify the outputs
    - Don’t mess with the innards
    - Rationale: the maintainer will update the innards and your hacks will break
    - Rationale: we don’t know much about the innards: changes will have unforeseen consequences

## Drop libtool `.la` files

**Symptom**

A downstream link fails with:

```
cannot find .../<dep>/v<old-version>+brewing/lib/lib<dep>.la
```

…even though `<dep>` is installed and its `.pc` file resolves correctly.

**Root cause**

Each `lib*.la` file embeds the absolute path of every transitive dependency at the moment the producing recipe was bottled, including the build-sandbox `+brewing` suffix and the exact version of every transitive dep. Both bits rot post-install: pantry's bottle relocation strips `+brewing`, and downstream deps move forward independently. libtool then refuses to link because the recorded path no longer exists.

Modern `pkg-config` carries the same dependency information from `.pc` files. `.la` files have been redundant since libtool 2.4 (2015) and most distros now delete them at packaging time.

**Fix** — in any C-library recipe, after install:

```yaml
build:
  - find {{prefix}}/lib -name '*.la' -delete
```

Retrospective on #7443 (x11.pc); resurfaced by #13111 (autotrace, via ImageMagick's bottle).

## Porting glibc-isms to darwin

**Symptom**

A C project builds on linux but fails to compile on darwin because it relies on glibc-only interfaces — most commonly `<argp.h>`, the GNU long-option style of `<getopt.h>`, or `<error.h>`.

**Root cause**

These headers ship with glibc but not with darwin's libc. Patching them out package-by-package is fragile; the upstream-friendly approach is to vendor a portable shim so the same source builds unchanged on both platforms.

**Fix**

If upstream uses meson, lean on its `wrap-file` subproject mechanism. For argp, zchunk vendors a `subprojects/argp-standalone.wrap`:

```ini
[wrap-file]
directory = argp-standalone-1.5.0
source_url = https://www.lysator.liu.se/~nisse/misc/argp-standalone-1.5.0.tar.gz
source_filename = argp-standalone-1.5.0.tar.gz
source_hash = ...

[provide]
argp = argp_dep
```

`meson.build` then probes the system first (`cc.has_header('argp.h')`) and only falls back to the wrap when the probe fails. On linux glibc the wrap is never expanded; on darwin it builds a tiny static `libargp.a` linked into the final binary.

For autotools projects the equivalent pattern is a `gnulib`-imported module (`bootstrap --import argp`). Both keep the source tree portable without per-platform `#ifdef` forests.

Surfaced by #13113 (zchunk).
