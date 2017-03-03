This group is for timer.

# Wall Clock Time
## ABT_get_wtime()
```c
double ABT_get_wtime(void)
```
* Get elapsed wall clock time.
* Return value
  * Elapsed wall clock time in seconds.
* Details
  * `ABT_get_wtime()` returns the elapsed wall clock time in seconds since an arbitrary time in the past. The resolution of elapsed time is at least a unit of microsecond.

# Timer
## ABT_timer_create()
```c
int ABT_timer_create(ABT_timer *newtimer)
```
* Create a new timer.
* Parameter
  * [out] `newtimer`: handle to a new timer
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_timer_create()` creates a new timer object and returns its handle through `newtimer`. If an error occurs in this function, a non-zero error code will be returned and `newtimer` will be set to `ABT_TIMER_NULL`.

## ABT_timer_dup()
```c
int ABT_timer_dup(ABT_timer timer, ABT_timer *newtimer)
```
* Duplicate the timer.
* Parameters
  * [in] `timer`: handle to the timer to be duplicated
  * [out] `newtimer`: handle to a new timer
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_timer_dup()` creates a new timer and copies the time values from the timer of `timer` to the new timer. The handle of new timer will be returned through `newtimer`. If an error occurs in this function, a non-zero error code will be returned and `newtimer` will be set to `ABT_TIMER_NULL`.

## ABT_timer_free()
```c
int ABT_timer_free(ABT_timer *timer)
```
* Free the timer object.
* Parameters
  * [in,out] `timer`: handle to the timer
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_timer_free()` deallocates the memory used for the timer object associated with the handle `timer`. If it is successfully processed, `timer` is set to `ABT_TIMER_NULL`. Using the timer handle after calling `ABT_timer_free()` may cause undefined behavior.

## ABT_timer_start()
```c
int ABT_timer_start(ABT_timer timer)
```
* Start the timer.
* Parameters
  * [in] `timer`: handle to the timer
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_timer_start()` starts the timer and saves the time when this function is called. When this function is called multiple times, the time of last call is only kept.

## ABT_timer_stop()
```c
int ABT_timer_stop(ABT_timer timer)
```
* Stop the timer.
* Parameters
  * [in] `timer`: handle to the timer
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_timer_stop()` stops the timer and saves the time when this function is called. When this function is called multiple times, the time of last call is only kept.

## ABT_timer_read()
```c
int ABT_timer_read(ABT_timer timer, double *secs)
```
* Read the elapsed time of the timer.
* Parameters
  * [in] `timer`: handle to the timer
  * [out] `secs`: elapsed time in seconds
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_timer_read()` returns the time difference in seconds between the start time of `timer` (when [ABT_timer_start()](#abt_timer_start) was called) and the end time of `timer` (when [ABT_timer_stop()](#abt_timer_stop) was called) through `secs`. The resolution of elapsed time is at least a unit of microsecond.

## ABT_timer_stop_and_read()
```c
int ABT_timer_stop_and_read(ABT_timer timer, double *secs)
```
* Stop the timer and read the elapsed time of the timer.
* Parameters
  * [in] `timer`: handle to the timer
  * [out] `secs`: elapsed time in seconds
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_timer_stop_and_read()` stops the timer and returns the time difference in seconds between the start time of `timer` (when [ABT_timer_start()](#abt_timer_start) was called) and the end time of `timer` (when this function was called) through `secs`. The resolution of elapsed time is at least a unit of microsecond.

## ABT_timer_stop_and_add()
```c
int ABT_timer_stop_and_add(ABT_timer timer, double *secs)
```
* Stop the timer and add the elapsed time of the timer.
* Parameters
  * [in] `timer`: handle to the timer
  * [in,out] `secs`: accumulated elapsed time in seconds
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_timer_stop_and_add()` stops the timer and adds the time difference between the start time of `timer` (when [ABT_timer_start()](#abt_timer_start) was called) and the end time of `timer` (when this function was called) to `secs`. That is, the elapsed time of the timer is accumulated in `secs`. The resolution of elapsed time is at least a unit of microsecond.

## ABT_timer_get_overhead()
```c
int ABT_timer_get_overhead(double *overhead)
```
* Obtain the overhead time of using ABT_timer.
* Parameter
  * [out] `overhead`: overhead time of ABT_timer
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_timer_get_overhead()` returns the overhead time when measuring the elapsed time with `ABT_timer`. It computes the time difference in consecutive calls of [ABT_timer_start()](#abt_timer_start) and [ABT_timer_stop()](#abt_timer_stop). The resolution of overhead time is at least a unit of microsecond.