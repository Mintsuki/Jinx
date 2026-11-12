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
- awk
- findutils (for `find` and `xargs`)
- git
- grep
- gzip
- perl (needed by `debootstrap`, and to drop root privileges)
- sed
- tar
- wget
- xz (for `xzcat`, needed by `debootstrap`)
- zstd
- coreutils or equivalent (for `sha256sum` and `chroot`)
- procps or equivalent (for `free`)
- util-linux or equivalent (for `unshare`, `flock` and `mount`)

Busybox counts as an equivalent: Jinx sticks to the commands and options that
busybox implements too, so that an Alpine Linux host, for example, only needs
`bash`, `git`, `perl`, and `zstd` on top of its base system.

### Documentation

Documentation can be found in [DOCUMENTATION.md](DOCUMENTATION.md).
