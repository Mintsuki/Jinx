![Logo|128](logo-small.png)

---

Jinx is a "meta-build-system" designed for bootstrapping operation systems. Primarily used by hobby OSDev projects such as [Aero](https://github.com/Andy-Python-Programmer/aero), [Gloire](https://codeberg.org/Ironclad/Gloire), and [Vinix](https://github.com/vlang/vinix), to aid in the creation of an image distribution. Jinx utilises a lightweight Debian "container" to provide a consistent build environment on every host system.

## Table of Contents

Links to different points in this documentation:
- [Acquisition](<#acquisition>)
- [Basic Project Structure](<#basic-project-structure>)
- [Commands](<#commands>)
	- [`build`](<#build>)
	- [`build-if-needed`](<#build-if-needed>)
	- [`rebuild`](<#rebuild>)
	- [`host-build`](<#host-build>)
	- [`host-rebuild`](<#host-rebuild>)
	- [`regenerate`](<#regenerate>)
	- [`install`](<#install>)
	- [`check-updates`](<#check-updates>)
	- [`rebuild-cache`](<#rebuild-cache>)
- [Config](<#config>)
- [Recipes](<#recipes>)
	- [Properties](<#properties>)
		- [`name`](<#name>)
		- [`version`](<#version>)
		- [`revision`](<#revision>)
		- [`skip_pkg_check`](<#skip_pkg_check>)
		- [`source_dir`](<#source_dir>)
		- [`from_source`](<#from_source>)
		- [`tarball_url`](<#tarball_url>)
		- [`tarball_blake2b`](<#tarball_blake2b>)
		- [`tarball_sha256`](<#tarball_sha256>)
		- [`tarball_sha512`](<#tarball_sha512>)
		- [`git_url`](<#git_url>)
		- [`commit`](<#commit>)
		- [`hg_url`](<#hg_url>)
		- [`tag`](<#tag>)
		- [`deps`](<#deps>)
		- [`hostdeps`](<#hostdeps>)
		- [`imagedeps`](<#imagedeps>)
		- [`allow_network`](<#allow_network>)
		- [`source_*`](<#source_*>)
		- [`repology_id`](<#repology_id>)
		- [`repology_srcname`](<#repology_srcname>)
		- [`repology_status`](<#repology_status>)
	- [Functions](<#functions>)
		- [`prepare()`](<#prepare>)
		- [`configure()`](<#configure>)
		- [`build()`](<#build-1>)
		- [`package()`](<#package>)
	- [Host Recipes](<#host-recipes>)
	- [Source Recipes](<#source-recipes>)
	- [Patches](<#patches>)
## Acquisition

Ensure the following prerequisites have been acquired:
- A POSIX-compatible shell located at `/bin/sh`.
- `curl`.
- `find` and `xargs` from a `findutils` package.
- `awk`.
- LLVM Clang, or the GNU C Compiler.
- `grep`.
- `gzip`.
- `tar`.
- `free` from a `procps` package.
- `xz`.

>[!warning]
>It is ***imperative*** that jinx be run under Linux, as the container environment relies on non-POSIX Linux features. You ***will*** run into issues on other systems.

Download the `jinx` executable script from https://codeberg.org/mintsuki/jinx. Ensure that it is marked as executable with `chmod +x jinx`. Finally, verify that it is functional with `./jinx help`, to display the help output.

## Basic Project Structure

Within a basic Jinx project, the bare minimum project structure is as follows:

```
example/
├── host-recipes/
├── jinx
├── jinx-config
├── patches/
├── recipes/
└── source-recipes/
```

- `host-recipes/` -> Recipes that produce results destined for usage on the host system. Typically for cross-compiler toolchain building. See [Host Recipes](<#host recipes>).

- `patches/` -> Contains patches for sources to be applied pre-[`prepare()`](<#prepare>). See [Patches](<#patches>).

- `recipes/` -> Generic recipes for most purposes. Will be handled exclusively within the container environment, however, they can act as sources for host recipes if referenced within the `from_source` property. See [Recipes](<#recipes>).

- `source-recipes/` -> Recipes that only act to provide prepared sources for other recipes, to later be built. Referenced by the [`from_source`](<#from_source>) property within a recipe. See [Source Recipes](<#source-recipes>).

- `jinx` -> Jinx executable, run `./jinx help` for commands. Additionally, see [Commands](<#commands>).

- `jinx-config` -> Jinx configuration file. Place global variables and functions referenced in recipes here. See [Config](<#config>).

## Commands

A brief summary for each command can be found within the usage output of `./jinx help`:

```
usage: ./jinx <command> <package>

   help|--help        Displays this message
   version|--version  Prints the version
   build              Builds package(s), does incremental builds
   build-if-needed    Builds package(s) if necessary
   rebuild            Rebuilds package(s)
   host-build         Same as build, but for host package(s)
   host-rebuild       Same as rebuild, but for host package(s)
   regenerate|regen   Regenerates patch for package(s) and re-runs prepare step
   install            Installs package(s)
   check-updates      Checks on repology whether package(s) are up to date
   rebuild-cache      Rebuilds Jinx cache

example: "./jinx check-updates '*'"
```

However, additional descriptions for each command are as follows:

#### `build`

Followed by as many arguments as recipes to build. `build` will *not* attempt to build recipes within `host-recipes/` unless a host recipe is a specified dependency of a to-be-built recipe.

#### `build-if-needed`

Identical functionality as `build`. Except, recipes will *only* be built if the specified recipe has either: not already been built, or the recipe is of a newer version. Other recipes will be skipped.

#### `rebuild`

Fairly self-explanatory. Essentially just the `build` command, but it will also force the recipe to run the recipe's [`configure()`](<#configure>) logic again (by removing the old build directory).

#### `host-build`

Normal `build`, but for `host-recipes`, as described by brief summary.

#### `host-rebuild`

Normal `rebuild`, but for `host-recipes`, as described by brief summary.

#### `regenerate`

- Alias: `regen`

*Only* runs initial source preparation steps. This could preface [`rebuild`](<#rebuild>) or [`host-rebuild`](<#host-rebuild>) when you must make in-place changes.

#### `install`

While not specified in the brief summary, `install` can take a `-f` flag to **force** the installation of a package (sometimes needed when reinstalling packages). This will only work with normal recipes, not host recipes.

>[!warning]
>The `install` command takes an additional argument preceding the list of packages to install. A "system root" directory **must** be specified for Jinx to determine where to install the packages (this directory will contain the file structure packaged into a distribution).
>
>The usage of the command is as follows: `./jinx install <sysroot> <package(s)>`, where `sysroot` specifies this "system root" directory. For example: `./jinx install "sysroot/" base` to install the `base` package (specified by `recipes/base`) into the `sysroot/` "system root" directory.

#### `check-updates`

Check against the https://repology.org API to see if the specified packages are up to date. This is only a helpful command for maintenance purposes, and performs no additional function on its own.

#### `rebuild-cache`

Reset Jinx back to its first use state. Will only purge the `.jinx-cache/` directory.

>[!tip]
>An argument of `'*'` can be used in place of recipe names to alias for every known recipe of its type. This is great for "full" distributions.

## Config

Jinx draws from the `jinx-config` file within the project directory when building **any** recipe. Variables and functions defined within this file will be sourced by recipes, allowing the definition of default compiler flags, or helper functions. As with recipes, the `jinx-config` file is a shell script, and supports its syntax.

A *bare minimum* config file is as follows:

```sh
#! /bin/sh

# Minimum `jinx-config` for Jinx 0.5.x.

# **REQUIRED** by jinx. This serves to ensure that a major update of Jinx isn't working with an out-of-date project.
JINX_MAJOR_VER=0.5 # 0.5 is the latest major version of Jinx, as of writing.

# **REQUIRED** by jinx for internal package management logic.
# This specifies the *target* architecture, not the host system's architecture.
JINX_ARCH=x86_64

# Any further logic or variable definitions will *also* be passed to every recipe.
```

>[!note]
>It is worth keeping in mind that helper functions referenced in recipes will be run inside the container, and will *not* run on the host.

>[!tip]
>A config file other than the default `jinx-config` can be specified as an environment variable when running `./jinx`. For example: `JINX_CONFIG_FILE="jinx-config-riscv64" ./jinx build '*'`.
>
>This can be valuable for multi-architecture builds, where a different set of helper functions and global variables would need to be specified.

## Recipes

In the Jinx build system, packages are specified by shell scripts called "recipes". There are several types of recipes, but all follow the same sort of system.

### Properties

Defined within the recipe, there are a number of properties that determine how Jinx will handle building the recipe. The full list is below:

>[!note]
>"Properties", as they are referred to in this documentation, are just POSIX shell variables that Jinx makes use of. Thus, any valid shell script syntax for defining these variables will be accepted.

#### `name`

- **Required**.

Basic property to define the name of the package that the recipe specifies. There is an expectation that the value of this property is equal to the file name of the recipe; however, the `name` property is used to find associated [Patches](<#patches>), and to identify the specific build and source directories (note: not source recipes) for the recipe.

Example:

```sh
#! /bin/sh
# recipes/test

name=test
# ...
```

#### `version`

- **Required**, but not for host recipes.

Basic property used to specify the version of this recipe. This **should** be set to the version number of the source this package is for (eg. `version=2.69` for `autoconf-2.69`). The `version` property is used by Jinx internally to determine whether to rebuild a package following a version bump. Additionally, the `version` property can be compared against the latest reported version from https://repology.org (see [Commands](<#commands>)).

Example:

```sh
#! /bin/sh
# recipes/test

# ...
version=2.4.0
# ...
```

#### `revision`

- **Required.**

Provides no special functionality, but is **required** for internal usage.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
revision=1
# ...
```

#### `skip_pkg_check`

- **Optional**.
- `yes`/`no

If the value of this property is `yes`, this package will be skipped during a [`check-updates`](<#check-updates>) command.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
skip_pkg_check=yes
# ...
```

#### `source_dir`

- **Optional.**
- Overrides other source options.

Tells Jinx to use the source directory specified in the recipe, noting that it'll now be a local package. This is useful for building code written for the operating system, with Jinx.

>[!warning]
>Keep in mind that files *outside* of the project directory will be **inaccessible** within the build container enforced upon *all* recipes.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
source_dir="test/"
# ...
```

>[!tip]
>This works really well for building the kernel from a project directory, especially for OSDev projects.
>
>For example:
>
>```sh
>#! /bin/sh
># recipes/kernel
>
>name=kernel
>version=0
>revision=1
>skip_pkg_check=yes # Repology won't ever have a package for this
>source_dir="kernel" # Our kernel project is within the `kernel/` directory.
>
># ...
>```

#### `from_source`

- **Optional**.

Specifies another recipe to draw from for build sources. This can be a specialised source recipe, or a normal recipe.

>[!note]
>Jinx will prioritise a source recipe **over** a normal recipe when searching for a recipe specified by this property. Meaning, a normal recipe of the same name as a source recipe can specify the latter in the `from_source` property, without issues.

>[!warning]
>Host recipes are **not** valid sources for the `from_source` property.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
from_source="test" # Assuming there is a source-recipes/test recipe.
# ...
```

#### `tarball_url`

- **Optional**.

Specifies a URL from which to download a tarball, to use a source for this recipe. A checksum will be used to verify whether the source is valid or not, and only then extract the source.

>[!warning]
>A method of checksum validation **must** be specified. This can be done by specifying a checksum in either the [`tarball_blake2b`](<#tarball_blake2b>), [`tarball_sha256`](<#tarball_sha256>), or [`tarball_sha512`](<#tarball_sha512>) property.

>[!tip]
>The [`version`](<#version>) property can be embedded within this property for the sake of convenience.
>
>For example:
>```sh
>#! /bin/sh
># recipes/xtrans
>
># ...
>version=1.6.0
>tarball_url="https://www.x.org/archive/individual/lib/xtrans-${version}.tar.gz"
># ...
>```
#### `tarball_blake2b`

- **Required** for [`tarball_url`](<#tarball_url>) checksum.

Specifies a BLAKE2B checksum for verifying the tarball.

Example:

```sh
#! /bin/sh
# recipes/xtrans

# ...
tarball_blake2b="446035fb78ec796c1534f36dc687b40fbe6227d47a623039314117a85cc4b3e37971934790932e46a6dc362de70dfb58ccd1fae43518461789ce8854e27adba8"
# ...
```

#### `tarball_sha256`

- **Required** for [`tarball_url`](<#tarball_url>) checksum.

Specifies a SHA256 checksum for verifying the tarball.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
tarball_sha256="97efeda496274082e4ed0edf641a7ce5559d4b030fd6b16547e2f13c6d9d00d5"
# ...
```

#### `tarball_sha512`

- **Required** for [`tarball_url`](<#tarball_url>) checksum.

Specifies a SHA512 checksum for verifying the tarball.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
tarball_sha512="172acc1bc70350b1f7e46063e98c4a5ce4dd3c245a7e7bd383d8fce4a44ea0d46057b55af0aa279cda1d4b1413e924377ccce9562cef9e83eb6e30fc136a383c"
# ...
```

#### `git_url`

- **Optional**.

URL to a Git repository that will be cloned to be used as the source for this recipe. Requires a [`commit`](<#commit>) property to specify the commit to checkout.

Example:

```sh
#! /bin/sh
# source-recipes/libgcc-binaries

# ...
git_url="https://codeberg.org/osdev/libgcc-binaries.git"
# ...
```
#### `commit`

- **Required** for [`git_url`](<#git_url>).

Specifies the commit to checkout from a Git repository specified by the [`git_url`](<#git_url>)

Example:

```sh
#! /bin/sh
# source-recipes/libgcc-binaries

# ...
commit="28257019ce04f784337cb9c3125abb4d02cef14d"
# ...
```

#### `hg_url`

- **Optional**.

URL to a Mercurial repository that will be cloned to be used as the source for this recipe. Requires a [`tag`](<#tag>) property to specify the commit to checkout.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
hg_url="https://repo.mercurial-scm.org/hg/"
# ...
```
#### `tag`

- **Required** for [`hg_url`](<#hg_url>).

Specifies the tag to update to for a Mercurial repository specified by the [`hg_url`](<#hg_url>)

Example:

```sh
#! /bin/sh
# recipes/test

# ...
tag="tip"
# ...
```

#### `deps`

- **Optional.**
- Space-separated list of recipes.

Specifies other normal recipes that should be built before this host/normal recipe will work. Dependencies in the `deps` property must **only** be normal recipes.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
deps="thing1 thing2"
# ...
```

#### `hostdeps`

- **Optional**.
- Space-separated list of recipes.

Specifies host recipes that must be built before this host/normal will work. Dependencies in the `hostdeps` property must **only** be host recipes.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
hostdeps="thing1 thing2"
# ...
```

#### `imagedeps`

- **Optional**.
- Space-separated list of packages.

Specifies Debian packages that must be installed into the container environment before this host/normal recipe will work. This could be something like `build-essential` before a recipe that needs to compile something with `gcc`.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
imagedeps="build-essential patchelf"
# ...
```

#### `allow_network`

- **Optional**.
- `yes`/`no`

If the value of this property is `yes`, then the container will be given access to the internet. Otherwise, the container will remain isolated.

>[!warning]
>This should not be assumed to provide security when building potentially unsafe recipes. The container is **not** a virtual machine.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
allow_network=yes # This recipe will be allowed access to the internet.
# ...
```

#### `source_*`

- **Optional**.

Recipes may also provide `source_deps`, `source_imagedeps`, `source_hostdeps`, and `source_allow_network` properties. These will **only** have an effect when part of a normal recipe. Within a recipe, these properties will be applied only to the [`prepare()`](<#prepare>) stage; instead, if a recipe with these properties is referenced by another using the [`from_source`](<#from_source>) property, these properties will be used to override **all** of their associated properties within the including recipe.

#### `repology_id`

- **Optional**.

Overrides the name to search on https://repology.org, when checking for package updates. By default, the search is for the [`name`](<#name>) property. Use this when the name of the recipe and the package name differ.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
name=test
repology_id=tester # Name on repology differs from the name of the recipe.
# ...
```

#### `repology_srcname`

- **Optional**.

Override default Repology search to include a specific source package name associated with the desired package. Used for narrowing down search results for the [`check-updates`](<#check-updates>) command.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
repology_srcname=testconf # This is the source package name relevant to this package.
# ...
```

#### `repology_status`

- **Optional**.
- `newest`/`devel`/`unique`/`outdated`/`legacy`/`rolling`/`noscheme`/`incorrect`/`untrusted`/`ignored`

Override default Repology search to include a specific package status associated with the desired package. Used for narrowing down search results for the [`check-updates`](<#check-updates>) command.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
repology_status=newest # Look for package marked as newest.
# ...
```


### Functions

Within a Jinx recipe, the recipe may specify the logic for different stages in package building in named functions. Functions may reference the recipe's properties or globally defined variables and functions within the [configuration file](<#config>), following typical POSIX shell script syntax.

```mermaid
flowchart LR
	A("prepare()") --> B
	B("configure()") --> C
	C("build()") --> D
	D("package()")
```
Recipe build logic flow, pictured above.

> [!note]
>No function is technically ***required*** to exist for a build to succeed; however, without functionality, the recipe does nothing. This works in the favour of [Source Recipes](<#source-recipes>).

> [!warning]
> Aside from the [`prepare()`](<#prepare>) function, these functions will all run within the build directory for the recipe. This means that the directory that the other functions drop code into will **not** contain the sources by default, and will have to be referenced like in the example: `${source_dir}/configure`. It is recommended that only the [`prepare()`](<#prepare>) step modifies the source directory.

#### `prepare()`

The first bit of function logic to ever run from a recipe. This function is run with dependencies specified in [`source_*`](<#source_*>) properties, regardless of whether it's referenced as a source or not. `prepare()` is run following source acquisition and the application of [patches](<#patches>), but before any other steps. Runs within the directory containing the recipe sources.

>[!tip]
>The `prepare()` function is an optimal place to do any further preparation of sources that a [patch](<#patches>) is unable to do.

Example:

```sh
#! /bin/sh
# source-recipes/test

# ...
allow_network=yes
# ...
prepare() {
	./download_prerequisites # Here, the prepare() stage is used to download anything needed for the later configure() stage.
	
	rm -rf src/broken.c # Removes unwanted files.
}
# ...
```

#### `configure()`

Following the [`prepare()`](<#prepare>) stage, `configure()` is intended for getting the sources ready for building. As in the name, the `configure()` stage is most suited to projects that require a `./configure` before builds. This stage is **only** called the **first** time a recipe is built (with the exception of using the [`rebuild`](<#rebuild>) command).

Example:

```sh
#! /bin/sh
# recipes/test

# ...
configure() {
	CFLAGS="$TARGET_CFLAGS" \ # Reference global TARGET_CFLAGS from `jinx-config`.
	LDFLAGS="$TARGET_LDFLAGS" \
	"${source_dir}"/configure \
	--prefix="${prefix}" # Configure for installation into Jinx default prefix.
}
# ...
```

>[!tip]
>It is good practice to reference the internal `prefix` property during `configure()` as it will point to `/usr/local` within the "system root" during installation; however, this can be whatever you desire.

#### `build()`

Run every build, the build function is to contain the logic used to build the recipe.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
build() {
	# work
	make -j${parallelism} # Build with a level of parallelism precalculated by Jinx.
}
# ...
```

>[!tip]
>Referencing the internal `parallelism` property will allow the inclusion of a pre-calculated optimal number of parallel jobs.

#### `package()`

Final stage in a recipe. After all other logic has been run, the `package()` stage is used to install the build results into an XBPS package for proper installation with the [`install`](<#install>) command. The location to install the build results is defined by the internal `dest_dir` property.

Example:

```sh
#! /bin/sh
# recipes/test

# ...
package() {
	DESTDIR="${dest_dir}" make install # Install build results to package directory.
}
# ...
```

>[!tip]
>The `package()` function can also be used for any post-build steps like binary stripping.

### Host Recipes

Located in `host-recipes/`, host recipes are unlike normal recipes in that they describe packages that should be built for the host system, as opposed to the target system. Regardless, the recipes will still be built within the container. Host recipes are valuable for utilising Jinx for building cross compiler toolchains or other utilities.

Host recipes can **not** be used in the [`from_source`](<#from_source>) property, but will accept the property if it references valid normal or source recipes.

>[!warning]
>Linking against dynamic shared libraries *may* have unexpected behaviour; running the built executable could have it searching for a library you might not have on the host system.

>[!note]
>Outputs from host recipes will end up in the `host-pkgs/<name of recipe>/` directory and can be run from here. For example: `./host-pkgs/limine/usr/local/bin/limine bios-install testos.iso`.

### Source Recipes

Located in `source-recipes/`, source recipes are not built as recipes themselves, rather, they provide sources (and perhaps a [`prepare()`](<#prepare>) stage). This type of recipe is prioritised over normal recipes when searching for a matching recipe in a [`from_source`](<#from_source>) property.

### Patches

Incredibly useful for OSDev projects, Jinx provides a fairly reasonable way to patch existing software sources for the needs of the target system. For each recipe that is to have its sources patched, there must be a `patches/<name>/jinx-working-patch.patch` file (where `<name>` is the [`name`](<#name>) property of the recipe that provides the source) filled with a git diff patch. Patches will be applied *before* the [`prepare()`](<#prepare>) stage in a recipe.

>[!tip]
>Patches can be created with the `diff` command like in the following example:
>
>```sh
>diff -urN --no-dereference binutils/ binutils-modified/ > patches/binutils/jinx-working-patch.patch
>```
>
>Here, `binutils/` is the original, unmodified version of the source, and `binutils-modified/` contains the modification we desire to turn into a patch.