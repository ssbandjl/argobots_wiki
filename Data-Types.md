This page presents data types, null objects, constants, and error classes used in this document.

* [Data Types](#data-types)
  * [Object Types](#object-types)
  * [Scalar Types](#scalar-types)
  * [Other Types](#other-types)
* [Null Objects](#null-objects)
* [Constants](#constants)
* [Error Classes](#error-classes)

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
| `ABT_sched_config_type` | Data type of scheduler configuration variable |
| `ABT_pool_kind`         | Pool kind                            |
| `ABT_pool_access`       | Pool access mode                     |
| `ABT_unit_type`         | Scheduling unit type                 |
| `ABT_thread_state`      | ULT state                            |
| `ABT_thread_id`         | ULT ID                               |
| `ABT_task_state`        | Tasklet state                        |
| `ABT_event_kind`        | Event kind                           |

## Other Types
These are function pointers and struct types for scheduler and pool.

### Scheduler-related Types
| Type Name                    | Description                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| `ABT_sched_config_var`       | Scheduler configuration variable, consisting of index and `ABT_sched_config_type` |
| `ABT_sched_init_fn`          | Scheduler init function                                      |
| `ABT_sched_run_fn`           | Scheduler run function                                       |
| `ABT_sched_free_fn`          | Scheduler free function                                      |
| `ABT_sched_get_migr_pool_fn` | Function that returns a pool ready for receiving a migration |
| `ABT_sched_def`              | Struct of `ABT_sched_type` and above functions               |

### Pool-related Types
| Type Name                        | Description                                                            |
| -------------------------------- | ---------------------------------------------------------------------- |
| `ABT_unit_get_type_fn`           | Function that returns `ABT_uint_type` from `ABT_unit`                  |
| `ABT_unit_get_thread_fn`         | Function that returns `ABT_thread` from `ABT_unit`                     |
| `ABT_unit_get_task_fn`           | Function that returns `ABT_task` from `ABT_unit`                       |
| `ABT_unit_is_in_pool_fn`         | Function that tells whether `ABT_unit` is in a pool or not             |
| `ABT_unit_create_from_thread_fn` | Function that creates `ABT_unit` from `ABT_thread`                     |
| `ABT_unit_create_from_task_fn`   | Function that creates `ABT_unit` from `ABT_task`                       |
| `ABT_unit_free_fn`               | Function that frees `ABT_unit`                                         |
| `ABT_pool_init_fn`               | Pool init function                                                     |
| `ABT_pool_get_size_fn`           | Function that returns the pool's size (i.e., the number of work units) |
| `ABT_pool_push_fn`               | Pool push function                                                     |
| `ABT_pool_pop_fn`                | Pool pop function                                                      |
| `ABT_pool_remove_fn`             | Function that removes a specific `ABT_unit` from the pool              |
| `ABT_pool_free_fn`               | Pool free function                                                     |
| `ABT_pool_def`                   | Struct of `ABT_pool_access` and above functions                        |

# Null Objects
The following constants are defined to indicate handles to null objects for some public data types.

# Constants

# Error Classes
