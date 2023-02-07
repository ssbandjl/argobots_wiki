* [Introduction](#introduction)
  * [ES Types](#es-types)
  * [ES States](#es-states)
* [ES Execution](#es-execution)
  * [ABT_xstream_create()](#abt_xstream_create)
  * [ABT_xstream_create_basic()](#abt_xstream_create_basic)
  * [ABT_xstream_create_with_rank()](#abt_xstream_create_with_rank)
  * [ABT_xstream_start()](#abt_xstream_start)
  * [ABT_xstream_free()](#abt_xstream_free)
  * [ABT_xstream_join()](#abt_xstream_join)
  * [ABT_xstream_exit()](#abt_xstream_exit)
  * [ABT_xstream_cancel()](#abt_xstream_cancel)
  * [ABT_xstream_self()](#abt_xstream_self)
  * [ABT_xstream_self_rank()](#abt_xstream_self_rank)
  * [ABT_xstream_set_rank()](#abt_xstream_set_rank)
  * [ABT_xstream_get_rank()](#abt_xstream_get_rank)
  * [ABT_xstream_set_main_sched()](#abt_xstream_set_main_sched)
  * [ABT_xstream_set_main_sched_basic()](#abt_xstream_set_main_sched_basic)
  * [ABT_xstream_get_main_sched()](#abt_xstream_get_main_sched)
  * [ABT_xstream_get_main_pools()](#abt_xstream_get_main_pools)
  * [ABT_xstream_get_state()](#abt_xstream_get_state)
  * [ABT_xstream_run_unit()](#abt_xstream_run_unit)
  * [ABT_xstream_check_events()](#abt_xstream_check_events)
  * [ABT_xstream_set_cpubind()](#abt_xstream_set_cpubind)
  * [ABT_xstream_get_cpubind()](#abt_xstream_get_cpubind)
  * [ABT_xstream_set_affinity()](#abt_xstream_set_affinity)
  * [ABT_xstream_get_affinity()](#abt_xstream_get_affinity)
* [ES Information](#es-information)
  * [ABT_xstream_equal()](#abt_xstream_equal)
  * [ABT_xstream_get_num()](#abt_xstream_get_num)
  * [ABT_xstream_is_primary()](#abt_xstream_is_primary)
* [Work Unit Migration](#work-unit-migration)
  * [ABT_xstream_migrate_threads()](#abt_xstream_migrate_threads)
  * [ABT_xstream_migrate_threads_to_pool()](#abt_xstream_migrate_threads_to_pool)
  * [ABT_xstream_migrate_tasks()](#abt_xstream_migrate_tasks)
  * [ABT_xstream_migrate_tasks_to_pool()](#abt_xstream_migrate_tasks_to_pool)
  * [ABT_xstream_migrate_units()](#abt_xstream_migrate_units)
  * [ABT_xstream_migrate_units_to_pool()](#abt_xstream_migrate_units_to_pool)

# Introduction
Execution Stream (ES) is the execution of a sequential instruction stream. Each instruction does not necessarily need to be SISD, but can also be vector or SIMD instructions. When bound to a hardware processing element (PE), ES can be thought as its software equivalent. Argobots considers one ES per PE. Oversubscription of ESs is possible but is not recommended.

执行流（ES）是顺序指令流的执行。 每条指令不一定是SISD，也可以是vector或SIMD指令。 当绑定到硬件处理元素 (PE) 时，ES 可以被认为是它的软件等价物。 Argobots 考虑每个 PE 一个 ES。 可以超额订阅 ES，但不建议这样做。

ES is conceptually a container of work units, which are User-level Threads (ULTs) or tasklets. Work units execute to completion, but their execution may be interleaved inside an ES. Since ES is a sequential instruction stream, there is no parallel execution of work units in a single ES. Only one work unit runs in an ES at a certain point. However, work units in different ESs can be executed in parallel. ES has implicitly-managed progress semantics handled by kernel or hardware, and thus one blocked ES cannot block other ESs.

ES 在概念上是工作单元的容器，工作单元是用户级线程 (ULT) 或微线程。 工作单元执行到完成，但它们的执行可能在 ES 中交错。 由于 ES 是顺序指令流，因此在单个 ES 中不存在工作单元的并行执行。 在某一时刻，只有一个工作单元在一个 ES 中运行。 但是，不同 ES 中的工作单元可以并行执行。 ES 具有由内核或硬件处理的隐式管理的进度语义，因此一个阻塞的 ES 不能阻塞其他 ES。

Each ES executes all associated work units according to the scheduling policy. The scheduler associated with the ES is in charge of scheduling. ES has its own scheduler and different ESs can adopt different schedulers. The Argobots runtime provides some predefined schedulers, but users can also create their own schedulers.

每个 ES 根据调度策略执行所有关联的工作单元。 与ES关联的调度器负责调度。 ES有自己的调度器，不同的ES可以采用不同的调度器。 Argobots 运行时提供了一些预定义的调度器，但用户也可以创建自己的调度器。

## ES Types
Argobots categorizes ESs into two types: primary and secondary. A major difference between them is that users can only create secondary ESs but not the primary ES.
Argobots 将 ES 分为两种类型：初级和次级。 它们之间的一个主要区别是用户只能创建辅助 ES 而不能创建主 ES。
* **Primary ES (initial ES)**. This is the ES that automatically starts when the Argobots library is initialized. It cannot be created explicitly, and an application has only one primary ES. When the primary ES starts, it implicitly contains one ULT, which has started the main function. In other words, when the Argobots is initialized, there exists one primary ES containing one ULT. Primary ES establishes parent/child relationship with secondary ESs. When it terminates, all secondary ESs are terminated.
初级 ES（初始 ES）。 这是Argobots库初始化时自动启动的ES。 不能显式创建，一个应用只有一个主 ES。 当主 ES 启动时，它隐式包含一个 ULT，它已经启动了 main 函数。 换句话说，当 Argobots 被初始化时，存在一个包含一个 ULT 的主 ES。 主要 ES 与次要 ES 建立父/子关系。 当它终止时，所有辅助 ES 都将终止。
* **Secondary ES**. Secondary ESs are ESs other than the primary ES. They are explicitly created at run time, and any work unit can create them. There is no parent/child relationship between secondary ESs. Termination of a secondary ES does not affect execution of other ESs including both primary ES and secondary ESs.
二级 ES。 辅助 ES 是主要 ES 之外的 ES。 它们是在运行时显式创建的，任何工作单元都可以创建它们。 辅助 ES 之间没有父/子关系。 终止辅助 ES 不会影响其他 ES 的执行，包括主要 ES 和辅助 ES。

## ES States
ES stays in one state at a certain point in time during its execution. Possible states of ES are CREATED, READY, RUNNING, and TERMINATED, and they correspond to `ABT_XSTREAM_STATE_CREATED`, `ABT_XSTREAM_STATE_READY`, `ABT_XSTREAM_STATE_RUNNING`, and `ABT_XSTREAM_STATE_TERMINATED` of `enum ABT_xstream_state`, respectively. The ES's state can be obtained with `ABT_xstream_get_state()`.

ES在执行过程中的某个时间点处于一种状态。 ES 的可能状态有 CREATED、READY、RUNNING 和 TERMINATED，它们分别对应于枚举 ABT_xstream_state 的 ABT_XSTREAM_STATE_CREATED、ABT_XSTREAM_STATE_READY、ABT_XSTREAM_STATE_RUNNING 和 ABT_XSTREAM_STATE_TERMINATED。 ES 的状态可以通过 ABT_xstream_get_state() 获得。

[[https://github.com/pmodels/argobots/blob/master/doc/img/es_states.png|alt=ES_states]]

The following describes each state of ES:

**CREATED**
* Initial state when an ES is created.

**READY**
* ES's state becomes READY when the ES starts its execution. The Argobots implementation may start ES immediately when it is created or delay until any work unit gets associated with the ES or the user explicitly starts the ES's execution. Once ES enters the READY state, it waits for creation of work units in the READY state.

当 ES 开始执行时，ES 的状态变为 READY。 Argobots 实现可以在 ES 创建时立即启动，也可以延迟到任何工作单元与 ES 关联或用户明确启动 ES 的执行。 一旦 ES 进入 READY 状态，它会等待创建处于 READY 状态的工作单元。

**RUNNING**
* When there exist work units to execute, ES moves to the RUNNING state, and it keeps the RUNNING state while there remain work units. If there is no more work unit to execute, the ES changes its state to READY again.

当存在要执行的工作单元时，ES 进入 RUNNING 状态，并在有剩余工作单元时保持 RUNNING 状态。 如果没有更多的工作单元要执行，则 ES 再次将其状态更改为 READY。

**TERMINATED**
* When there is a join request, ES's state becomes TERMINATED. In the CREATED state, it is immediately terminated. However, in other states, it will be terminated after finishing execution of all associated work units. It processes the join request only when it is in READY state.
* When a work unit invokes the ES exit function, its associated ES is terminated irrespective of the existence of work units and the ES's state becomes TERMINATED. The user has to migrate remaining work units associated with the terminating ES, if necessary.
* When there is a cancel request, ES's state becomes TERMINATED from all states. When a work unit is running, i.e., the ES is in RUNNING state, the ES will be terminated after the associated scheduler gets a chance to execute. Like the ES exit function, the user may need to migrate remaining work units associated with the terminating ES, if necessary.

当有加入请求时，ES 的状态变为 TERMINATED。 在 CREATED 状态下，它立即终止。 但是，在其他状态下，它将在所有相关工作单元执行完毕后终止。 它仅在处于 READY 状态时才处理加入请求。
当工作单元调用 ES 退出函数时，无论工作单元是否存在，其关联的 ES 都会终止，并且 ES 的状态变为 TERMINATED。 如有必要，用户必须迁移与终止 ES 关联的剩余工作单元。
当有取消请求时，ES的状态从所有状态变为TERMINATED。 当工作单元正在运行时，即 ES 处于 RUNNING 状态，ES 将在关联的调度程序有机会执行后终止。 与 ES 退出功能一样，如果需要，用户可能需要迁移与终止 ES 关联的剩余工作单元。

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
  等待 ES 终止。 如果目标 ES 已经终止，则此函数立即返回
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

## ABT_xstream_set_main_sched()
```c
int ABT_xstream_set_main_sched(ABT_xstream xstream, ABT_sched sched)
```
* Set the main scheduler of the target ES.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [out] `sched`: handle to the scheduler
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_set_main_sched()` sets `sched` as the main scheduler for `xstream`. The scheduler `sched` will first run when the ES `xstream` is started.  Only ULTs are allowed to call this function.
  * If `xstream` is a handle to the primary ES, `sched` will be automatically freed on `ABT_finalize()` or when the main scheduler of the primary ES is changed again.  In this case, the explicit call `ABT_sched_free()` for `sched` may cause undefined behavior.

## ABT_xstream_set_main_sched_basic()
```c
int ABT_xstream_set_main_sched_basic(ABT_xstream xstream,
        ABT_sched_predef predef, int num_pools, ABT_pool *pools)
```
* Set the main scheduler for the target ES with a predefined scheduler.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [in] `predef`: predefined scheduler
  * [in] `num_pools`: number of pools
  * [in] `pools`: pools to be associated with the scheduler
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_set_main_sched_basic()` internally creates a scheduler of the `predef` type, which uses the array of pools `pools` whose number of elements is `num_pools`, and sets the created scheduler as the main scheduler for `xstream`. The scheduler will first run when the ES `xstream` is started.  Only ULTs are allowed to call this function.
  * The scheduler created in this function will be automatically freed when the associated ES is freed.

## ABT_xstream_get_main_sched()
```c
int ABT_xstream_get_main_sched(ABT_xstream xstream, ABT_sched *sched)
```
* Get the main scheduler of the target ES.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [out] `sched`: handle to the scheduler
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_get_main_sched()` returns the handle of the main scheduler for the target ES `xstream` through `sched`.

## ABT_xstream_get_main_pools()
```c
int ABT_xstream_get_main_pools(ABT_xstream xstream, int max_pools, ABT_pool *pools)
```
* Get the pools of the main scheduler of the target ES.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [in] `max_pools`: maximum number of pools
  * [out] `pools`: array of handles to the pools
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_get_main_pools()` returns at most `max_pools` pools of the main scheduler associated with the ES `xstream`.  Basically, this function is a convenient function that calls `ABT_xstream_get_main_sched()` to get the main scheduler, and then `ABT_sched_get_pools()` to retrieve the associated pools.

## ABT_xstream_get_state()
```c
int ABT_xstream_get_state(ABT_xstream xstream, ABT_xstream_state *state)
```
* Return the state of the target ES.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [out] `state`: ES state
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_get_state()` returns the state of target ES `xstream`. It returns one of the [ABT_xstream_state](https://github.com/pmodels/argobots/wiki/Data-Types#abt_xstream_state) values.

## ABT_xstream_run_unit()
```c
int ABT_xstream_run_unit(ABT_unit unit, ABT_pool pool)
```
* Execute a unit on the local ES.
* Parameters
  * [in] `unit`: handle to the unit to run
  * [out] `pool`: pool where the unit came from
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_run_unit()` executes the work unit `unit`, which was popped from `pool`, on the local ES. The local ES means the ES associated with the caller of this function. This function is intended to be called by a scheduler after picking up one unit. Thus, this function is expected to be used when the users write their own scheduler.

## ABT_xstream_check_events()
```c
int ABT_xstream_check_events(ABT_sched sched)
```
* Check events on the local ES and process them.
* Parameter
  * [in] `sched`: handle to the scheduler
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_check_events()` checks events associated with the scheduler `sched` on the local ES, i.e., the ES associated with the caller of this function, and the events are handled in this function if they have occurred. This function is intended to be called by a scheduler periodically, thus it is expected that the users call this function in their own scheduler definition.

  ABT_xstream_check_events() 检查与本地 ES 上调度程序关联的事件，即与该函数的调用者关联的 ES，如果事件发生，则在该函数中处理。 此函数旨在由调度程序定期调用，因此希望用户在他们自己的调度程序定义中调用此函数。

## ABT_xstream_set_cpubind()
```c
int ABT_xstream_set_cpubind(ABT_xstream xstream, int cpuid)
```
* Bind the target ES to a target CPU.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [in] `cpuid`: CPU ID
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_set_cpubind()` binds the target ES `xstream` to the target CPU whose ID is `cpuid`.  Here, the CPU ID corresponds to the processor index used by OS.

## ABT_xstream_get_cpubind()
```c
int ABT_xstream_get_cpubind(ABT_xstream xstream, int *cpuid)
```
* Get the CPU binding for the target ES.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [out] `cpuid`: CPU ID
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_get_cpubind()` returns the ID of CPU, which the target ES `xstream` is bound to. If `xstream` is bound to more than one CPU, only the first CPU ID is returned.

## ABT_xstream_set_affinity()
```c
int ABT_xstream_set_affinity(ABT_xstream xstream, int cpuset_size, int *cpuset)
```
* Set the CPU affinity of the target ES.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [in] `cpuset_size`: the number of `cpuset` entries
  * [in] `cpuset`: array of CPU IDs
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_set_cpubind()` binds the target ES `xstream` on the given CPU set, `cpuset`, which is an array of CPU IDs.  Here, the CPU IDs correspond to processor indexes used by OS.

## ABT_xstream_get_affinity()
```c
int ABT_xstream_get_affinity(ABT_xstream xstream, int cpuset_size, int *cpuset,
                             int *num_cpus)
```
* Get the CPU affinity for the target ES.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [in] `cpuset_size`: the number of \c cpuset entries
  * [out] `cpuset`: array of CPU IDs
  * [out] `num_cpus`: the number of total CPU IDs
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_get_cpubind()` writes CPU IDs (at most, `cpuset_size`) to `cpuset` and returns the number of elements written to `cpuset` to `num_cpus`.  If `num_cpus` is `NULL`, it is ignored.
  * If `cpuset` is `NULL`, `cpuset_size` is ignored and the nubmer of all CPUs on which `xstream` is bound is returned through `num_cpus`. Otherwise, i.e., if `cpuset` is `NULL`, `cpuset_size` must be greater than zero.

# ES Information
## ABT_xstream_equal()
```c
int ABT_xstream_equal(ABT_xstream xstream1, ABT_xstream xstream2, ABT_bool *result)
```
* Compare two ES handles for equality.
* Parameters
  * [in] `xstream1`: handle to the ES 1
  * [in] `xstream2`: handle to the ES 2
  * [out] `handle to the ES 2`: comparison result (`ABT_TRUE`: same, `ABT_FALSE`: not same)
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_equal()` compares two ES handles for equality. If two handles are associated with the same ES, `result` will be set to `ABT_TRUE`. Otherwise, `result` will be set to `ABT_FALSE`.

## ABT_xstream_get_num()
```c
int ABT_xstream_get_num(int *num_xstreams)
```
* Return the number of current existing ESs.
* Parameter
  * [out] `num_xstreams`: the number of ESs
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_get_num()` returns the number of ESs that exist in the current Argobots environment through `num_xstreams`.

## ABT_xstream_is_primary()
```c
int ABT_xstream_is_primary(ABT_xstream xstream, ABT_bool *flag)
```
* Check if the target ES is the primary ES.
* Parameters
  * [in] `xstream`: handle to the target ES
  * [out] `flag`: result (`ABT_TRUE`: primary ES, `ABT_FALSE`: not)
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_is_primary()` checks whether the target ES is the primary ES. If the ES `xstream` is the primary ES, `flag` is set to `ABT_TRUE`. Otherwise, `flag` is set to `ABT_FALSE`.

# Work Unit Migration
## ABT_xstream_migrate_threads()
```c
int ABT_xstream_migrate_threads(ABT_xstream source, ABT_xstream target, int n, ABT_bool blocking,
        void (*cb_func)(ABT_thread *threads, int n, void *cb_arg), void *cb_arg, int cb_place)
```
* Migrate a certain number of ULTs associated with the source ES to the target ES.
* Parameters
  * [in] `source`: handle to the source ES
  * [in] `target`: handle to the target ES
  * [in] `n`: the number of ULTs to be migrated
  * [in] `blocking`: Boolean value to indicate a blocking migration (`ABT_TRUE`) or a non-blocking migration (`ABT_FALSE`)
  * [in] `cb_func`: callback function pointer
  * [in] `cb_arg`: argument for the callback function
  * [in] `cb_place`: bit-field to specify the location where the callback function is executed
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_migrate_threads()` migrates at most `n` ULTs from the source ES specified by `source` to the target ES specified by `target`. The ULTs will be added to proper pools of scheduler associated with the target ES, and they will be running on the target ES after migration. The actual number of migrated ULTs may be smaller than `n` depending on the number of migratable ULTs on the source ES. Users can get the actual number of migrated ULTs through the second argument of callback function.
  * If `target` is `ABT_XSTREAM_NULL`, the target ES will be determined by the runtime. However, `source` cannot be `ABT_XSTREAM_NULL`.
  * If `blocking` is `ABT_TRUE`, the migration operation is blocking, and the routine will return after migration. Otherwise, i.e., if `blocking` is `ABT_FALSE`, the migration operation is non-blocking. The routine requests the migration and returns immediately regardless of completion of migration.
  * If a callback function, `cb_func`, is set (not `NULL`), it will be invoked with `cb_arg` when the migration happens. The runtime will provide the first argument `threads`, which is an array of ULTs migrated, and the second argument `n`, which is the number of ULTs actually migrated.
  * `cb_place` is a bit-field to specify the location where the callback function is executed. If no callback function is set, `cb_place` is ignored. The following list describes the possible values of `cb_place`. If `cb_place` is 0, `ABT_CB_ON_TARGET` is used as the default value.
    * `ABT_CB_ON_TARGET`: the callback function will be executed by the scheduler of target ES. 
    * `ABT_CB_ON_SOURCE`: the callback function will be executed by the scheduler of source ES. `ABT_CB_ON_SOURCE` can be used with `ABT_CB_ON_TARGET` like `(ABT_CB_ON_TARGET | ABT_CB_ON_SOURCE)`, to execute the callback function on both sides.

## ABT_xstream_migrate_threads_to_pool()
```c
int ABT_xstream_migrate_threads_to_pool(ABT_xstream source, ABT_pool target, int n, ABT_bool blocking,
        void (*cb_func)(ABT_thread *threads, int n, void *cb_arg), void *cb_arg, int cb_place)
```
* Migrate a certain number of ULTs associated with the source ES to the target pool.
* Parameters
  * [in] `source`: handle to the source ES
  * [in] `target`: handle to the target pool
  * [in] `n`: the number of ULTs to be migrated
  * [in] `blocking`: Boolean value to indicate a blocking migration (`ABT_TRUE`) or a non-blocking migration (`ABT_FALSE`)
  * [in] `cb_func`: callback function pointer
  * [in] `cb_arg`: argument for the callback function
  * [in] `cb_place`: bit-field to specify the location where the callback function is executed
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_migrate_threads_to_pool()` migrates at most `n` ULTs from the source ES specified by `source` to the target pool specified by `target`. The actual number of migrated ULTs may be smaller than `n` depending on the number of migratable ULTs on the source ES. Users can get the actual number of migrated ULTs through the second argument of callback function.
  * If `target` is `ABT_POOL_NULL`, the target pool will be determined by the runtime. However, source cannot be `ABT_XSTREAM_NULL`.
  * If `blocking` is `ABT_TRUE`, the migration operation is blocking, and the routine will return after migration. Otherwise, i.e., if `blocking` is `ABT_FALSE`, the migration operation is non-blocking. The routine requests the migration and returns immediately regardless of completion of migration.
  * If a callback function, `cb_func`, is set (not `NULL`), it will be invoked with `cb_arg` when the migration happens. The runtime will provide the first argument `threads`, which is an array of ULTs migrated, and the second argument `n`, which is the number of ULTs actually migrated.
  * `cb_place` is a bit-field to specify the location where the callback function is executed. If no callback function is set, `cb_place` is ignored. The following list describes the possible values of `cb_place`. If `cb_place` is 0, `ABT_CB_ON_TARGET` is used as the default value.
    * `ABT_CB_ON_TARGET`: the callback function will be executed by the scheduler associated with target pool.
    * `ABT_CB_ON_SOURCE`: the callback function will be executed by the scheduler of source ES. `ABT_CB_ON_SOURCE` can be used with `ABT_CB_ON_TARGET` like `(ABT_CB_ON_TARGET | ABT_CB_ON_SOURCE)`, to execute the callback function on both sides.

## ABT_xstream_migrate_tasks()
```c
int ABT_xstream_migrate_tasks(ABT_xstream source, ABT_xstream target, int n, ABT_bool blocking,
        void (*cb_func)(ABT_task *tasks, int n, void *cb_arg), void *cb_arg, int cb_place)
```
* Migrate a certain number of tasklets associated with the source ES to the target ES.
* Parameters
  * [in] `source`: handle to the source ES
  * [in] `target`: handle to the target ES
  * [in] `n`: the number of tasklets to be migrated
  * [in] `blocking`: Boolean value to indicate a blocking migration (`ABT_TRUE`) or a non-blocking migration (`ABT_FALSE`)
  * [in] `cb_func`: callback function pointer
  * [in] `cb_arg`: argument for the callback function
  * [in] `cb_place`: bit-field to specify the location where the callback function is executed
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_migrate_tasks()` migrates at most `n` tasklets from the source ES specified by `source` to the target ES specified by `target`. The tasklets will be added to proper pools of scheduler associated with the target ES, and they will be running on the target ES after migration. The actual number of migrated tasklets may be smaller than `n` depending on the number of migratable tasklets on the source ES. Users can get the actual number of migrated tasklets through the second argument of callback function.
  * If `target` is `ABT_XSTREAM_NULL`, the target ES will be determined by the runtime. However, `source` cannot be `ABT_XSTREAM_NULL`.
  * If `blocking` is `ABT_TRUE`, the migration operation is blocking, and the routine will return after migration. Otherwise, i.e., if `blocking` is `ABT_FALSE`, the migration operation is non-blocking. The routine requests the migration and returns immediately regardless of completion of migration.
  * If a callback function, `cb_func`, is set (not `NULL`), it will be invoked with `cb_arg` when the migration happens. The runtime will provide the first argument `tasks`, which is an array of tasklets migrated, and the second argument `n`, which is the number of tasklets actually migrated.
  * `cb_place` is a bit-field to specify the location where the callback function is executed. If no callback function is set, `cb_place` is ignored. The following list describes the possible values of `cb_place`. If `cb_place` is 0, `ABT_CB_ON_TARGET` is used as the default value.
    * `ABT_CB_ON_TARGET`: the callback function will be executed by the scheduler of target ES. 
    * `ABT_CB_ON_SOURCE`: the callback function will be executed by the scheduler of source ES. `ABT_CB_ON_SOURCE` can be used with `ABT_CB_ON_TARGET` like `(ABT_CB_ON_TARGET | ABT_CB_ON_SOURCE)`, to execute the callback function on both sides.

## ABT_xstream_migrate_tasks_to_pool()
```c
int ABT_xstream_migrate_tasks_to_pool(ABT_xstream source, ABT_pool target, int n, ABT_bool blocking,
        void (*cb_func)(ABT_task *tasks, int n, void *cb_arg), void *cb_arg, int cb_place)
```
* Migrate a certain number of tasklets associated with the source ES to the target pool.
* Parameters
  * [in] `source`: handle to the source ES
  * [in] `target`: handle to the target pool
  * [in] `n`: the number of tasklets to be migrated
  * [in] `blocking`: Boolean value to indicate a blocking migration (`ABT_TRUE`) or a non-blocking migration (`ABT_FALSE`)
  * [in] `cb_func`: callback function pointer
  * [in] `cb_arg`: argument for the callback function
  * [in] `cb_place`: bit-field to specify the location where the callback function is executed
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_migrate_tasks_to_pool()` migrates at most `n` tasklets from the source ES specified by `source` to the target pool specified by `target`. The target pool can be any type of pool including global pools. The actual number of migrated tasklets may be smaller than `n` depending on the number of migratable tasklets on the source ES. Users can get the actual number of migrated tasklets through the second argument of callback function.
  * If `target` is `ABT_POOL_NULL`, the target pool will be determined by the runtime. However, `source` cannot be `ABT_XSTREAM_NULL`.
  * If `blocking` is `ABT_TRUE`, the migration operation is blocking, and the routine will return after migration. Otherwise, i.e., if `blocking` is `ABT_FALSE`, the migration operation is non-blocking. The routine requests the migration and returns immediately regardless of completion of migration.
  * If a callback function, `cb_func`, is set (not `NULL`), it will be invoked with `cb_arg` when the migration happens. The runtime will provide the first argument `tasks`, which is an array of tasklets migrated, and the second argument `n`, which is the number of tasklets actually migrated.
  * `cb_place` is a bit-field to specify the location where the callback function is executed. If no callback function is set, `cb_place` is ignored. The following list describes the possible values of `cb_place`. If `cb_place` is 0, `ABT_CB_ON_TARGET` is used as the default value.
    * `ABT_CB_ON_TARGET`: the callback function will be executed by the scheduler associated with target pool. 
    * `ABT_CB_ON_SOURCE`: the callback function will be executed by the scheduler of source ES. `ABT_CB_ON_SOURCE` can be used with `ABT_CB_ON_TARGET` like `(ABT_CB_ON_TARGET | ABT_CB_ON_SOURCE)`, to execute the callback function on both sides.

## ABT_xstream_migrate_units()
```c
int ABT_xstream_migrate_units(ABT_xstream source, ABT_xstream target, int n, ABT_bool blocking,
        void (*cb_func)(ABT_unit *units, int n, void *cb_arg), void *cb_arg, int cb_place)
```
* Migrate a certain number of work units associated with the source ES to the target ES.
* Parameters
  * [in] `source`: handle to the source ES
  * [in] `target`: handle to the target ES
  * [in] `n`: the number of work units to be migrated
  * [in] `blocking`: Boolean value to indicate a blocking migration (`ABT_TRUE`) or a non-blocking migration (`ABT_FALSE`)
  * [in] `cb_func`: callback function pointer
  * [in] `cb_arg`: argument for the callback function
  * [in] `cb_place`: bit-field to specify the location where the callback function is executed
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_migrate_units()` migrates at most `n` work units from the source ES specified by `source` to the target ES specified by `target`. The work units will be added to proper pools of scheduler associated with the target ES, and they will be running on the target ES after migration. The actual number of migrated work units may be smaller than `n` depending on the number of migratable work units on the source ES. Users can get the actual number of migrated work units through the second argument of callback function.
  * If `target` is `ABT_XSTREAM_NULL`, the target ES will be determined by the runtime. However, `source` cannot be `ABT_XSTREAM_NULL`.
  * If `blocking` is `ABT_TRUE`, the migration operation is blocking, and the routine will return after migration. Otherwise, i.e., if `blocking` is `ABT_FALSE`, the migration operation is non-blocking. The routine requests the migration and returns immediately regardless of completion of migration.
  * If a callback function, `cb_func`, is set (not `NULL`), it will be invoked with `cb_arg` when the migration happens. The runtime will provide the first argument `units`, which is an array of work units migrated, and the second argument `n`, which is the number of work units actually migrated.
  * `cb_place` is a bit-field to specify the location where the callback function is executed. If no callback function is set, `cb_place` is ignored. The following list describes the possible values of `cb_place`. If `cb_place` is 0, `ABT_CB_ON_TARGET` is used as the default value.
    * `ABT_CB_ON_TARGET`: the callback function will be executed by the scheduler of target ES. 
    * `ABT_CB_ON_SOURCE`: the callback function will be executed by the scheduler of source ES. `ABT_CB_ON_SOURCE` can be used with `ABT_CB_ON_TARGET` like `(ABT_CB_ON_TARGET | ABT_CB_ON_SOURCE)`, to execute the callback function on both sides.

## ABT_xstream_migrate_units_to_pool()
```c
int ABT_xstream_migrate_units_to_pool(ABT_xstream source, ABT_pool target, int n, ABT_bool blocking,
        void (*cb_func)(ABT_unit *units, int n, void *cb_arg), void *cb_arg, int cb_place)
```
* Migrate a certain number of work units associated with the source ES to the target pool.
* Parameters
  * [in] `source`: handle to the source ES
  * [in] `target`: handle to the target pool
  * [in] `n`: the number of work units to be migrated
  * [in] `blocking`: Boolean value to indicate a blocking migration (`ABT_TRUE`) or a non-blocking migration (`ABT_FALSE`)
  * [in] `cb_func`: callback function pointer
  * [in] `cb_arg`: argument for the callback function
  * [in] `cb_place`: bit-field to specify the location where the callback function is executed
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_xstream_migrate_units_to_pool()` migrates at most `n` work units from the source ES specified by `source` to the target pool specified by `target`. The actual number of migrated work units may be smaller than `n` depending on the number of migratable work units on the source ES. Users can get the actual number of migrated work units through the second argument of callback function.
  * If `target` is `ABT_POOL_NULL`, the target pool will be determined by the runtime. However, `source` cannot be `ABT_XSTREAM_NULL`.
  * If `blocking` is `ABT_TRUE`, the migration operation is blocking, and the routine will return after migration. Otherwise, i.e., if `blocking` is `ABT_FALSE`, the migration operation is non-blocking. The routine requests the migration and returns immediately regardless of completion of migration.
  * If a callback function, `cb_func`, is set (not `NULL`), it will be invoked with `cb_arg` when the migration happens. The runtime will provide the first argument `units`, which is an array of work units migrated, and the second argument `n`, which is the number of work units actually migrated.
  * `cb_place` is a bit-field to specify the location where the callback function is executed. If no callback function is set, `cb_place` is ignored. The following list describes the possible values of `cb_place`. If `cb_place` is 0, `ABT_CB_ON_TARGET` is used as the default value.
    * `ABT_CB_ON_TARGET`: the callback function will be executed by the scheduler associated with target pool. 
    * `ABT_CB_ON_SOURCE`: the callback function will be executed by the scheduler of source ES. `ABT_CB_ON_SOURCE` can be used with `ABT_CB_ON_TARGET` like `(ABT_CB_ON_TARGET | ABT_CB_ON_SOURCE)`, to execute the callback function on both sides.
