
# How to use this fork

This is a fork from  [tomaskubar](https://github.com/tomaskubar/gromacs-dftbplus) with a couple of modifications.
This code is tested on Ubuntu Ubuntu 22.04.5 LTS with g++ 11.4.0 and is pretty dependent on the right version of requirements and is not compatible with latest versions as of today:

✅ Works with [Plumed 2.6.6](https://github.com/plumed/plumed2/releases/tag/v2.6.6)
❌ Doesn't work with Plumed 2.9.3

✅ Works with [DFTB+ 21.1](https://github.com/dftbplus/dftbplus/releases/tag/21.1)
❌ Doesn't work with DFTB+ 24.1

## Compilation
Probably there are other ways to make this code work, but the following setup has worked for me.  Before start compiling Gromacs, make sure you have compiled **right versions** of Plumed and DFTB+ on your computer (precompiled versions don't work).
### Forked Gromacs:

    git clone https://github.com/saeid-lab/gromacs-dftbplus.git
We leave it here for the moment.

### Plumed:
Extracting the source and :

    ./configure --enable-modules=all --prefix=/path/to/install --enable-shared
    make
    make install

After a successful compilation, you must make 3 symlinks of *Plumed.cmake.shared*, *Plumed.h*, and *Plumed.inc* in the root directory of cloned Gromacs code:

🛑 Please note, the code is already patched with Plumed and there is no need to run `patch -p`.

    cd gromacs-dftbplus
    
    ln -s /path/to/install/lib/plumed/src/lib/Plumed.cmake.shared Plumed.cmake 
    ln -s /path/to/install/include/plumed/wrapper/Plumed.h Plumed.h
    ln -s /path/to/install/lib/plumed/src/lib/Plumed.inc Plumed.inc 

### DFTB+:
Extracting the source:
Run: `python /utils/get_opt_externals ALL` to get dispersion correction D3.

    mkdir build && cd build
    cmake .. -DBUILD_SHARED_LIBS=OFF -DCMAKE_INSTALL_PREFIX=path/to/install -DWITH_API=ON   -DWITH_DFTD3=ON -DWITH_OMP=ON
    make
    make install

Going back to Gromacs source directory:

    mkdir build && cd build
    cmake .. -DGMX_BUILD_OWN_FFTW=ON -DCMAKE_POLICY_VERSION_MINIMUM=3.5 -DCMAKE_INSTALL_PREFIX=/path/to/install -DGMX_DOUBLE=ON -DGMX_OPENMP=ON -DGMX_PLUMED=ON -DGMX_QMMM_PROGRAM=dftbplus -DGMX_QMMM_DFTBPLUS_LIB=/dftbplus/path/build/lib  -DBUILD_TESTING=OFF -DCMAKE_EXE_LINKER_FLAGS="-L/usr/lib/x86_64-linux-gnu/openblas-pthread  -lopenblas" -DGMX_INSTALL_NBLIB_API:BOOL=OFF
    make


🔎 If you get linking error(s) like the following:

    /usr/bin/ld: /path/to/dftbplust/lib/libdftbplus.a(lapackroutines.f90.o): undefined reference to symbol 'sgesvd_'
    /usr/bin/ld: /lib/x86_64-linux-gnu/libopenblas.so.0: error adding symbols: DSO missing from command line
    collect2: error: ld returned 1 exit status
    make[2]: *** [src/programs/CMakeFiles/gmx.dir/build.make:103: bin/gmx_d] Error 1
    make[1]: *** [CMakeFiles/Makefile2:3239: src/programs/CMakeFiles/gmx.dir/all] Error 2
    make[1]: *** Waiting for unfinished jobs....

Simply:  `vim gromacs-dftbplus/build/src/programs/CMakeFiles/gmx.dir/link.txt`
and add `-lopenblas` at the end of this file.

You can try once more by `make` and `make install`

Enjoy!

> Special thanks to Tomáš Kubař for helping me make this work

---



               Welcome to the official version of GROMACS!

If you are familiar with Unix, it should be fairly trivial to compile and
install GROMACS. GROMACS uses only the CMake build sytem, and our
installation guide can be found at
http://manual.gromacs.org/documentation/current/install-guide/index.html

Of course we will do our utmost to help you with any problems, but PLEASE 
READ THE INSTALLATION INSTRUCTIONS BEFORE CONTACTING US!

There are also several other online resources available from the homepage, 
and special information for developers.

If you are a developer, or change the source for any other reason, check
out http://www.gromacs.org/Developer_Zone.

                               * * * * *

GROMACS is free software, distributed under the GNU Lesser General
Public License, version 2.1 However, scientific software is a little
special compared to most other programs. Both you, we, and all other
GROMACS users depend on the quality of the code, and when we find bugs
(every piece of software has them) it is crucial that we can correct
it and say that it was fixed in version X of the file or package
release. For the same reason, it is important that you can reproduce
other people's result from a certain GROMACS version.

The easiest way to avoid this kind of problems is to get your modifications
included in the main distribution. We'll be happy to consider any decent 
code. If it's a separate program it can probably be included in the contrib 
directory straight away (not supported by us), but for major changes in the 
main code we appreciate if you first test that it works with (and without) 
MPI, threads, double precision, etc.

If you still want to distribute a modified version or use part of GROMACS
in your own program, remember that the entire project must be licensed
according to the requirements of the LGPL v2.1 license under which you
received this copy of GROMACS. We request that it must clearly be labeled as
derived work. It should not use the name "official GROMACS", and make
sure support questions are directed to you instead of the GROMACS developers.
Sorry for the hard wording, but it is meant to protect YOUR reseach results!

                               * * * * *

The development of GROMACS is mainly funded by academic research grants. 
To help us fund development, we humbly ask that you cite the GROMACS papers:

* GROMACS: A message-passing parallel molecular dynamics implementation
  H.J.C. Berendsen, D. van der Spoel and R. van Drunen
  Comp. Phys. Comm. 91, 43-56 (1995)
  DOI: https://doi.org/10.1016/0010-4655(95)00042-E
 
* GROMACS 4: Algorithms for highly efficient, load-balanced, and scalable
  molecular simulation
  B. Hess and C. Kutzner and D. van der Spoel and E. Lindahl
  J. Chem. Theory Comput. 4 (2008) pp. 435-447
  DOI: https://doi.org/10.1021/ct700301q

* GROMACS 4.5: a high-throughput and highly parallel open source
  molecular simulation toolkit
  Sander Pronk, Szilárd Páll, Roland Schulz, Per Larsson, Pär Bjelkmar,
  Rossen Apostolov, Michael R. Shirts, Jeremy C. Smith, Peter M. Kasson,
  David van der Spoel, Berk Hess, Erik Lindahl.
  Bioinformatics 29 (2013) pp. 845-54
  DOI: https://doi.org/10.1093/bioinformatics/btt055

* Tackling Exascale Software Challenges in Molecular Dynamics Simulations
  with GROMACS
  Szilárd Páll, Mark J. Abraham, Carsten Kutzner, Berk Hess, Erik Lindahl
  In S. Markidis & E. Laure (Eds.), Solving Software Challenges for Exascale,
  Lecture Notes for Computer Science, 8759 (2015) pp. 3–27
  DOI: https://doi.org/10.1007/978-3-319-15976-8_1

* GROMACS: High performance molecular simulations through multi-level parallelism from laptops to supercomputers
  M. J. Abraham, T. Murtola, R. Schulz, S. Páll, J. C. Smith, B. Hess, E. Lindahl,
  SoftwareX, 1, (2015), 19-25
  DOI: https://doi.org/10.1016/j.softx.2015.06.001

There are a lot of cool features we'd like to include in future versions,
but our resources are limited. All kinds of donations are welcome, both in 
form of code, hardware and funding! Industrial users who choose to pay
for a license pro bono (it is still LGPL and can be redistributed freely) or
contribute in other ways are listed as GROMACS supporters on our webpages. 
Don't hesitate to contact us if you are interested.


                       Good luck with your simulations!

                              The GROMACS Crew
