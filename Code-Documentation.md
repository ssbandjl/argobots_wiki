# Documenting the code
Below is an example of code documentation:
```c
/** @defgroup ULT User-level Thread (ULT) 
* This group is for User-level Thread (ULT). 
*/ 

/** 
 * @ingroup ULT 
 * @brief   Create a new ULT and return its handle through \c newthread. 
 * 
 * \c ABT_thread_create creates a new ULT that is pushed into \c pool. The 
 * insertion is done from the ES where this call is made. Therefore, the access 
 * type of \c pool should comply with that. Only a \a secondary ULT can be 
 * created explicitly, and the \a primary ULT is created automatically. 
 * 
 * If \c newthread is NULL, the ULT object will be automatically released when 
 * this \a unnamed ULT completes the execution of \c thread_func. Otherwise, 
 * \c ABT_thread_free() can be used to explicitly release the ULT object. 
 * 
 * @param[in]  pool handle to the associated pool 
 * @param[in]  thread_func  function to be executed by a new ULT 
 * @param[in]  arg          argument for \c thread_func 
 * @param[in]  arrt         ULT attribute. If it is \c ABT_THREAD_ATTR_NULL, 
 *                          the default attribute is used. 
 * @param[out] newthread    handle to a newly created thread 
 * @return Error code 
 * @retval ABT_SUCCESS on success 
 */ 
int ABT_thread_create(ABT_pool pool, void(*thread_func)(void *), 
                      void *arg, ABT_thread_attr attr, 
                      ABT_thread *newthread) 
{ 
    /* .... */ 
}
```
You can find commands for doxygen in [here](http://www.stack.nl/~dimitri/doxygen/manual/commands.html). Although you can add a brief description in the header file, it is recommended to put detailed code documentation in the source file.

# Generating doxygen documents
Doxygen documents are not automatically generated when Argobots is build. When you need doxygen documents, use the following in the top-level directory:
```
$ make doxygen
```
Generated documents will be located in `doc/doxygen`.

# Current snapshot
The following link shows the doxygen html generated from the current git repository.
* <http://www.mcs.anl.gov/~sseo/public/argobots/>