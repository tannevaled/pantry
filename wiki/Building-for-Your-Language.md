# Make

This is a common pattern.

```
make --jobs {{ hw.concurrency }}
make prefix={{ prefix }} install
```

Moustaches: `hw.concurrency` is the number of CPU cores, `prefix` is the path to the package install location.

# Generic Scripts (Bash/Perl/Python/Ruby/etc)

You might need to `mkdir {{prefix}}/bin`. Move files, don't copy. For example:

```
mkdir {{prefix}}/bin
mv script {{prefix}}/bin
```

Change the first line (shebang line) to? `#!/usr/bin/env perl`?