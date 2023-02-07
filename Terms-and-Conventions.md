This page describes Argobots terms and conventions used throughout this document.

* [Glossary](#glossary) 术语
* [Conventions](#conventions)
  * [Naming Convention](#naming-convention)
  * [API Convention](#api-convention)

# Glossary 术语
* **Condition Variable**: a condition on which ULTs are waiting until it is signaled. 条件变量：ULT 等待直到收到信号的条件。
* **Execution Stream (ES)**: a sequential instruction stream that contains one or more work units. 执行流（ES）：包含一个或多个工作单元的顺序指令流。
* **Future**: a mechanism for passing a value between work units, allowing a work unit to wait for a value that is set asynchronously. Future：一种在工作单元之间传递值的机制，允许工作单元等待异步设置的值。
* **Handle**: an opaque reference to an Argobots object. 句柄：对 Argobots 对象的不透明引用。
* **Mutex**: a synchronization method to support mutual exclusion between work units. Mutex：一种支持工作单元之间互斥的同步方法。
* **Object**: abstract representation of resources that are manipulated by the Argobots API. For example, there are ES objects, ULT objects, tasklet objects, etc. 对象：由 Argobots API 操作的资源的抽象表示。 比如有ES对象，ULT对象，tasklet对象等。
* **Pool**: a data structure that contains scheduling units. 池：包含调度单元的数据结构。
* **Scheduler**: an object that controls execution order of scheduling units in the ES. A scheduler is associated with an ES. Scheduler：控制ES中调度单元执行顺序的对象。 调度程序与 ES 相关联。
* **Scheduling Unit (SU)**: an object handled by a scheduler with a scheduling policy. It includes work units, such as ULTs or tasklets, and schedulers. 调度单元 (SU)：由具有调度策略的调度器处理的对象。 它包括工作单元，例如 ULT 或 tasklet，以及调度程序。
* **Tasklet**: an indivisible unit of work, with dependence only on its input data, and typically providing output data upon completion. Tasklet is associated with a function but it does not have its own stack. It is not allowed to make blocking calls and does not explicitly yield control. Tasklet：一个不可分割的工作单元，只依赖于它的输入数据，通常在完成时提供输出数据。 Tasklet 与一个函数相关联，但它没有自己的堆栈。 不允许进行阻塞调用，也不会显式让出控制权。
* **User-level Thread (ULT)**: an independent execution unit in user space. It is associated with a function and its own stack. ULTs provide standard thread semantics at a very low context-switching cost. ULTs can make blocking calls and explicitly yield control. 用户级线程（ULT）：用户空间中的独立执行单元。 它与一个函数和它自己的堆栈相关联。 ULT 以非常低的上下文切换成本提供标准线程语义。 ULT 可以进行阻塞调用并显式让出控制权。
* **Work Unit (WU)**: a lightweight execution unit, such as ULT or tasklet. Work Unit (WU)：轻量级的执行单元，例如ULT或tasklet。

# Conventions 惯例
## Naming Convention
In Argobots, all names including data types, constants, and APIs begin with `ABT_` prefix. Following name after `ABT_` prefix for data types and APIs is all lower-case, but constants use all capital letters.

在 Argobots 中，包括数据类型、常量和 API 在内的所有名称都以 ABT_ 前缀开头。 数据类型和 API 的 ABT_ 前缀后的名称全部小写，但常量全部使用大写字母。

## API Convention
```
int ABT_xxx()
```
All public APIs begin with `ABT_` prefix and return an error code of `int` type. `xxx` is lower-case.
The arguments of Argobots APIs in this document are marked as `[in]`, `[out]`, `[in,out]` to indicate how they are used during execution of the routine. Their meanings are:

所有公共 API 都以 ABT_ 前缀开头，并返回 int 类型的错误码。 xxx 是小写的。 本文档中 Argobots API 的参数标记为 [in]、[out]、[in,out] 以指示它们在例程执行期间的使用方式。 它们的含义是：

* `[in]`: the argument is used as the input and it is not updated during execution of the routine.
* `[out]`: the argument is used as the output and it is updated in the routine.
* `[in,out]`: the argument is used as both the input and the output.

[in]：参数用作输入，在例程执行期间不更新。
[out]：参数用作输出并在例程中更新。
[in,out]：参数同时用作输入和输出。

