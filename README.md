## Jinx

![Jinx logo](/logo-small.png?raw=true "Jinx logo")

Jinx (Jinx Is Not Xbstrap) is a meta-build-system for bootstrapping operating
system distributions inspired by [xbstrap](https://github.com/managarm/xbstrap),
Void Linux's [void-packages](https://github.com/void-linux/void-packages),
and Arch Linux's [PKGBUILDs](https://wiki.archlinux.org/title/PKGBUILD).

An example OS using Jinx is [Gloire](https://codeberg.org/Ironclad/Gloire).

### Dependencies
- Linux distro (other OSes are not supported)
- `bash` (the script itself is bash)
- POSIX compatible `/bin/sh` (used inside containers for mount/chroot setup)
- awk
- findutils (for `find` and `xargs`)
- git
- GNU make
- grep
- gzip
- sed
- tar
- wget
- zstd
- coreutils or equivalent (for `sha256sum`/`sha256`)
- procps or equivalent (for `free`)
- util-linux or equivalent (for `unshare`)

### Documentation

Documentation can be found in [DOCUMENTATION.md](DOCUMENTATION.md).
