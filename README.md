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
- curl
- debootstrap
- fakechroot
- fakeroot
- findutils (for `find` and `xargs`)
- awk
- cc (gcc or clang work)
- git
- grep
- gzip
- tar
- unshare (from `util-linux`)
- procps (for `free`)
- xz

### Documentation

Documentation can be found in [DOCUMENTATION.md](DOCUMENTATION.md).
