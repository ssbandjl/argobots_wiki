# Error Message
## ABT_error_get_str()
```c
int ABT_error_get_str(int err, char *str, size_t *len)
```
* Get the string of error code and its length.
* Parameters
  * [in] `err`: error code
  * [out] `str`: error string
  * [out] `len`: the length of string in bytes
* Return values
  * On success, `ABT_SUCCESS` is returned.
  * On error, a non-zero error code is returned.
* Details
  * `ABT_error_get_str()` returns the string of given error code through `str` and its length in bytes via `len`. If `str` is `NULL`, only `len` is returned. If `str` is not `NULL`, it should have enough space to save `len` bytes of characters. If `len` is `NULL`, `len` is ignored.
