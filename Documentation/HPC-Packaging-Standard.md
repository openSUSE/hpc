# HPC Packaging Standard

This document describes the packaging standard for HPC libraries.

## Goals

HPC compliant mathematical libraries may exist in different 
versions and may be built with different compilers and compiler 
versions. Also different flavors of the same library may exist.

To make it possible that these different versions coexist on
the same system and are available to each individual user
according to his preferences, environment modules are used.

At SUSE we will use Lmod, an environment framework written in
Lua.

## Implementation

### Installation Paths

For different versions and falvors of the same library to be available
concurrently, each library - but also supporting files, like headers,
plugins etc. need to reside in their own directory structure.
- this directory structure starts under `/usr/lib/hpc/`.
- the next element shall be the compiler family name, optionally followed
  by the compiler 'dependency' version (usually the major version): 
  the version number is added only if the version is different 
  from the base compiler for the product (*NOTE:* this may have 
  to be changed to ease migration):
  `<compiler_family>[<compiler_dependency_version>]/`.
- if the package was built with MPI support, this will be followed by 
  the MPI family and its 'dependency' version (usually the major version):
  `[<mpi_family><mpi_dependency_version>]/`.
  (*NOTE1:* If the MPI ABI cannot be considered stable across different
  MPI minor versions, we need to change this to the full version).
  (*NOTE2:* If different flavors of the same MPI family exist, which
  provide a different ABI and thus render MPI-dependent libraries flavor-
  dependent as well, the flavor may be appended to the family name,
  separated by a hyphen: `<mpi_family>[-<flavor>]`. For the puropse
  of depenencies, different flavors with different ABIs may be treated
  as different MPI families.)
- this is followed by the package (ie library) name and an optional
  flavor indentifier (separated from the library name by a hyphen `-`).
  `<package_name>[-<package_flavor>]/`
- the package name directory is followed by a directory representing
  the package version: `<version>/`.
- Below this, the standard directory structure (normally found under
  `$prefix`) will follow.
	

### Package Naming Convention

Libraries should follow this naming sceme to avoid conflicts between
versions and flavors and to allow different versions to be installed
in parallel.

The package name should consist of:
- the source package name
  `<source_package_name>`
- optionally the flavor if the same source package can be built
  in different flavors. The flavor shall be separated from the
  package name by a hyphen:
  `[-<build_flavor>]`
- The package version (with any dot replaced by an underscore.
  The version is separated from the previous parts by an underscore:
  `_<package_version_underscore>`
- The compiler family, separated by a hyphen:
  -<compiler_family>
- If the package was not built with the base compiler, the 'dependency' 
  version of the compiler:
  `[<compiler_dependency_version>]`
- If the package requires MPI support, the MPI family used, separated
  by a hyphen, followed by the major version of the MPI family:
  `[-<mpi_family><mpi_family_version>]` (<mpi_family> may augmented
  by a 'build flavor' if needed, spearated by a hyphen: `<mpi_family>[-<flavor>]`
- followed by the string 'hpc' separated by a hyphen: `-hpc`.

### Environment Module files

Packages need to supply environment modules so that the users 
environment can be set up to locate and use the correct library
versions.

- Base directory for module files (except for compilers) is:
  `/usr/share/lmod/moduledeps/`
- this is followed by a directory according to the compiler family
  used. If a different compiler than the base compiler is used,
  the name of the directory is the name of the compiler family
  combined with the compiler dependecy version (generally the major
  version) separeated by a hyphen:
  `<compiler_family_name>[-<compiler_family_dependency_version>]/`.
- if the library uses MPI, this is folled by a directlry named
  according to the MPI family used combined with the MPI family
  dependency version (generally the MPI family major version) separated
  by a hyphen:
  `[<mpi_family_name>-<mpi_family_dependency_version>]/`
  (*NOTE1:* If different MPI build flavors differ in ABI, the family
  name may be augmented by the build flavor, separated by a hyphen:
  `<mpi_family>-<flavor>`.)
  (*NOTE2:* If different flavors have the same ABI
  care must be taken to package the modules files separately when
  building the MPI library and require this package from each library
  package).
- this is followed by a directory whose name is the name of the
  source package optionally augmented by the build flavor if this
  exists, separated by a hyphen.
  `<package_name>[-<build_flavor>]/`.
- the final element of the directory structure is a directory representing
  the package version: `<version>/`.


