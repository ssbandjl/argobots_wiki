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
