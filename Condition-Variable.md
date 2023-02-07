# Introduction
Condition variables are signal/wait synchronization mechanism. A ULT waits on a condition variable until it is signaled by another ULT. It is also possible that many ULTs wait on the same condition variable and a different ULT signals (broadcasts) to all waiting ULTs to wake them up.

条件变量是信号/等待同步机制。 一个 ULT 等待一个条件变量，直到它被另一个 ULT 发出信号。 也有可能许多 ULT 等待相同的条件变量和不同的 ULT 信号（广播）到所有等待的 ULT 以唤醒它们。
