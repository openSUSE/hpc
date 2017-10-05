# HPC Dependency Generators

* `hpc_elf.attr`, `hpc_elflib.attr` - RPM dependency generator attribute files
* `hpc_elf.pl`- RPM dependency generator script
* `dlinfo.c` - Simple tool to check if library is located in the default
   search path.

Unlike the standard library dependency generators, these scripts skip 
libraries which are not located in the standard library search path. 
This avoids 'generic' library dependencies being added without additional 
information which may be satisfied by multiple library packages.

Multiple libraries may satisfy the same library dependencies but still differ
in ABI implemenation details: one example are the MPI libraries openmpi,
mvapich2 and mpich: while all provide libmpi.so.12()(64bit), they are not
ABI compatible. In HPC environments this is typically handled by environment
modules only allowing compatible libraries to be found.

`hpc_elf.pl` currrently only handles default dependencies (like the standard
dependency generator) but skips any library in a non-default search path (ie
one that requires `LD_LIBRARY_PATH` to be set up correctly).
It is planned to handle these libraries in the future adding further
information to identify additional dependency requirements.

`dlinfo.c` - build with: `gcc -o dlinfo dlinfo.c -ldl`;
 Usage: `dlinfo <library>`, will return value 0 if library is found,
 otherwise != 0.
