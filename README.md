# Kotlin eselect Module

`eselect-kotlin` is a Gentoo [eselect][eselect] module which allows users to
select a default Kotlin compiler to use from all compilers installed on a
Gentoo system.

[eselect]: https://wiki.gentoo.org/wiki/Project:Eselect

## Installation

`eselect-kotlin` is automatically installed with a Kotlin compiler package for
Gentoo.  For more details about installing Kotlin packages on Gentoo, please
consult [relevant information on Gentoo Wiki][wiki-kotlin-install].

[wiki-kotlin-install]: https://wiki.gentoo.org/wiki/User:Leo3418/Kotlin#Installation

## Basic Usage

The Kotlin eselect module can be invoked with a family of `eselect kotlin`
commands.  The basic actions for this module include:

- `eselect kotlin list`
- `eselect kotlin show`
- `eselect kotlin set`

For a synopsis of the eselect module's usage, please run `eselect kotlin help`.
For detailed instructions, please refer to [the relevant
section][wiki-kotlin-eselect] in the Kotlin page on Gentoo Wiki.

[wiki-kotlin-eselect]: https://wiki.gentoo.org/wiki/User:Leo3418/Kotlin#Eselect

## Gentoo Kotlin Compiler Package Specification

The following information is mainly intended for maintainers of Kotlin packages
to help them integrate the packages with this eselect module.  The requirements
for Kotlin compiler packages are marked with keywords in bold, like **must**
and **must not**.

### Compiler Preferences

This module implements selection of preferred Kotlin compiler package for the
following options:

- **User compiler**: The compiler preference for a non-root user account.  The
  `root` account is not permitted to have a user compiler preference.
- **System compiler**: The compiler preference for all user accounts without a
  user compiler preference, including the `root` account.
- **Compiler for a feature release**: The default compiler to be used for a
  specific Kotlin feature release version (e.g. 1.4, 1.5) that applies
  globally to the entire system.  This is useful when multiple Kotlin compiler
  packages for the same Kotlin feature release are installed on the system.

A compiler preference is stored as a symbolic link to the top directory of the
Kotlin compiler installation.  Each type of compiler preference is stored by
this eselect module at the path specified as follows:

- **User compiler**: `${HOME}/.gentoo${EPREFIX}/kotlin/home`
- **System compiler**: `${EROOT}/etc/eselect/kotlin/homes/system`
- **Compiler for a feature release**, e.g. x.y:
  `${EROOT}/etc/eselect/kotlin/homes/x.y`

#### Preference Validity

A valid preference is a symbolic link that points to a directory.  Files that
are not symbolic links, broken symbolic links, and symbolic links whose target
is not an existing directory are all invalid preferences.

The following compiler preferences **must** be valid all the time, except when
this eselect module is performing an operation, or a compiler package is being
installed or uninstalled:
- The system compiler preference, if any Kotlin compiler package is installed
  on the system.
- The feature release compiler preference for every feature release with at
  least one compiler package for it installed on the system.

In addition, if no compiler packages for a feature release exist on the system,
then the feature release **must not** have any pertinent symbolic link for its
preference, valid or not, remaining on the system.  This requirement allows
programs to query if a Kotlin compiler for a feature release x.y is available
by checking the existence of the `${EROOT}/etc/eselect/kotlin/homes/x.y`
symbolic link.

The above requirements may be met after invoking `eselect kotlin cleanup`
and `eselect kotlin update`.  The former command removes all invalid
preferences, whereas the latter one creates symbolic links for the required
preferences at correct paths.  However, please note that `eselect kotlin
update` is permitted **to not overwrite** any existing file that is not a
symbolic link and then exit with an error should this occur.

### Compiler Package Description Files

Each Kotlin compiler package on Gentoo **must** install a *package description
file* under the `${EROOT}/usr/share/eselect-kotlin/pkgs/x.y` directory where
`x.y` is the feature release the compiler is for.  The package description
file's base name **must** be the same as the last component of the path to the
top directory of the Kotlin compiler installation.

For example, assume `dev-java/kronstadt-1.1.8` is a Kotlin compiler package for
Kotlin feature release 1.5 that is installed to `/usr/share/kronstadt-1.1`,
then its package description file shall be installed at
`${EROOT}/usr/share/eselect-kotlin/pkgs/1.5/kronstadt-1.1`.

The package description file **must** be a file that can be passed as the
file name to the `source` Bash built-in command without any errors, and it
**must** set the following variables in any environment where `source` is
called against it:

- `GENTOO_KOTLIN_HOME`: The top directory of the Kotlin compiler installed by
  the package.

*Note*: In `src_*` phases for an ebuild, please use `ED` to replace `EROOT`
because the ebuild is not permitted to use `ROOT`, `EROOT` or any related
variables in those phases.  When the package's files are being installed to the
system, files in `ED` will be copied into `EROOT`.

### Kotlin Tools Recognized by This Module

This module supports the following Kotlin tool commands:

- `kapt`
- `kotlin`
- `kotlinc`
- `kotlinc-js`
- `kotlinc-jvm`
- `kotlin-dce-js`

The executables for those commands are installed by **this module** into
`/usr/bin`.  Kotlin compiler packages **must not** install executables with the
same name into `/usr/bin`.

#### Kotlin Tool Launcher Script

To honor Kotlin compiler preference for different users, the system and all
installed Kotlin feature releases, a Kotlin tool launcher script that reads the
preferences and selects the compiler accordingly is installed to
`${EROOT}/usr/libexec/eselect-kotlin/run-kotlin-tool.sh`, and the Kotlin tool
executables are installed as symbolic links to this script.

The Kotlin tool launcher script selects the Kotlin compiler package using the
following rules, in order:

1. If the Kotlin tool command being invoked is versioned, i.e. it contains the
   Kotlin feature release version number at the end of its name (see the next
   section for more details), then it uses the preferred compiler package for
   the feature release indicated in the command name.

2. If the `GENTOO_KOTLIN_VER` environment variable is set to a valid Kotlin
   feature release version number, then it uses the preferred compiler package
   for the indicated feature release.  The variable's value can also be
   `system`, in which case the package selected by the system compiler
   preference is used.

3. If the user compiler preference is set, then the package selected by it is
   used.

4. The package selected by the system compiler preference is used.

It is an error if any of the following conditions is met:
- The Kotlin tool launcher script is invoked directly.
- The Kotlin version specification (set either by the versioned command's name
  or `GENTOO_KOTLIN_VER`) or the user/system compiler preference picked by the
  script is invalid.
- The tool command does not exist in the `bin` directory under the top
  directory of the Kotlin compiler installation selected by the script.

#### Versioned Tool Commands

This module also supports versioned commands for each of the Kotlin tools
listed above whose names are in the format of `<tool-name><feature-release>`.
For example, `kotlinc1.5` is a versioned command supported by this module for
`kotlinc` from a Kotlin compiler package for Kotlin feature release 1.5.

This module neither installs nor requires versioned commands.  Kotlin compiler
packages may install versioned commands that adhere to the name format by
creating symbolic links to the Kotlin tool launcher script.

*Note*: Great care must be exercised if multiple Kotlin compiler packages for
the same feature release that install the versioned commands exist.  If each
package installs its own set of versioned commands under the same location,
then it will likely cause a file collision between packages.  It is recommended
that the versioned commands are not associated with any specific Kotlin
compiler package.  Possible ways of achieving this include making a separate
package dedicated for the versioned commands and creating an eclass for Kotlin
compiler packages that manage the versioned commands without registering them
as a package's files.
