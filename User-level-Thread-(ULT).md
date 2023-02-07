# Introduction
User-level Threads (ULTs) are independent execution units in user space. ULT is associated with a function and has its own stack. ULTs provide standard thread semantics at a very low context-switching cost. ULTs are yieldable and can be explicitly yielded to using their handles.

用户级线程（ULT）是用户空间中的独立执行单元。 ULT 与一个函数相关联，并有自己的堆栈。 ULT 以非常低的上下文切换成本提供标准线程语义。 ULT 是可屈服的，并且可以显式屈服以使用它们的句柄。

ULTs can make blocking calls, but users must be careful. If a ULT is blocked without yielding, it will block the entire ES. An efficient user library built on Argobots should design carefully to reduce the blocking time as much as possible.

ULT 可以进行阻塞调用，但用户必须小心。 如果一个 ULT 在没有让步的情况下被阻塞，它将阻塞整个 ES。 一个基于 Argobots 的高效用户库应该仔细设计，尽可能减少阻塞时间。

Each ULT executes inside the associated ES and it is scheduled by the scheduler in the ES. ULTs in a single ES are not executed in parallel, i.e., only one ULT runs in an ES at a specific point in time. However, ULTs in different ESs may run in parallel as ESs execute concurrently.

每个 ULT 在关联的 ES 内部执行，并由 ES 中的调度程序调度。 单个 ES 中的 ULT 不是并行执行的，即在特定时间点，只有一个 ULT 在 ES 中运行。 然而，不同 ES 中的 ULT 可能会在 ES 并发执行时并行运行。

## ULT Types
In Argobots, there are two kinds of ULTs: primary or secondary. A major difference between them is that users can only create secondary ULTs but not the primary ULT.

在 Argobots 中，有两种 ULT：主要或次要。 它们之间的一个主要区别是用户只能创建辅助 ULT，而不能创建主 ULT。

* **Primary ULT (initial ULT)**. Primary ULT, or initial ULT, is the ULT that has started the main function. It is started automatically by the Argobots library when the library is initialized. Its associated ES is the primary ES by default. 主要 ULT（初始 ULT）。 主 ULT，或初始 ULT，是已启动 main 功能的 ULT。 它在库初始化时由 Argobots 库自动启动。 其关联的 ES 默认为主 ES。

* **Secondary ULT**. Secondary ULTs are ULTs other than the primary ULT. Any ULT can create secondary ULTs and they can be migrated to other ESs. 二级 ULT。 次要 ULT 是主要 ULT 以外的 ULT。 任何 ULT 都可以创建辅助 ULT，并且它们可以迁移到其他 ES。

ULTs can also be categorized depending on their handles: named or unnamed. ULT 也可以根据其句柄进行分类：命名或未命名。
* **Named ULT**. The named ULT is a ULT whose handle is returned to user program. Handles can be used to query information about ULTs, to explicitly yield ULTs, or to migrate ULTs. When a named ULT is created, its initial reference count is one. Its reference count is internally managed to determine when it can be freed safely. When the reference count becomes zero, the memory used for the ULT object is freed.  命名为 ULT。 命名的 ULT 是一个 ULT，它的句柄返回给用户程序。 句柄可用于查询有关 ULT 的信息、显式生成 ULT 或迁移 ULT。 创建命名 ULT 时，其初始引用计数为 1。 它的引用计数在内部进行管理，以确定何时可以安全地释放它。 当引用计数变为零时，用于 ULT 对象的内存将被释放。
* **Unnamed ULT**. The unnamed ULT is a ULT whose handle is never returned to user program. It cannot be explicitly migrated to other ESs because its handle is not used. The memory object for unnamed ULT is automatically freed when it terminates.  未命名的 ULT。 未命名的 ULT 是其句柄永远不会返回给用户程序的 ULT。 它不能显式迁移到其他 ES，因为它的句柄未被使用。 未命名 ULT 的内存对象在它终止时自动释放。

## ULT Stack Size  ULT堆栈大小
The primary ULT has a stack size determined by the underlying library, e.g., Pthread, used to implement ESs. On the other hand, secondary ULTs have a default stack size defined by the runtime, but users can specify a different stack size in the ULT attribute when they create a secondary ULT. 主要 ULT 的堆栈大小由用于实现 ES 的底层库决定，例如 Pthread。 另一方面，辅助 ULT 具有由运行时定义的默认堆栈大小，但用户可以在创建辅助 ULT 时在 ULT 属性中指定不同的堆栈大小。

## ULT States
ULT can be in one state among READY, RUNNING, BLOCKED, and TERMINATED at a certain point in time during its execution. Each state corresponds to `ABT_THREAD_STATE_READY`, `ABT_THREAD_STATE_RUNNING`, `ABT_THREAD_STATE_BLOCKED`, and `ABT_THREAD_STATE_TERMINATED` of enum [ABT_thread_state](https://github.com/pmodels/argobots/wiki/Data-Types#abt_thread_state), respectively. The ULT's state can be obtained with [ABT_thread_get_state()](#abt_thread_get_state). 

ULT在执行过程中的某个时间点可以处于READY、RUNNING、BLOCKED、TERMINATED中的一种状态。 每个状态分别对应枚举 ABT_thread_state 的 ABT_THREAD_STATE_READY、ABT_THREAD_STATE_RUNNING、ABT_THREAD_STATE_BLOCKED 和 ABT_THREAD_STATE_TERMINATED。 ULT 的状态可以通过 ABT_thread_get_state() 获得。

[[https://github.com/pmodels/argobots/blob/master/doc/img/ult_states.png|alt=ULT_states]]

The following describes each state of ULT:

**READY**
* READY is the initial state when a ULT is created. The ULT's state can also be READY from RUNNING or BLOCKED when it needs to be rescheduled. The ULT in the READY state waits for its scheduling turn.  READY 是创建 ULT 时的初始状态。 当需要重新安排时，ULT 的状态也可以从 RUNNING 变为 READY 或 BLOCKED。 处于 READY 状态的 ULT 等待轮到它的调度。

**RUNNING**
* ULT moves to the RUNNING state when it is scheduled by the ES's scheduler and starts its execution.
* When the ULT yields the control, its state becomes READY.
* When the ULT makes a blocking call, it goes to the BLOCKED state.
* When the ULT finishes its execution, its state becomes TERMINATED. This includes the case when the ULT invokes the ULT exit function or the cancelation is requested.

**BLOCKED**
* In the BLOCKED state, a ULT waits until the blocking condition is removed. After that, the ULT moves to the READY state.

在 BLOCKED 状态下，ULT 等待直到阻塞条件被移除。 之后，ULT 进入 READY 状态。

**TERMINATED**
* This state represents that the ULT has finished its execution.
* When there is a cancel request, a ULT becomes TERMINATED from all states. However, when the ULT is in the RUNNING state, it will be terminated after finishing its execution.
* If the ULT is named, its resource is kept in the associated ES until its reference count becomes zero. On the other hands, if the ULT is unnamed, its resource is immediately released.  如果 ULT 被命名，它的资源将保存在关联的 ES 中，直到它的引用计数变为零。 另一方面，如果 ULT 未命名，其资源将立即释放。
