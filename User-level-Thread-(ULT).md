# Introduction
User-level Threads (ULTs) are independent execution units in user space. ULT is associated with a function and has its own stack. ULTs provide standard thread semantics at a very low context-switching cost. ULTs are yieldable and can be explicitly yielded to using their handles.

ULTs can make blocking calls, but users must be careful. If a ULT is blocked without yielding, it will block the entire ES. An efficient user library built on Argobots should design carefully to reduce the blocking time as much as possible.

Each ULT executes inside the associated ES and it is scheduled by the scheduler in the ES. ULTs in a single ES are not executed in parallel, i.e., only one ULT runs in an ES at a specific point in time. However, ULTs in different ESs may run in parallel as ESs execute concurrently.

## ULT Types
In Argobots, there are two kinds of ULTs: primary or secondary. A major difference between them is that users can only create secondary ULTs but not the primary ULT.
* **Primary ULT (initial ULT)**. Primary ULT, or initial ULT, is the ULT that has started the main function. It is started automatically by the Argobots library when the library is initialized. Its associated ES is the primary ES by default.
* **Secondary ULT**. Secondary ULTs are ULTs other than the primary ULT. Any ULT can create secondary ULTs and they can be migrated to other ESs.

ULTs can also be categorized depending on their handles: named or unnamed.
* **Named ULT**. The named ULT is a ULT whose handle is returned to user program. Handles can be used to query information about ULTs, to explicitly yield ULTs, or to migrate ULTs. When a named ULT is created, its initial reference count is one. Its reference count is internally managed to determine when it can be freed safely. When the reference count becomes zero, the memory used for the ULT object is freed.
* **Unnamed ULT**. The unnamed ULT is a ULT whose handle is never returned to user program. It cannot be explicitly migrated to other ESs because its handle is not used. The memory object for unnamed ULT is automatically freed when it terminates.

## ULT Stack Size
The primary ULT has a stack size determined by the underlying library, e.g., Pthread, used to implement ESs. On the other hand, secondary ULTs have a default stack size defined by the runtime, but users can specify a different stack size in the ULT attribute when they create a secondary ULT.

## ULT States
ULT can be in one state among READY, RUNNING, BLOCKED, and TERMINATED at a certain point in time during its execution. Each state corresponds to `ABT_THREAD_STATE_READY`, `ABT_THREAD_STATE_RUNNING`, `ABT_THREAD_STATE_BLOCKED`, and `ABT_THREAD_STATE_TERMINATED` of enum [ABT_thread_state](https://github.com/pmodels/argobots/wiki/Data-Types#abt_thread_state), respectively. The ULT's state can be obtained with [ABT_thread_get_state()](#abt_thread_get_state).

[[https://github.com/pmodels/argobots/blob/master/doc/img/ult_states.png|alt=ULT_states]]

The following describes each state of ULT:

**READY**
* READY is the initial state when a ULT is created. The ULT's state can also be READY from RUNNING or BLOCKED when it needs to be rescheduled. The ULT in the READY state waits for its scheduling turn.

**RUNNING**
* ULT moves to the RUNNING state when it is scheduled by the ES's scheduler and starts its execution.
* When the ULT yields the control, its state becomes READY.
* When the ULT makes a blocking call, it goes to the BLOCKED state.
* When the ULT finishes its execution, its state becomes TERMINATED. This includes the case when the ULT invokes the ULT exit function or the cancelation is requested.

**BLOCKED**
* In the BLOCKED state, a ULT waits until the blocking condition is removed. After that, the ULT moves to the READY state.

**TERMINATED**
* This state represents that the ULT has finished its execution.
* When there is a cancel request, a ULT becomes TERMINATED from all states. However, when the ULT is in the RUNNING state, it will be terminated after finishing its execution.
* If the ULT is named, its resource is kept in the associated ES until its reference count becomes zero. On the other hands, if the ULT is unnamed, its resource is immediately released.
