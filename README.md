# SUSE HPC Packaging Auxiliary Files and Documentation

Directories:

* `general/` Macros and scripts for HPC library packages
  * `macros.hpc` Scripts to simplify packaging libraries according to
    the HPC packaging standard.
  * `hpc_elf.attr`, `hpc_elflib.attr`, `hpc_elf.pl`, `dlinfo.c` RPM dependency
     generator helpers  - check
    [here](general/README.dependency-generators.md) for further details.
* `MPI/` Macros for different MPI-flavors for MPI-dependent packages
         (referenced from `macros.suse-hpc`)
* `compiler/` Macros for compilers (referenced from `macros.suse-hpc`)
* `Documentation/` Documentation on
   * [Packaging Guidelines](Documentation/HPC-Packaging-Standard.md)
   * [a step-by-step description](Documentation/HPC-Packaging.md) how 
     to package HPC libraries or convert packages to the HPC packaging 
     standard
