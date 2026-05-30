The `package.yml` file is the core of building new software. They can range from
the exceedingly simple ([just.systems/package.yml](https://github.com/pkgxdev/pantry/blob/main/projects/just.systems/package.yml)),
to the very, very complicated ([python.org/package.yml](https://github.com/pkgxdev/pantry/blob/main/projects/python.org/package.yml)),
and everything in between. Generally, they are very simple, as package
maintainers do a lot internally to simplify building for the benefit of their users.

Here, we'll break down the structure of the YAML, so you can quickly build a new
package.

## A note about moustaches

[`useMoustaches.ts`](https://github.com/pkgxdev/libpkgx/blob/main/src/hooks/useMoustaches.ts)
defines a number of mappings usable within a `package.yml` file to prevent
hard-coding variant values. In particular, every build/runtime dependency is
also included under the `deps` key, such as `{{deps.invisible-island.net/ncurses.version.major}}`.

The available mappings are as follows.

- deps
- hw.arch (`aarch64`|`x86-64`)
- hw.target
- hw.platform (`darwin`|`linux`|`windows`)
- hw.concurrency (number of CPU cores)
- prefix
- version.build
- version.major
- version.marketing (major.minor)
- version.minor
- version.patch
- version.raw
- version.tag

Note: `hw` may be replaced with `host` in the future.

## `package.yml` keys

### `distributable`

`distributable` is the key that tells `tea` how to find the sources. Generally,
it has two sub-keys: `url`, and `strip-components`. The value of `url` tells the
build script how to find the sources, and generally uses `{{version}}` or
`{{version.raw}}` to generalize the URL. `{{version}}` is processed and will
appear in the form x.x.x. `strip-components` is _nearly always_
`1`, and strips the package name from the unpacked directory structure, as
expected by the other tools.

### `versions`

There are two variants of the `versions` key in use today: an array of text
versions, such as:

```yaml
versions:
  - 1.1.0
  - 1.1.2
  - 1.2.0
```

and a version that can parse GitHub releases or tags, as e.g.: `nodejs/node` or
`python/cpython/tags`. This can be coupled with a `strip` key, which describes
data to remove from the tag, as in `curl`'s: `strip: /^curl /` Future
development will likely allow for complex version parsers, as shown in comments
in [openssl.org/package.yml](https://github.com/pkgxdev/pantry/blob/main/projects/openssl.org/package.yml).

### `provides`

`provides` is an array of text keys, listing the binaries provided in the final
package, relative to the `{{prefix}}`. It is used by `pkgx -X` to locate executables.

### `platforms`

`platforms` is a single text or an array of text keys. Some packages won't build everywhere, as much as
we might like them to. This key is a whitelist of available platforms, if the whole gamut
cannot be covered:

```yaml
# FIXME: linux/x86-64 has a subtle bug I cannot figure out at the moment
platform:
  - darwin
  - linux/aarch64
```

### `interprets`

Used by script interpreters, such as `deno.land`, it contains two subkeys:

- `extensions`: either a `string | string[]` of file extensions to call this
  interpreter for.
- `args`: the commands needed to interpret a script; e.g. `args: [deno, run]`.

### `dependencies`

A set of run-time dependencies, in `pkgx` notation: `package-name: semver-string`.
Note that platform- or architecture-specific dependencies can be sub-keyed under
an appropriate descriptor, such as:

```yaml
dependencies:
  invisible-island.net/ncurses: 6
  darwin:
    gnu.org/binutils: '*'
  linux/aarch64:
    github.com/numactl/numactl: '*'
```

### `runtime`

A key to specify run-time specific environment variables, such as:
```yaml
runtime:
  env:
    FOO: bar
```

### `build` and `test`

`build` and `test` function very similarly. They can take a single text string,
which will be the entirety of the build/test by itself, but more commonly they
take some/all of the subkeys below:

#### `dependencies` (subkey)

Some dependencies are only needed for building/testing a package, but not in
general use. These go here, in the same format as [`dependencies`](#user-content-my-paragraph-title).

#### `script`

The main build/test script. This can be arbitrarily complicated, and is likely
to make copious use of [moustaches](#user-content-a-note-about-moustaches). Any
files alongside the `package.yml` in the repository will be available under the
`props/` directory while building. This is useful for providing `diff`s and
other related content.

#### `env`

Control setting the environment for the build/test. Many things are possible
here, including platform/arch-specific configurations, as referenced in
[`dependencies`](#user-content-my-paragraph-title). `env` keys will override
(but may reference, e.g.: `CPATH: $CPATH:/path/to/somewhere`, existing
environment variables). Note that moustaches may be prefixed with `$` to deal
with YAML requirements: `VERSION: ${{version}}`.

#### `fixture`

An additional option for providing files is to provide their text in a `fixture`
key. This file will be available to the script at the location stored in the
`$FIXTURE` environment variable.
