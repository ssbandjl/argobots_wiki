# Introduction
Condition variables are signal/wait synchronization mechanism. A ULT waits on a condition variable until it is signaled by another ULT. It is also possible that many ULTs wait on the same condition variable and a different ULT signals (broadcasts) to all waiting ULTs to wake them up.
