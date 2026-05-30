A package is a release of an open source package, built into binaries for the platforms tea supports.

# Getting started

Firstly clone the pantry

```sh
git clone https://github.com/pkgxdev/pantry
cd pantry
dev 
```

Now you need to create a new `package.yml` in the `./projects` directory, we have a helper:

```sh
$ bk init
# ^^ creates a new “wip” package

# or…

$ bk init foo.com
# ^^ creates ./projects/foo.com/package.yml
```

We name packages after their homepages. If the project has no homepage we name it after
their `github.com/user/repo`. Add subdirectories if you need namespacing, eg. `gnu.org/wget`.

Firstly we need to create a new `package.yml` entry in your local clone of the pantry.

# Editing your pkg

```sh
bk edit
```

Opens up the `package.yml` for what you are working on in your `$EDITOR`.

* Generally we’d recommend searching the pantry for existing packages that are similar to yours.
* [Structure of a pkg](https://github.com/pkgxdev/pantry/wiki/Structure-of-a-%60package.yml%60)
* [Examples](https://github.com/pkgxdev/pantry/wiki/Examples)
* [Packaging Guide](https://github.com/pkgxdev/pantry/wiki/Packaging-Guide)

Here is a simple example `package.yml`:

```yaml
distributable:
  url: https://example.com/download/{{version}}/src.tar.gz
  strip-components: 1

versions:
  - 1.0.0

build:
  script: |
    touch "{{prefix}}"/example
  # ^^ this script must install something to `{{prefix}}` or the build will be failed!

test:
  script: |
    ls -l
```

# Building your pkg

```sh
bk build
```

> If your package grabs version information from GitHub you’ll need a [PAT].
> Either ensure `GITHUB_TOKEN` is set in your environment or `gh auth login` first.

* Downloads the `distributable` to `./srcs`
* Builds the srcs to `./builds`
     * Builds run via a script called `pkgx.com/pkg.com+platform/dev.pkgx.build.sh`
     * Successive runs of `bk build` do *not* clean first!†
* You can use docker to run the build on Linux with `bk -L build`

> †  We do this because it usually makes the successive builds faster and it makes debugging builds easier.

# Troubleshooting the build

* Usually it's easiest to just edit the `package.yml` and re-run `bk build`
* However you can step into the `./builds/pkg.com/pkg.com+platform` and:
    * edit `dev.pkgx.build.sh` and run it yourself
    * try out build commands yourself, eg. `./configure --help`

> TIP: If you need to create the env of the pkg then step into the build directory, edit the build script
> and remove everything but the `export` commands, then source it (`source dev.pkgx.build.sh`)

# Testing your pkg

Our CI/CD infra requires a `test` YAML node. This script should thoroughly verify all
the functionality of the package is working. You can run the test with:

```sh
bk test
```


# Contributing

Push and create a pull request.

Our CI/CD will build and test on all platforms we support.

We prefer you make the package work on all platforms but if it doesn’t we’ll merge whatever works. Someone else can make it work when they want that platform.

We require all packages be relocatable.
Our CI will verify this for you. You can check locally by moving the installation from
`~/.pkgx` to another tea installation (eg. `~/scratch/pkgx`§ and running the
test again.

> § `PKGX_PREFIX=~/scratch/pkgx sh <(curl pkgx.sh)`



# After Your Contribution

We build “bottles” (tar’d binaries) and upload them to both our centralized
bottle storage and decentralized [IPFS].

tea automatically builds new releases of packages *as soon as they are
released* (usually starting the builds within seconds). There is no need to
submit PRs for updates.

[pantry]: https://github.com/pkgx/pantry
[pkgx]: https://github.com/pkgx/pkgx
