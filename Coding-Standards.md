# General Principles
Coding standards are intended to make this project more successful and enjoyable. Some general guidelines are the following:
* Use the [K&R style](https://en.wikipedia.org/wiki/Indent_style#K.26R_style)
* Use four spaces instead of using tab when you indent the code

# Basic Source File Structure
All source files follow similar structure; the file `maint/template.c` gives an example.
```c
/* -*- Mode: C; c-basic-offset:4 ; indent-tabs-mode:nil ; -*- */
/*
 * See COPYRIGHT in top-level directory.
 */ 
#include "abti.h"

int ABTI_template(ABTI_thread *p_thread, void *p_arg)
{
    int abt_errno = ABT_SUCCESS;

    if (p_thread == NULL) {
        HANDLE_ERROR("NULL THREAD");
        abt_errno = ABT_ERR_INV_THREAD;
        goto fn_fail;
    }

    /* Implementation */
    /* ... */

  fn_exit:
    return abt_errno;

  fn_fail:
    /* Error handling */
    goto fn_exit;
}
```

## Style header
This sets the style for the file and sets a common indentation amount for C and C++ programs. In addition, the style header is important for C++ header files so that the style checker will know that the file is C++ instead of C.
```c
/* -*- Mode: C; c-basic-offset:4 ; indent-tabs-mode:nil ; -*- */
```

## Copyright
`COPYRIGHT` file exists in top-level directory. If the default copyright is applied to the source file, the following can be used:
```c
/*
 * See COPYRIGHT in top-level directory.
 */
```

## Header files
Header files should have their contents wrapped within an `ifndef` of the form
```c
#ifndef FILENAME_H_INCLUDED
#define FILENAME_H_INCLUDED
...
#endif /* FILENAME_H_INCLUDED */
```
where `FILENAME` is the name of the file, in upper case.

## Column width
Lines within files should be limited to 80 columns.

## Comments
Use `/* */` and avoid `//` for writing comments. It allows compilers not supporting C99 to compile the source file with no problem on comments.
If you want to write comments for doxygen documentation, refer to [Code Documentation](https://github.com/pmodels/argobots/wiki/Code-Documentation).

## Code cleanup script
The script `maint/code-cleanup.sh` can be used to cleanup whitespace, comments, and line-wrapping in existing source files. It is a good idea to run it on any newly created files in the git repo.

# Naming Convention
Naming is important to distinguish internal implementation from public interface. Argobots uses prefix rules to differentiate codes according to their intention, i.e., public or internal.
All characters in names except the argobots prefix should be lower-case. For example,
```c
int ABT_thread_create()
```
After `ABT_`, characters are all in lower case.

## Type and routine names
Names for types and routines should be chosen in a way that (1) is descriptive of their function and (2) is clearly distinct from any name that may be used by another runtime library or user code. To achieve this, we have chosen to reserve a few prefixes for the use of the Argobots implementation:

| Prefix   | Description |
| -------- | ----------- |
| `ABT_`   | This is used for public interface that is exposed to users. All public data types and APIs should start with this prefix. |
| `ABTI_`  | This is used for internal data types and routines to implement the Argobots public APIs. This does not include device-specific types and routines. |
| `ABTD_`  | This is used for device-specific data types and routines to implement `ABT_` and `ABTI_` routines. It plays a role of an interface to hide internal device-specific implementation. |
| `ABTDI_` | This is used for internal implementation for device-specific types and routines. It implements `ABTD_` types and routines according to the target architecture. The configure script should correctly choose an appropriate implementation for the target architecture. |
| `ABTU_`  | This is used for utility routines, which support general functions not only limited in implementation of Argobots. Therefore, utility routines should be able to be compiled independently. |

## Macro and enum names
Just like type and routine names, it is important to pick macro and enum names to avoid possible conflicts with others defined in system header files.

| Prefix/Suffix | Description |
| ------------- | ----------- |
| `ABT_`        | Used for public interface. |
| `ABTI_`, `ABTD_`, `ABTDI_`, `ABTU_` | Used for internal implementation. Each prefix has same meaning as one in type and routine names.  |
| `DEBUG_`      | Used for debugging. |
| `_H_INCLUDED` | This suffix should be used as the test for including a header (`.h`) file. |

Every macro and enum name starting with above prefixes should be all uppercase.

## Variables
Some prefixes are used to distinguish global variables and pointers. For regular variables, currently there is no strict rule for naming them.

| Prefix | Description |
| ------ | ----------- |
| `g_`   | Used for global variables |
| `gp_`  | Used for global pointers |
| `l_`   | Used for ES local variables, which are shared only in an ES |
| `lp_`  | Used for ES local pointers |
| `p_`   | Used for pointer variables |
| `f_`   | Used for function pointers |

# Error Handling
TBD

# Memory Allocation
To manage dynamically-allocated memory in Argobots, we use a special set of memory allocation macros that are treated as utility routines. The macro names and their corresponding functions are the following:

| Macro Name     | Corresponding Function |
| -------------- | ---------------------- |
| `ABTU_malloc`  | `malloc`  |
| `ABTU_calloc`  | `calloc`  |
| `ABTU_free`    | `free`    |
| `ABTU_realloc` | `realloc` |

The definitions are defined in `src/abtu.h`. We are planning to exploit these macros to test and diagnose memory corruption and leak problems when they occur.
