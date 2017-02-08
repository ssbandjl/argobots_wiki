1. [`autogen.sh` Errors](#autogen)
   * ['LIBTOOL' is undefined](#libtool)


# <a name="autogen">`autogen.sh` Errors</a>
## <a name="libtool">'LIBTOOL' is undefined</a>
If you get this error when running `./autogen.sh`
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
Then, try running the following sequence:
```
$ libtoolize
$ aclocal
$ autoheader
```

