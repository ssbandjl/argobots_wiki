# Introduction
Execution Stream (ES) is the execution of a sequential instruction stream. Each instruction does not necessarily need to be SISD, but can also be vector or SIMD instructions. When bound to a hardware processing element (PE), ES can be thought as its software equivalent. Argobots considers one ES per PE. Oversubscription of ESs is possible but is not recommended.

ES is conceptually a container of work units, which are User-level Threads (ULTs) or tasklets. Work units execute to completion, but their execution may be interleaved inside an ES. Since ES is a sequential instruction stream, there is no parallel execution of work units in a single ES. Only one work unit runs in an ES at a certain point. However, work units in different ESs can be executed in parallel. ES has implicitly-managed progress semantics handled by kernel or hardware, and thus one blocked ES cannot block other ESs.

Each ES executes all associated work units according to the scheduling policy. The scheduler associated with the ES is in charge of scheduling. ES has its own scheduler and different ESs can adopt different schedulers. The Argobots runtime provides some predefined schedulers, but users can also create their own schedulers.

## ES Types
Argobots categorizes ESs into two types: primary and secondary. A major difference between them is that users can only create secondary ESs but not the primary ES.
* **Primary ES (initial ES)**. This is the ES that automatically starts when the Argobots library is initialized. It cannot be created explicitly, and an application has only one primary ES. When the primary ES starts, it implicitly contains one ULT, which has started the main function. In other words, when the Argobots is initialized, there exists one primary ES containing one ULT. Primary ES establishes parent/child relationship with secondary ESs. When it terminates, all secondary ESs are terminated.
* **Secondary ES**. Secondary ESs are ESs other than the primary ES. They are explicitly created at run time, and any work unit can create them. There is no parent/child relationship between secondary ESs. Termination of a secondary ES does not affect execution of other ESs including both primary ES and secondary ESs.

## ES States
ES stays in one state at a certain point in time during its execution. Possible states of ES are CREATED, READY, RUNNING, and TERMINATED, and they correspond to `ABT_XSTREAM_STATE_CREATED`, `ABT_XSTREAM_STATE_READY`, `ABT_XSTREAM_STATE_RUNNING`, and `ABT_XSTREAM_STATE_TERMINATED` of `enum ABT_xstream_state`, respectively. The ES's state can be obtained with `ABT_xstream_get_state()`.

[[https://github.com/pmodels/argobots/blob/master/doc/img/es_states.png|alt=ES_states]]

The following describes each state of ES:

**CREATED**
* Initial state when an ES is created.

**READY**
* ES's state becomes READY when the ES starts its execution. The Argobots implementation may start ES immediately when it is created or delay until any work unit gets associated with the ES or the user explicitly starts the ES's execution. Once ES enters the READY state, it waits for creation of work units in the READY state.

**RUNNING**
* When there exist work units to execute, ES moves to the RUNNING state, and it keeps the RUNNING state while there remain work units. If there is no more work unit to execute, the ES changes its state to READY again.

**TERMINATED**
* When there is a join request, ES's state becomes TERMINATED. In the CREATED state, it is immediately terminated. However, in other states, it will be terminated after finishing execution of all associated work units. It processes the join request only when it is in READY state.
* When a work unit invokes the ES exit function, its associated ES is terminated irrespective of the existence of work units and the ES's state becomes TERMINATED. The user has to migrate remaining work units associated with the terminating ES, if necessary.
* When there is a cancel request, ES's state becomes TERMINATED from all states. When a work unit is running, i.e., the ES is in RUNNING state, the ES will be terminated after the associated scheduler gets a chance to execute. Like the ES exit function, the user may need to migrate remaining work units associated with the terminating ES, if necessary.

# ES Execution
## ABT_xstream_create()
```c
int ABT_xstream_create(ABT_sched sched, ABT_xstream *newxstream)
```
* Create a new ES.
* Parameters
  * [in] `sched`: handle to the scheduler used for a new ES
  * [out] `newxstream`: handle to a newly created ES
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_create()` creates a new ES with a specific scheduler passed as `sched` and returns its handle through `newxstream`.  If `sched` is `ABT_SCHED_NULL`, the runtime-provided scheduler is used for the new ES.

## ABT_xstream_create_basic()
```c
int ABT_xstream_create_basic(ABT_sched_predef predef, int num_pools, ABT_pool *pools,
                             ABT_sched_config config, ABT_xstream *newxstream)
```
* Create a new ES with a predefined scheduler.
* Parameters
  * [in] `predef`: predefined scheduler (see [here](https://github.com/pmodels/argobots/wiki/Data-Types#abt_sched_predef) for details)
  * [in] `num_pools`: number of pools
  * [in] `pools`: array of pools to be associated with the scheduler
  * [in] `config`: config used during the scheduler creation
  * [out] `newxstream`: handle to a newly created ES
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_create_basic()` creates a new ES with a system-provided scheduler specified as `predef` and returns its handle through `newxstream`.  If `predef` is a scheduler that includes automatic creation of pools, `pools` can be `NULL`.

## ABT_xstream_create_with_rank()
```c
int ABT_xstream_create_with_rank(ABT_sched sched, int rank, ABT_xstream *newxstream)
```
* Create a new ES with a specific rank.
* Parameters
  * [in] `sched`: handle to the scheduler used for a new ES
  * [in] `rank`: target rank
  * [out] `newxstream`: handle to a newly created ES
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * `ABT_ERR_INV_XSTREAM_RANK`, if `rank` is invalid.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_create_with_rank()` creates a new ES with a scheduler `sched` and a specific rank `rank` and returns its handle through `newxstream`. If `sched` is `ABT_SCHED_NULL`, the runtime-provided scheduler is used for the new ES.

## ABT_xstream_start()
```c
int ABT_xstream_start(ABT_xstream xstream)
```
* Start the target ES.
* Parameters
  * [in] `xstream`: handle to the target ES
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_start()` starts the target ES `xstream` if it has not been started. That is, this routine is effective only when the state of the target ES is CREATED or READY, and once this routine returns, the ES's state becomes RUNNING.

## ABT_xstream_free()
```c
int ABT_xstream_free(ABT_xstream *xstream)
```
* Free the ES object.
* Parameters
  * [in,out] `xstream`: handle to the target ES
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_free()` deallocates the memory used for the ES object associated with the handle `xstream`. If it successfully returns, `xstream` is set to `ABT_XSTREAM_NULL`.
  * If the ES of `xstream` is still running when this routine is called, the deallocation happens after the ES terminates and then this routine returns. This means that the caller can be blocked until this routine returns.
  * The scheduler object is not freed by this routine if it was created and provided by a user. It should be freed by `ABT_sched_free()`. If the user did not specify a scheduler for the target ES, the memory for default scheduler is automatically freed by this routine.
  * This function must not be used to free the primary ES. The object for primary ES will be freed when `ABT_finalize()` is called. And, a work unit associated with the target ES cannot call this function.

## ABT_xstream_join()
```c
int ABT_xstream_join(ABT_xstream xstream)
```
* Wait for the ES to terminate.
* Parameters
  * [in] `xstream`: handle to the target ES
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * The caller of `ABT_xstream_join()` waits for the ES to terminate. If the target ES has already terminated, this function returns immediately.
  * The target ES cannot be the same as the ES associated with the calling work unit. And the primary ES cannot be joined. If one of these conditions is violated, this routine will return the error code, `ABT_ERR_INV_XSTREAM`.

## ABT_xstream_exit()
```c
int ABT_xstream_exit(void)
```
* Terminate the ES.
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * The ULT calling `ABT_xstream_exit()` forces the associated ES to terminate regardless of other remaining work units in the ES. Work units that are not terminated yet will need to be migrated to different ESs or pools.
  * Since the ES associated with calling ULT finishes its execution, this function never returns to the caller if the ES terminates successfully. Tasklets are not allowed to call this function.

## ABT_xstream_cancel()
```c
int ABT_xstream_cancel(ABT_xstream xstream)
```
* Request the cancelation of the target ES.
* Parameters
  * [in] `xstream`: handle to the target ES
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_cancel()` requests the cancellation of the target ES and immediately returns to the caller. The result of the cancellation can be checked later by `ABT_xstream_get_state()`. Only secondary ESs can be canceled by this function.
  * Cancelation may be deferred if a work unit is running on the ES. The actual cancelation will take place when the associated scheduler gets a chance to execute. Other remaining work units in the ES will not be executed and will need to be migrated to different ESs or pools.

## ABT_xstream_self()
```c
int ABT_xstream_self(ABT_xstream *xstream)
```
* Return the ES handle associated with the caller work unit.
* Parameter
  * [out] `xstream`: ES handle
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_self()` returns the handle to ES object associated with the caller work unit through `xstream`. When an error occurs, `xstream` is set to `ABT_XSTREAM_NULL`.

## ABT_xstream_self_rank()
```c
int ABT_xstream_self_rank(int *rank)
```
* Return the rank of ES associated with the caller work unit.
* Parameter
  * [out] `rank `: ES rank
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_self_rank()` returns the rank of ES associated with the caller work unit.

## ABT_xstream_set_rank()
```c
int ABT_xstream_set_rank(ABT_xstream xstream, const int rank)
```
* Set the rank for target ES.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [in] `rank`: ES rank
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_set_rank()` sets the rank for target ES specified by `xstream`.

## ABT_xstream_get_rank()
```c
int ABT_xstream_get_rank(ABT_xstream xstream, int *rank)
```
* Return the rank of target ES.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [out] `rank`: ES rank
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_get_rank()` returns the rank of target ES through `rank`.
