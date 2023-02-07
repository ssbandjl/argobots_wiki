* [Introduction](#introduction)
* [API](#api)
  * [ABT_init()](#abt_init)
  * [ABT_finalize()](#abt_finalize)
  * [ABT_initialized()](#abt_initialized)
* [Example](#example)

# Introduction
Argobots code should include the initialization function, `ABT_init()`, before using other Argobots functions and the finalization function, `ABT_finalize()`, after using all Argobots functions. If `ABT_init()` and `ABT_finalize()` are paired, they can be used more than once in a program.

Argobots 代码应在使用其他 Argobots 函数之前包含初始化函数 ABT_init()，在使用所有 Argobots 函数之后包含终结函数 ABT_finalize()。 如果 ABT_init() 和 ABT_finalize() 配对，它们可以在一个程序中多次使用。

# API
## ABT_init()
```c
int ABT_init(int argc, char **argv)
```
* Initialize the Argobots execution environment.
* Parameters
  * [in] `argc`: the number of arguments
  * [in] `argv`: the argument vector
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_init()` initializes the Argobots library and its execution environment. It internally creates objects for the *primary ES* and the *primary ULT*.
  * `ABT_init()` must be called by the primary ULT before using any other Argobots functions. `ABT_init()` can be called again after `ABT_finalize()` is called.

  ABT_init() 初始化 Argobots 库及其执行环境。 它在内部为主要 ES 和主要 ULT 创建对象。在使用任何其他 Argobots 功能之前，主 ULT 必须调用 ABT_init()。 
  ABT_init()可以在调用ABT_finalize()之后再次调用。

## ABT_finalize()
```c
int ABT_finalize(void)
```
* Terminate the Argobots execution environment.
* Return values	
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_finalize()` terminates the Argobots execution environment and deallocates memory internally used in Argobots. This function also contains deallocation of objects for the primary ES and the primary ULT.
  * `ABT_finalize()` must be called by the primary ULT. Invoking the Argobots functions after `ABT_finalize()` is not allowed. To use the Argobots functions after calling `ABT_finalize()`, `ABT_init()` needs to be called again.

  ABT_finalize() 终止 Argobots 执行环境并释放 Argobots 内部使用的内存。 此功能还包含为主要 ES 和主要 ULT 解除对象分配。
ABT_finalize() 必须由主 ULT 调用。 不允许在 ABT_finalize() 之后调用 Argobots 函数。 要在调用 ABT_finalize() 后使用 Argobots 函数，需要再次调用 ABT_init()。

## ABT_initialized()
```c
int ABT_initialized(void)
```
* Check whether `ABT_init()` has been called.
* Return values
  * If `ABT_init()` has been called, `ABT_SUCCESS` is returned.
  * If `ABT_init()` has not been called, `ABT_ERR_UNINITIALIZED` is returned.
* Details
  * `ABT_initialized()` returns `ABT_SUCCESS` if `ABT_init()` has been called. Otherwise, it returns `ABT_ERR_UNINITIALIZED`.

# Example
The simplest Argobots code can be written like the example below.
```c
#include <abt.h>

int main(int argc, char **argv)
{
    ABT_init(argc, argv);
    ABT_finalize();
    return 0;
}
```