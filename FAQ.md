1. [`autogen.sh` shows errors](#autogen)
   * ['LIBTOOL' is undefined](#libtool)
2. [Does Argobots work with address sanitizers?](#asan)

## <a name="autogen"> 1. autogen.sh shows errors</a>
### <a name="libtool">'LIBTOOL' is undefined</a>
If you download a release tarball of Argobots, you don't need to run `./autogen.sh`.  `configure` exists.

If you clone Argobots from GitHub, you need to run `./autogen.sh` to create `configure`. If you get this error when running `./autogen.sh`
```
=================================================================
Checking autotools installations
================================================================= 
Checking for autoconf version... >= 2.67 
Checking for automake version... >= 1.12.3 
Checking for libtool version... >= 2.4
================================================================= 
Generating required temporary files 
================================================================= 
Generating a helper maint/Version... done 
Updating the README... done
================================================================= 
Generating configure and friends 
================================================================= 
autoreconf: Entering directory `.' 
autoreconf: configure.ac: not using Gettext 
autoreconf: running: aclocal --force -I m4 
autoreconf: configure.ac: tracing 
autoreconf: configure.ac: not using Libtool 
autoreconf: running: /usr/bin/autoconf --force 
autoreconf: running: /usr/bin/autoheader --force 
autoreconf: running: automake --add-missing --copy --force-missing 
configure.ac:70: installing 'm4/ar-lib' 
configure.ac:68: installing 'm4/install-sh' 
configure.ac:68: installing 'm4/missing' 
Makefile.am:12: error: Libtool library used but 'LIBTOOL' is undefined 
Makefile.am:12: The usual way to define 'LIBTOOL' is to add 'LT_INIT' 
Makefile.am:12: to 'configure.ac' and run 'aclocal' and 'autoconf' again. 
Makefile.am:12: If 'LT_INIT' is in 'configure.ac', make sure 
Makefile.am:12: its definition is in aclocal's search path. 
Makefile.am: installing 'm4/depcomp' 
autoreconf: automake failed with exit status: 1
```
First, please confirm that your system has `libtool`.

If your system has `libtool` but still shows this error, try running the following sequence:
```
$ libtoolize
$ aclocal
$ autoheader
```

## <a name="asan">2. Does Argobots work with address sanitizers?</a>

Yes. Argobots supports address sanitizers of GCC and Clang.

Argobots 1.1b1 and later versions are regularly tested with address sanitizers of relatively newer GCC and Clang (i.e., GCC 9.1 and Clang 10.0). Argobots should work with them without any special flags. Because user-level context switches used in Argobots manipulate stack pointers and instruction addresses in an unusual way, however, we cannot guarantee that all versions of address sanitizers perfectly work with Argobots.  For example, older or newer address sanitizers of GCC and Clang might warn Argobots' user-level context switches.

We would like to support as many versions of address sanitizers (especially those of GCC and Clang) as we can.  If you find any issues, please let us know.
