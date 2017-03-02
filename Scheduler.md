# Introduction
A scheduler controls the execution order of scheduling units in the ES. Each scheduler is associated with an ES. A scheduler can be assigned to an ES at creation time using [ABT_xstream_create()](https://github.com/pmodels/argobots/wiki/Execution-Stream-(ES)#abt_xstream_create) or after creation using [ABT_xstream_set_main_sched()](https://github.com/pmodels/argobots/wiki/Execution-Stream-(ES)#abt_xstream_set_main_sched). A scheduler is associated with a single ES, and thus each ES can use a different scheduler. The scheduling units of a scheduler include work units (such as ULTs or tasklets) and other schedulers.

Argobots provides predefined schedulers, but users can also define their own schedulers. Users create a new scheduler by defining the pool data structure and scheduling functions for the new scheduler. In addition, users can change the scheduler for the ES during run time. This functionality enables users to plug in custom scheduling strategies for different algorithms or applications. In addition, allowing the scheduler to handle schedulers as scheduling units makes it possible to stack another scheduler on top of a scheduler.

Scheduling units are scheduled by context-switching to each other. When a work unit completes its execution or a ULT yields, it context-switches to the scheduler. The scheduler then continues its scheduling routine to choose another scheduling unit to execute. However, if a ULT explicitly yields the control to a specific ULT, the context switches from the calling ULT to the target ULT directly without going through the scheduler. That is, it can remove the scheduler from ULT's execution path.

## Scheduler Types
In Argobots, users can create two kinds of schedulers: basic and user.

**Predefined scheduler**. Predefined schedulers are schedulers provided by the Argobots runtime. Argobots currently supports three kinds of basic schedulers: BASIC, PRIO, and RANDWS (refer to enum [ABT_sched_predef](https://github.com/pmodels/argobots/wiki/Data-Types#abt_sched_predef). Users can create a predefined scheduler with a scheduler creation API and can use it when creating an ES. However, if an ES is created with a null scheduler, the default one (BASIC) is used.

**User-defined scheduler**. This is a scheduler created by users. It uses the custom pool data structure to contain scheduling units and custom scheduling functions.
