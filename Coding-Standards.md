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
If you want to write comments for doxygen documentation, refer to Code Documentation.

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
