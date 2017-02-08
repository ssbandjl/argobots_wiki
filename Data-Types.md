This page presents data types, null objects, constants, and error classes used in this document.

* [Data Types](#data-types)
  * [Object Types](#object-types)
  * [Scalar Types](#scalar-types)
  * [Other Types](#other-types)
* [Null Objects](#null-objects)
* [Constants](#constants)
  * [Constant Values](#constant-values)
  * [Enum Values](#enum-values)
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
| `ABT_cond`            | Handle to condition variable          |
| `ABT_rwlock`          | Handle to readers-write lock          |
| `ABT_eventual`        | Handle to eventual                    |
| `ABT_future`          | Handle to future                      |
| `ABT_barrier`         | Handle to work unit barrier           |
| `ABT_timer`           | Handle to timer                       |

## Scalar Types
| Type Name               | Description                                   |
| ----------------------- | --------------------------------------------- |
| `ABT_bool`              | Boolean type                                  |
| `ABT_xstream_state`     | ES state (see [values](#abt_xstream_state))   |
| `ABT_sched_predef`      | Predefined scheduler (see [values](#abt_sched_predef)) |
| `ABT_sched_state`       | Scheduler state (see [values](#abt_sched_state))       |
| `ABT_sched_type`        | Scheduler type (see [values](#abt_sched_type))         |
| `ABT_sched_config_type` | Data type of scheduler configuration variable (see [values](#abt_sched_config_type)) |
| `ABT_pool_kind`         | Pool kind (see [values](#abt_pool_kind))            |
| `ABT_pool_access`       | Pool access mode (see [values](#abt_pool_access))   |
| `ABT_unit_type`         | Scheduling unit type (see [values](#abt_unit_type)) |
| `ABT_thread_state`      | ULT state (see [values](#abt_thread_state))         |
| `ABT_thread_id`         | ULT ID                                              |
| `ABT_task_state`        | Tasklet state (see [values](#abt_task_state))       |
| `ABT_event_kind`        | Event kind (see [values](#abt_event_kind))          |

## Other Types
These are function pointers and struct types for scheduler and pool.

### Scheduler-related Types
| Type Name                    | Description                                                                       |
| ---------------------------- | --------------------------------------------------------------------------------- |
| `ABT_sched_config_var`       | Scheduler configuration variable, consisting of index and `ABT_sched_config_type` |
| `ABT_sched_init_fn`          | Scheduler init function                                                           |
| `ABT_sched_run_fn`           | Scheduler run function                                                            |
| `ABT_sched_free_fn`          | Scheduler free function                                                           |
| `ABT_sched_get_migr_pool_fn` | Function that returns a pool ready for receiving a migration                      |
| `ABT_sched_def`              | Struct of `ABT_sched_type` and above functions                                    |

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

| Constant                   | Description                                |
| -------------------------- | ------------------------------------------ |
| `ABT_XSTREAM_NULL`         | Handle to the null ES                      |
| `ABT_XSTREAM_BARRIER_NULL` | Handle to the null ES barrier              |
| `ABT_SCHED_NULL`           | Handle to the null scheduler               |
| `ABT_SCHED_CONFIG_NULL`    | Handle to the null scheduler configuration |
| `ABT_POOL_NULL`            | Handle to the null pool                    |
| `ABT_POOL_CONFIG_NULL`     | Handle to the null pool configuration      |
| `ABT_UNIT_NULL`            | Handle to the null unit                    |
| `ABT_THREAD_NULL`          | Handle to the null ULT                     |
| `ABT_THREAD_ATTR_NULL`     | Handle to the null ULT attribute           |
| `ABT_TASK_NULL`            | Handle to the null tasklet                 |
| `ABT_KEY_NULL`             | Handle to the null key                     |
| `ABT_MUTEX_NULL`           | Handle to the null mutex                   |
| `ABT_MUTEX_ATTR_NULL`      | Handle to the null mutex attribute         |
| `ABT_COND_NULL`            | Handle to the null condition variable      |
| `ABT_RWLOCK_NULL`          | Handle to the null readers-write lock      |
| `ABT_EVENTUAL_NULL`        | Handle to the null eventual                |
| `ABT_FUTURE_NULL`          | Handle to the null future                  |
| `ABT_BARRIER_NULL`         | Handle to the null work unit barrier       |
| `ABT_TIMER_NULL`           | Handle to the null timer                   |

# Constants
## Constant Values
| Constant               | Description                                                             |
| ---------------------- | ----------------------------------------------------------------------- |
| `ABT_TRUE`             | True value for `ABT_bool`                                               |
| `ABT_FALSE`            | False value for `ABT_bool`                                              |
| `ABT_XSTREAM_ANY_RANK` | Rank for indicating any ES                                              |
| `ABT_CB_ON_TARGET`     | Indicates that the callback function needs to be executed on the target |
| `ABT_CB_ON_SOURCE`     | Indicates that the callback function needs to be executed on the source |

## Enum Values
### ABT_xstream_state
Each value represents ES's state.
```c
enum ABT_xstream_state {
    ABT_XSTREAM_STATE_CREATED,
    ABT_XSTREAM_STATE_READY,
    ABT_XSTREAM_STATE_RUNNING,
    ABT_XSTREAM_STATE_TERMINATED
};
```

### ABT_sched_predef
Each value is used to create a predefined scheduler.
```c
enum ABT_sched_predef {
    ABT_SCHED_DEFAULT,   /* Default scheduler */
    ABT_SCHED_BASIC,     /* Basic scheduler */
    ABT_SCHED_PRIO,      /* Priority scheduler */
    ABT_SCHED_RANDWS     /* Random work-stealing scheduler */
};
```

### ABT_sched_state
Each value represents schedulers' state.
```c
enum ABT_sched_state {
    ABT_SCHED_STATE_READY,
    ABT_SCHED_STATE_RUNNING,
    ABT_SCHED_STATE_STOPPED,
    ABT_SCHED_STATE_TERMINATED
};
```

### ABT_sched_type
Each value represents a type of scheduler, such as ULT-type or tasklet-type.
```c
enum ABT_sched_type {
    ABT_SCHED_TYPE_ULT,  /* can yield */
    ABT_SCHED_TYPE_TASK  /* cannot yield */
};
```

### ABT_sched_config_type
Each value is used to indicate the type of scheduler configuration variable (i.e., `ABT_sched_config_var`).
```c
enum ABT_sched_config_type {
    ABT_SCHED_CONFIG_INT    = 0, /* Parameter of type int */
    ABT_SCHED_CONFIG_DOUBLE = 1, /* Parameter of type double */
    ABT_SCHED_CONFIG_PTR    = 2, /* Parameter of type pointer */
};
```

### ABT_pool_kind
Each value represents a kind of predefined pool and is used to create a predefined pool.
```c
enum ABT_pool_kind {
    ABT_POOL_FIFO
};
```

### ABT_pool_access
Each value represents an access mode for pool. It is used when creating a pool.
```c
enum ABT_pool_access {
    ABT_POOL_ACCESS_PRIV, /* Used by only one ES */
    ABT_POOL_ACCESS_SPSC, /* Producers on ES1, consumers on ES2 */
    ABT_POOL_ACCESS_MPSC, /* Producers on any ES, consumers on the same ES */
    ABT_POOL_ACCESS_SPMC, /* Producers on the same ES, consumers on any ES */
    ABT_POOL_ACCESS_MPMC  /* Producers on any ES, consumers on any ES */
};
```

### ABT_unit_type
Each value represents the type of object that an `ABT_unit` object contains.
```c
enum ABT_unit_type {
    ABT_UNIT_TYPE_THREAD,   /* ULT */
    ABT_UNIT_TYPE_TASK,     /* Tasklet */
    ABT_UNIT_TYPE_XSTREAM,  /* ES */
    ABT_UNIT_TYPE_EXT       /* External thread */
};
```

### ABT_thread_state
Each value represents ULT's state.
```c
enum ABT_thread_state {
    ABT_THREAD_STATE_READY,
    ABT_THREAD_STATE_RUNNING,
    ABT_THREAD_STATE_BLOCKED,
    ABT_THREAD_STATE_TERMINATED
};
```

### ABT_task_state
Each value represents tasklet's state.
```c
enum ABT_task_state {
    ABT_TASK_STATE_CREATED,
    ABT_TASK_STATE_READY,
    ABT_TASK_STATE_RUNNING,
    ABT_TASK_STATE_TERMINATED
};
```

### ABT_event_kind
Each value represents a kind of event.
```c
enum ABT_event_kind {
    ABT_EVENT_STOP_XSTREAM,     /* stop the target ES */
    ABT_EVENT_ADD_XSTREAM,      /* create a new ES */
};
```

# Error Classes
| Constant                      | Description                                       |
| ----------------------------- | ------------------------------------------------- |
| `ABT_SUCCESS`                 | Successful return code                            |
| `ABT_ERR_UNINITIALIZED`       | Argobots is not initialized                       |
| `ABT_ERR_MEM`                 | Memory allocation failure                         |
| `ABT_ERR_OTHER`               | Other error                                       |
| `ABT_ERR_INV_XSTREAM`         | Invalid ES                                        |
| `ABT_ERR_INV_XSTREAM_RANK`    | Invalid ES rank                                   |
| `ABT_ERR_INV_XSTREAM_BARRIER` | Invalid ES barrier                                |
| `ABT_ERR_INV_SCHED`           | Invalid scheduler                                 |
| `ABT_ERR_INV_SCHED_KIND`      | Invalid scheduler kind                            |
| `ABT_ERR_INV_SCHED_PREDEF`    | Invalid predefined scheduler                      |
| `ABT_ERR_INV_SCHED_TYPE`      | Invalid scheduler type                            |
| `ABT_ERR_INV_SCHED_CONFIG`    | Invalid scheduler configuration                   |
| `ABT_ERR_INV_POOL`            | Invalid pool                                      |
| `ABT_ERR_INV_POOL_ACCESS`     | Invalid pool access mode                          |
| `ABT_ERR_INV_UNIT`            | Invalid scheduling unit                           |
| `ABT_ERR_INV_THREAD`          | Invalid ULT                                       |
| `ABT_ERR_INV_THREAD_ATTR`     | Invalid ULT attribute                             |
| `ABT_ERR_INV_TASK`            | Invalid tasklet                                   |
| `ABT_ERR_INV_KEY`             | Invalid key                                       |
| `ABT_ERR_INV_MUTEX`           | Invalid mutex                                     |
| `ABT_ERR_INV_MUTEX_ATTR`      | Invalid mutex attribute                           |
| `ABT_ERR_INV_COND`            | Invalid condition variable                        |
| `ABT_ERR_INV_RWLOCK`          | Invalid readers-writer lock                       |
| `ABT_ERR_INV_EVENTUAL`        | Invalid eventual                                  |
| `ABT_ERR_INV_FUTURE`          | Invalid future                                    |
| `ABT_ERR_INV_BARRIER`         | Invalid work unit barrier                         |
| `ABT_ERR_INV_TIMER`           | Invalid timer                                     |
| `ABT_ERR_INV_EVENT`           | Invalid event                                     |
| `ABT_ERR_XSTREAM`             | ES-related error                                  |
| `ABT_ERR_XSTREAM_STATE`       | ES state error                                    |
| `ABT_ERR_XSTREAM_BARRIER`     | ES barrier-related error                          |
| `ABT_ERR_SCHED`               | Scheduler-related error                           |
| `ABT_ERR_SCHED_CONFIG`        | Scheduler configuration error                     |
| `ABT_ERR_POOL`                | Pool-related error                                |
| `ABT_ERR_UNIT`                | Scheduling unit-related error                     |
| `ABT_ERR_THREAD`              | ULT-related error                                 |
| `ABT_ERR_TASK`                | Task-related error                                |
| `ABT_ERR_KEY`                 | Key-related error                                 |
| `ABT_ERR_MUTEX`               | Mutex-related error                               |
| `ABT_ERR_MUTEX_LOCKED`        | Return value when the mutex is already locked     |
| `ABT_ERR_COND`                | Condition variable-related error                  |
| `ABT_ERR_COND_TIMEDOUT`       | Return value when condition variable is timed out |
| `ABT_ERR_RWLOCK`              | rwlock-related error                              |
| `ABT_ERR_EVENTUAL`            | Eventual-related error                            |
| `ABT_ERR_FUTURE`              | Future-related error                              |
| `ABT_ERR_BARRIER`             | Work unit barrier-related error                   |
| `ABT_ERR_TIMER`               | Timer-related error                               |
| `ABT_ERR_EVENT`               | Event-related error                               |
| `ABT_ERR_MIGRATION_TARGET`    | Target does not accept migration                  |
| `ABT_ERR_MIGRATION_NA`        | Migration is not available                        |
| `ABT_ERR_MISSING_JOIN`        | An ES or more did not join                        |
| `ABT_ERR_FEATURE_NA`          | Feature not available                             |