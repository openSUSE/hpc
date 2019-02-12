# How to package HPC Libraries for SUSE

**IMPORTANT WARNING:** *Please note that this document may change frequently*

## Introduction

To simplify building multiple versions of HPC libraries consistently
utilizing the lmod environment module system, a set of macros have been
created.
Furthermore the [multibuild](http://openbuildservice.org/help/manuals/obs-reference-guide/cha.obs.multibuild.html)
feature of OBS is used. To utilize the macros, the line:
  ```
  BuildRequires:   suse-hpc
  ```
needs to be included in the spec file.
SUSE supports different versions of the same library built with different
compilers and compiler versions and - if applicable - for different flavors
and versions of MPI. Moreover, these libraries can be installed simultaneously.

This has the following implications:

* The package version is included in the package name and install path.
* The compiler name and version is included in the package name and
  install path.
* The MPI name and versions is included in the package name and install
  path.
* Environment modules are created and installed which include these
  directories.

## Step by Step

To set up the build of a specific compiler/MPI combo the following steps
are required (anything in [..] is optional):

1. create an entry to the `_multibuild` file of the form:

   ```
   <package>
   $compiler_family[${compiler_family_version}]-[${mpi_family}[${mpi_family_version}]]-]hpc[-${other_modifier}]
   </package>
   ```
like:
    ```
    <multibuild>
    ...
    <package>gnu-openmpi3-hpc</package>
    ...
    </multibuild>
   ```

2. In the spec file add a line:
    ```
    %global flavor @BUILD_FLAVOR@%{nil}
    ```

The macro `%{nil}` will avoid that the syntax of the spec file becomes
incorrect if `@BUILD_FLAVOR@` expands to nothing.

3. Define:
    1. Macro `%pname` with the package name:
    ```
    %define pname <package_name>
    ```
    2. Macro `%vers` with the package version:
    ```
    %define vers <package_version>
    ```
    3. Macro `%_vers` with the package version with underscores:
    ```
    %define _vers %(echo %{vers} | tr . _)
    ```
    (This may be replaced with a macro in the future.)

    Except for `%pname` you may use different names for
    all other variables - like `%vers` and `%_vers`,
    they need to be consistent across the spec file for the
    purpose of this document, they are used consistently
    throughout here.

4. for each package line in `_multibuild` add conditionals like:
    ```
    %if "%flavor" == "$compiler_family[${compiler_family_version}]-[${mpi_family}[${mpi_family_version}]]-]hpc[-${other_modifier}]"
    %define compiler_family gnu
    [%define c_f_ver ${compiler_family_version}]
    [%define mpi_family ${mpi_family}]
    [%define mpi_ver ${mpi_ver}]
    ... other defines as needed ...
    %endif
    ```
    like
    ```
    %if "%{flavor}" == "gnu-openmpi-hpc"
    %bcond_without hpc
    %define compiler_family gnu
    %undefine c_f_ver
    %global mpi_family openmpi
    %define mpi_ver 1
    %endif
    ```
   Each MPI flavor (or family) will require its own conditional section.

   **NOTE**: The variable names may be chosen differently, i.e.
  `${mpi_flavor}` instead of `${mpi_family}` However, they need to be
   consistent throughout the spec file.

5. Define the variables and add the call to `hpc_init`

   The call to `hpc_init` is a must for every HPC package, this macro will
   setup all the variables and make all the variables expansion needed.

    ```
    %{?hpc_init:%hpc_init -c %compiler_family [-v %c_f_ver] [ -m %mpi_family [-V %mpi_family_version]] [-e %ext]}
    ```

   The variables used and defined, can change a lot from a package to
   another, depending on the type of software. Every package is different,
   below, there are examples that can be taken as starting point and
   adapt as needed.

   This example is the case of a package that is always built HPC packages
   with MPI support and provides at least a non-HPC version also built
   with MPI support.

    ```
    %{?mpi_flavor:%{bcond_without mpi}}%{!?mpi_flavor:%{bcond_with mpi}}
    %{?with_hpc:%{!?compiler_family:%global compiler_family gnu}}
    %{?with_mpi:%{!?mpi_flavor:%global mpi_flavor openmpi}}
    %{?with_mpi:%global pkg_suffix -%{mpi_flavor}}

    %if %{with hpc}
	 %{hpc_init -c %compiler_family %{?with_mpi:-m %mpi_flavor} %{?c_f_ver:-v %{c_f_ver}} %{?mpi_ver:-V %{mpi_ver}} %{?ext:-e %{ext}}}
	 %global hpc_module_pname %{pname}
	 %define pkg_prefix %hpc_prefix
	 %define pkg_bindir %hpc_bindir
	 %define pkg_libdir %hpc_libdir
	 %define pkg_incdir %hpc_includedir
	 %define pkg_datadir %hpc_datadir
	 %define pkg_sysconfdir %{hpc_prefix}/etc
	 %define package_name   %{hpc_package_name %_vers}
     %define libname(l:s:)   lib%{pname}%{-l*}%{hpc_package_name_tail %{?_vers}}
    %else
	 %define pkg_prefix %{_libdir}/mpi/gcc/%{mpi_flavor}
	 %define pkg_bindir %{pkg_prefix}/bin
	 %define pkg_libdir %{pkg_prefix}/%{_lib}/
	 %define pkg_incdir %{pkg_prefix}/include/
	 %define pkg_datadir %{pkg_prefix}/share/
	 %define pkg_sysconfdir %{pkg_prefix}/etc/
	 %define package_name   %pname%{?pkg_suffix}
	 %define libname(l:s:)   lib%{pname}%{!-l:%{-s:-}}%{-l*}%{-s*}%{?pkg_suffix}
	%endif
    ```

6. Create the RPM preamble section as for any regular package with the
following changes:

   a. To set the package name correctly, use:
      ```
      Name:    %{package_name}
      Version: %{vers}
      ```

   b. Add the standard build requires as needed making sure the HPC extra
      requires are there as well:
      ```
      %if %{with hpc}
      BuildRequires:  suse-hpc
      BuildRequires:  %{compiler_family}%{?c_f_ver}-compilers-hpc-macros-devel
      BuildRequires:  %{mpi_family}%{?mpi_family_version}-mpi-hpc-macros-devel
      BuildRequires:  lua-lmod
      %endif
      ```

      **NOTE**: this cannot be done by a macro and this section may not depend on any
      value set in macros.hpc.

      For special purposes in the preamble (like adding an so-version) the
      macro `%hpc_package_name_tail()` is available. This is identical to
      `%hpc_package_name()` however `%pname` is missing.
      It is debatable whether an .so-version is required as there will be an
      individual package for each minor package version.

7. Add Requires:
   a. Add the macro `%hpc_requires` to  the Requires: list of the base package
      (which may be a library package).
   b. Add the macro `%hpc_devel_requires` to the Requires: list of the devel
      package.


8. In the build and install sections, make sure the paths to the compiler and
   MPI libraries are set correctly. To do so, add:
    ```
    %hpc_setup
    ```
   before any call to configure or make. Any further module dependency settings
   should follow this macro.

9. To make sure that everything is set up correctly and aid debugging there is
   the macro:
    ```
    %hpc_debug
    ```
   This can be added to any section which creates executable shell commands:
   ie `%prep`, `%build` and `%install`

10. There are a number of macros available which may be used in arguments to
    make:
    ```
    %hpc_prefix
    %hpc_exec_prefix
    %hpc_libdir
    %hpc_includedir
    %hpc_sbindir
    %hpc_bindir
    %hpc_libexecdir
    %hpc_localstatedirhasbeen
    %hpc_sharedstatedir
    %hpc_mandir
    %hpc_infodir
    %hpc_sysconfdir
    %hpc_datadir
    %hpc_docdir
    %hpc_pkgconfigdir
    ```

    These may be used in the `%files` sections later on as well.

11. There is a macro to create a basic packageconfig file if a project doesn't
    provide one:
    ```
    %hpc_write_pkgconfig [-n <pkgconfig_name>][-l <libname>]
    ```

12. In order for your libraries to be usable it is required to create modules
    files. They should be created from within the spec file (ie build script)
    using a 'here-style' document:
    ```
	%if %{with hpc}
	%hpc_write_modules_files
	#%%Module1.0#####################################################################

	proc ModulesHelp { } {

	puts stderr " "
	puts stderr "This module loads the %{pname} library built with the %{compiler_family} compiler"
	puts stderr "toolchain and the %{mpi_flavor} MPI stack."
	puts stderr "\nVersion %{version}\n"

	}
	module-whatis "Name: %{pname} built with %{compiler_family} compiler and %{mpi_flavor} MPI"
	module-whatis "Version: %{version}"
	module-whatis "Category: runtime library"
	module-whatis "Description: %{summary}"
	module-whatis "%{url}"

	set     version                     %{version}

	depends-on phdf5

	prepend-path    PATH                %{hpc_bindir}
	prepend-path    LD_LIBRARY_PATH     %{hpc_libdir}

	setenv          %{hpc_upcase %pname}_DIR        %{hpc_prefix}
	setenv          %{hpc_upcase %pname}_BIN        %{hpc_bindir}
	setenv          %{hpc_upcase %pname}_LIB        %{hpc_libdir}

	if {[file isdirectory  %{hpc_includedir}]} {
	prepend-path    LIBRARY_PATH        %{hpc_libdir}
	prepend-path    CPATH               %{hpc_includedir}
	prepend-path    C_INCLUDE_PATH      %{hpc_includedir}
	prepend-path    CPLUS_INCLUDE_PATH  %{hpc_includedir}
	prepend-path    INCLUDE             %{hpc_includedir}

	setenv          %{hpc_upcase %pname}_INC        %{hpc_includedir}
	}

	EOF
	%endif
    ```

    The macro `%hpc_write_module_files` will create all the needed
    directories - only the content of the the main modules file will have to
    be provided. Since `%hpc_write_module_files` expects a 'here' document,
    make sure the 'body' follows in the next line and ends in a single `EOF`
    at the beginning of the line.
    There are some convenience macros to aid creating the content:
    * `%hpc_modulepath` will expand to the correct value for MODULEPATH which
      is required for compilers and mpi libraries.
    * `%hpc_modulefile_add_pkgconfig_path` If you have generated a pkgconfig
      file in 11. (or used the project provided one) you can use:
    ```
    %hpc_modulefile_add_pkgconfig_path
    ```
      to add the right PKGCONFIG_PATH clause to the module file.
      There may be macros to generate the entire module files in the future.
    `%hpc_modules_files` may be used in the `%files`-section of the package
    which is to provide the modules files to add all required files and directories.

13. If a (library) package installs a modules file, it needs to check if it
    has been chosen as the default on uninstall and remove this default.
    This can be handled by placing the macro
    ```
    %hpc_module_delete_if_default
    ```
    into an `%postun` section for this package.

14. In the `%files` section (most likely of the library package itself), you may
    add:
    ```
    %hpc_dirs
    ```
    or
    `%hpc_mpi_dirs` when building an MPI library (`%hpc_cf_dirs`
    when building a compiler) to make sure all destination directories
    are added.

15. To add the module files, add:
    ```
    %hpc_modules_files
    ```
16. The pkgconfig file may be added to the library devel package using:
    ```
    %hpc_pkgconfig_file [-n <pkgconfig_name>]
    ```
17. For any other path the macros described in 10. may be used.

18. Master packages:
    Master packages are there to install the latest version of a specific
    package without having to worry about the version number.
    In most cases, this should only be needed for the library and devel
    package. Only if the module files are in a separate package or there
    is a macro file that's needed by the build service, this should be
    required.
    There are numerous options to the macro:
    ```
    %hpc_master_package [-n <full_package_name>][-g <group>][-s <.so-version>][-l][-L][-a] <package_ext>
    ```
    The most commonly used options are -l and -L: -l builds a master package
    for the library, -L adds a %post install section to create the symlink
    of the module version file to the latest version (to make it default).
    -a marks a package architecture dependent (default: independent). The
    -l and -s flags imply -a.
    Also a package extension (like `devel`) may be specified. If there is
    no other way, the full package name can be specified as well using the
    `-n` option - this is similar to the options for `%package`, `%description`
    and `%files`.

19. `%postun` rule:
    The master package which has a -L specified sets up a link to mark the
    latest version of a package default (by setting a link in the module
    files).
    If the package which provides the module file linked to is uninstalled,
    this link should be removed. This can be accomplished by creating a
    `%postun` script for this package containing the macro:
    `%hpc_module_delete_if_default`.

# Hints:

It may be necessary to build HPC style packages along side 'standard' ones.
In this case it is advisable to set `%{bcond_without hpc}` for HPC builds
and `%{bcond_with hpc}` for 'standard' builds.

HPC builds can be identified using:

    ```
    %if %{with hpc}
    ...
    %endif
    ```
or
    ```
    %{?with_hpc:...}
    ```
While 'standard' builds may be identified using:

    ```
    %if %{without hpc}
    ...
    %endif
    ```
or
    ```
    %{!?with_hpc:...}
    ```


# Useful macros:
1. `%{hpc_upcase lower}` will make uppercase the string "lower".
2. `%{hpc_compress_man <num>}` will compress man pages in directory `%hpc_mandir/man${j}/`


# Python packages
Python packages use a sophisticated macro infrastructure to build for both
python 2.7 and 3. These macros can be found in the macro file
`macros.python_all` from the package python-rpm-macros. For the macros below
to work it is expected that this macro file is available and its
infrastructure is used.
1. `%hpc_python_sitearch`:  provides the path to the arch dependent packages in
  the HPC directory structure.
2. `%hpc_python_version`: obtain an abbreviated python version number for
  package names and directories.
3. `%{hpc_python_master_package [-n <package_name>] [-g <group>] [-l][-L][-a]}`
  * `-n`: specify a package name
  * `-g`: specify a group name
  * `-l`: mark the package as library package
  * `-L`: (for environment modules) create a link from the
    `.version.<version_no>` file  to the `.version`-file to
    mark the default version.
  * `-a`: mark package as architecture dependent
