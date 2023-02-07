# Introduction
A scheduler controls the execution order of scheduling units in the ES. Each scheduler is associated with an ES. A scheduler can be assigned to an ES at creation time using [ABT_xstream_create()](https://github.com/pmodels/argobots/wiki/Execution-Stream-(ES)#abt_xstream_create) or after creation using [ABT_xstream_set_main_sched()](https://github.com/pmodels/argobots/wiki/Execution-Stream-(ES)#abt_xstream_set_main_sched). A scheduler is associated with a single ES, and thus each ES can use a different scheduler. The scheduling units of a scheduler include work units (such as ULTs or tasklets) and other schedulers.

调度器控制 ES 中调度单元的执行顺序。 每个调度器都与一个 ES 相关联。 可以在创建时使用 ABT_xstream_create() 或在创建后使用 ABT_xstream_set_main_sched() 将调度程序分配给 ES。 调度器与单个 ES 关联，因此每个 ES 可以使用不同的调度器。 调度器的调度单元包括工作单元（例如 ULT 或 tasklet）和其他调度器。

Argobots provides predefined schedulers, but users can also define their own schedulers. Users create a new scheduler by defining the pool data structure and scheduling functions for the new scheduler. In addition, users can change the scheduler for the ES during run time. This functionality enables users to plug in custom scheduling strategies for different algorithms or applications. In addition, allowing the scheduler to handle schedulers as scheduling units makes it possible to stack another scheduler on top of a scheduler.

Argobots 提供预定义的调度器，但用户也可以定义自己的调度器。 用户通过为新的调度器定义池数据结构和调度函数来创建新的调度器。 此外，用户可以在运行时更改 ES 的调度程序。 此功能使用户能够为不同的算法或应用程序插入自定义调度策略。 此外，允许调度器将调度器作为调度单元来处理，这使得在一个调度器之上堆叠另一个调度器成为可能。

Scheduling units are scheduled by context-switching to each other. When a work unit completes its execution or a ULT yields, it context-switches to the scheduler. The scheduler then continues its scheduling routine to choose another scheduling unit to execute. However, if a ULT explicitly yields the control to a specific ULT, the context switches from the calling ULT to the target ULT directly without going through the scheduler. That is, it can remove the scheduler from ULT's execution path.

调度单元通过相互上下文切换来调度。 当工作单元完成执行或 ULT 产生时，它会上下文切换到调度程序。 调度程序然后继续其调度例程以选择另一个调度单元来执行。 但是，如果 ULT 显式地将控制权交给特定的 ULT，则上下文会从调用 ULT 直接切换到目标 ULT，而无需通过调度程序。 也就是说，它可以从 ULT 的执行路径中删除调度程序。

## Scheduler Types
In Argobots, users can create two kinds of schedulers: basic and user.  在 Argobots 中，用户可以创建两种调度程序：基本和用户。

**Predefined scheduler**. Predefined schedulers are schedulers provided by the Argobots runtime. Argobots currently supports three kinds of basic schedulers: BASIC, PRIO, and RANDWS (refer to enum [ABT_sched_predef](https://github.com/pmodels/argobots/wiki/Data-Types#abt_sched_predef). Users can create a predefined scheduler with a scheduler creation API and can use it when creating an ES. However, if an ES is created with a null scheduler, the default one (BASIC) is used.

预定义的调度程序。 预定义调度器是 Argobots 运行时提供的调度器。 Argobots 目前支持三种基本调度器：BASIC、PRIO 和 RANDWS（参考 enum ABT_sched_predef。用户可以使用调度器创建 API 创建预定义的调度器，并可以在创建 ES 时使用它。但是，如果创建的 ES 是使用 null 调度程序，使用默认调度程序 (BASIC)。

**User-defined scheduler**. This is a scheduler created by users. It uses the custom pool data structure to contain scheduling units and custom scheduling functions.

用户定义的调度程序。 这是用户创建的调度程序。 它使用自定义池数据结构来包含调度单元和自定义调度函数。