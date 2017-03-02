This group is for work units.

# Work Unit and Pool
## ABT_unit_set_associated_pool()
```c
int ABT_unit_set_associated_pool(ABT_unit unit, ABT_pool pool)
```
* Set the associated pool for the target work unit.
* Parameters
  * [in] `unit`: handle to the work unit
  * [in] `pool`: handle to the pool
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_unit_set_associated_pool()` changes the associated pool of the target work unit `unit`, such as ULT or tasklet, to `pool`. This function must be called after `unit` is popped from its original associated pool (i.e., `unit` must not be inside any pool), which is the pool where `unit` was residing in.