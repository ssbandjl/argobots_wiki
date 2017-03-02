# Introduction
Tasklet is an indivisible unit of work, with dependence only on its input data, and typically providing output data upon completion. Tasklet is associated with a function but it does not have its own stack. It is executed using the runtime's stack. It is not allowed to make blocking calls and does not explicitly yield control.

A tasklet is usually created with no association with ESs, but it is executed on a specific ES. When the scheduler of ES tries to schedule a work unit, it can take a tasklet that has no associated ES if there exists, and execute it. On the other hand, a tasklet can be associated with a specific ES when it is created. In that case, it is scheduled according to the scheduling policy of the ES’s scheduler. The tasklet may be delayed to run due to the execution of other work units in the ES.

## Tasklet Types
Tasklets in Argobots are classified according to their handle: named or unnamed.
* **Named tasklet**. The named tasklet is a tasklet whose handle is returned to user program. Handles can be used to query information about tasklets or to control the execution of tasklets. When a named tasklet is created, its initial reference count is one. Its reference count is internally managed to determine when it can be freed safely. When the reference count becomes zero, the memory used for the tasklet object is freed.
* **Unnamed tasklet**. The unnamed tasklet is a tasklet whose handle is never returned to user program. The memory object for unnamed tasklet is automatically freed when it terminates.

## Tasklet Stack Size
Tasklets do not have their own stack frame. The scheduler provides a tasklet with the stack when it is run on the ES. Because the scheduler has a stack size as large as the primary ULT has, users can regard tasklet’s stack size as large enough to execute the tasklet.

## Tasklet States
Tasklet can be in one state among READY, RUNNING, and TERMINATED at a certain point in time during its execution. Each state corresponds to `ABT_TASK_STATE_READY`, `ABT_TASK_STATE_RUNNING`, and `ABT_TASK_STATE_TERMINATED` of enum [ABT_task_state](https://github.com/pmodels/argobots/wiki/Data-Types#abt_task_state), respectively. The tasklet's state can be obtained with [ABT_task_get_state()](#abt_task_get_state).

[[https://github.com/pmodels/argobots/blob/master/doc/img/tasklet_states.png|alt=Tasklet_states]]

The following describes each state of tasklet:

**READY**
* Initial state when a tasklet is created.

**RUNNING**
* The tasklet's state becomes RUNNING when it starts its execution on a specific ES.

**TERMINATED**
* This state represents that the tasklet has finished its execution.
* When there is a cancel request, a tasklet becomes TERMINATED from all states. However, when the tasklet is in the RUNNING state, it will be terminated after finishing its execution.
* If the tasklet is named, its resource is kept in the associated ES until its reference count becomes zero. On the other hands, if the tasklet is unnamed, its resource is immediately released.

# Tasket Execution
