## Jinx

![Jinx logo](/logo-small.png?raw=true "Jinx logo")

Jinx (Jinx Is Not Xbstrap) is a meta-build-system for bootstrapping operating
system distributions inspired by [xbstrap](https://github.com/managarm/xbstrap),
Void Linux's [void-packages](https://github.com/void-linux/void-packages),
and Arch Linux's [PKGBUILDs](https://wiki.archlinux.org/title/PKGBUILD).

An example OS using Jinx is [Gloire](https://codeberg.org/Ironclad/Gloire).

### Dependencies
- Linux distro (other OSes are not supported)
- POSIX compatible `/bin/sh`
- awk
- working cc (gcc or clang work)
- curl
- debootstrap
- findutils (for `find` and `xargs`)
- git
- GNU make
- grep
- gzip
- pkg-config (or `pkgconf`)
- tar
- procps or equivalent (for `free`)
- util-linux or equivalent (for `unshare`)
- `libarchive(-dev)`
- `libssl(-dev)`/`openssl(-dev)`
- `zlib(-dev)` (AKA `zlib1g-dev` on Debian-based distros)

### Documentation

Documentation can be found in [DOCUMENTATION.md](DOCUMENTATION.md).
