This chapter describes Argobots terms, conventions, and data types used throughout this document.

* [Glossary](#glossary)
* [Conventions](#conventions)
  * [Naming Convention](#naming-convention)
  * [API Convention](#api-convention)
* [Data Types](#data-types)
  * [Object Types](#object-types)
  * [Scalar Types](#scalar-types)
  * [Other Types](#other-types)
* [Null Objects](#null-objects)
* [Constants](#constants)
* [Error Classes](#error-classes)

# Glossary
* **Condition Variable**: a condition on which ULTs are waiting until it is signaled.
* **Execution Stream (ES)**: a sequential instruction stream that contains one or more work units.
* **Future**: a mechanism for passing a value between work units, allowing a work unit to wait for a value that is set asynchronously.
* **Handle**: an opaque reference to an Argobots object.
* **Mutex**: a synchronization method to support mutual exclusion between work units.
* **Object**: abstract representation of resources that are manipulated by the Argobots API. For example, there are ES objects, ULT objects, tasklet objects, etc.
* **Pool**: a data structure that contains scheduling units.
* **Scheduler**: an object that controls execution order of scheduling units in the ES. A scheduler is associated with an ES.
* **Scheduling Unit (SU)**: an object handled by a scheduler with a scheduling policy. It includes work units, such as ULTs or tasklets, and schedulers.
* **Tasklet**: an indivisible unit of work, with dependence only on its input data, and typically providing output data upon completion. Tasklet is associated with a function but it does not have its own stack. It is not allowed to make blocking calls and does not explicitly yield control.
* **User-level Thread (ULT)**: an independent execution unit in user space. It is associated with a function and its own stack. ULTs provide standard thread semantics at a very low context-switching cost. ULTs can make blocking calls and explicitly yield control.
* **Work Unit (WU)**: a lightweight execution unit, such as ULT or tasklet.

# Conventions
## Naming Convention
In Argobots, all names including data types, constants, and APIs begin with `ABT_` prefix. Following name after `ABT_` prefix for data types and APIs is all lower-case, but constants use all capital letters.

## API Convention
```
int ABT_xxx()
```
All public APIs begin with `ABT_` prefix and return an error code of `int` type. `xxx` is lower-case.
The arguments of Argobots APIs in this document are marked as `[in]`, `[out]`, `[in,out]` to indicate how they are used during execution of the routine. Their meanings are:
* `[in]`: the argument is used as the input and it is not updated during execution of the routine.
* `[out]`: the argument is used as the output and it is updated in the routine.
* `[in,out]`: the argument is used as both the input and the output.

# Data Types
## Object Types
All Argobots objects are opaque to users and can only be accessed via handles. All public APIs use handles or pointers to handles.

| Type Name             | Description                           |
| --------------------- | ------------------------------------- |
| `ABT_xstream`         | Handle to Execution Stream (ES)       |
| `ABT_xstream_barrier` | Handle to ES barrier                  |
| `ABT_sched`           | Handle to scheduler                   |
| `ABT_sched_config`    | Handle to scheduler configuration     |
| `ABT_pool`            | Handle to pool                        |
| `ABT_pool_config`     | Handle to pool configuration          |
| `ABT_unit`            | Handle to scheduling unit             |
| `ABT_thread`          | Handle to User-level Thread (ULT)     |
| `ABT_thread_attr`     | Handle to ULT attribute               |
| `ABT_task`            | Handle to tasklet                     |
| `ABT_key`             | Handle to work unit-specific data key |
| `ABT_mutex`           | Handle to mutex                       |
| `ABT_mutex_attr`      | Handle to mutex attribute             |
| `ABT_rwlock`          | Handle to readers-write lock          |
| `ABT_cond`            | Handle to condition variable          |
| `ABT_eventual`        | Handle to eventual                    |
| `ABT_future`          | Handle to future                      |
| `ABT_barrier`         | Handle to work unit barrier           |
| `ABT_timer`           | Handle to timer                       |

## Scalar Types
| Type Name               | Description                          |
| ----------------------- | ------------------------------------ |
| `ABT_bool`              | Boolean type                         |
| `ABT_xstream_state`     | ES state                             |
| `ABT_sched_predef`      | Predefined scheduler                 |
| `ABT_sched_state`       | Scheduler state                      |
| `ABT_sched_type`        | Scheduler type                       |
| `ABT_sched_config_type` | Data type of scheduler configuration |
| `ABT_pool_kind`         | Pool kind                            |
| `ABT_pool_access`       | Pool access mode                     |
| `ABT_unit_type`         | Scheduling unit type                 |
| `ABT_thread_state`      | ULT state                            |
| `ABT_thread_id`         | ULT ID                               |
| `ABT_task_state`        | Tasklet state                        |
| `ABT_event_kind`        | Event kind                           |

## Other Types

# Null Objects
The following constants are defined to indicate handles to null objects for some public data types.

# Constants

# Error Classes
