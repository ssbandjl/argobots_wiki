* [Overview](#overview)
* [Execution Model](#execution-model)
  * [User-level Threads and Tasklets](#user-level-threads-and-tasklets)
  * [Work Unit Migration](#work-unit-migration)
  * [Stackable Scheduler with Pluggable Strategies](#stackable-scheduler-with-pluggable-strategies)
* [Memory Model](#memory-model)
  * [Consistency Domains](#consistency-domains)
* [Organization of this Document](#organization-of-this-document)

# Overview
Concurrency in today's most powerful HPC systems is exploding. As the future exascale systems are likely to be composed of hundreds of millions of arithmetic units, future applications may contain billions of threads or tasks to exploit concurrency provided by the underlying hardware. However, achieving billion-way parallelism requires highly dynamic computational and data scheduling. The traditional, low-performing heavyweight threading and messaging models are not capable of supporting the massive parallelism because they often optimize the segregation and layering of components in favor of one dimension, e.g., computation or communication. Therefore, it is required to provide a lightweight execution model that combines low-latency thread and task scheduling with optimized data-movement functionality.

当今最强大的 HPC 系统的并发性呈爆炸式增长。 由于未来的百亿亿次级系统可能由数亿个运算单元组成，未来的应用程序可能包含数十亿个线程或任务以利用底层硬件提供的并发性。 然而，实现十亿路并行需要高度动态的计算和数据调度。 传统的、低性能的重量级线程和消息传递模型无法支持大规模并行，因为它们经常优化组件的隔离和分层以支持一维，例如计算或通信。 因此，需要提供一种轻量级的执行模型，将低延迟线程和任务调度与优化的数据移动功能相结合

**Argobots** is a low-level infrastructure that supports a lightweight thread and task model for massive concurrency. It will directly leverage the lowest-level constructs in the hardware and OS: lightweight notification mechanisms, data movement engines, memory mapping, and data placement strategies. It consists of an execution model and a memory model.

Argobots 是一个低级基础设施，支持用于大规模并发的轻量级线程和任务模型。 它将直接利用硬件和操作系统中的最低级别构造：轻量级通知机制、数据移动引擎、内存映射和数据放置策略。 它由执行模型和内存模型组成

# Execution Model 执行模型
Argobots supports two levels of parallelism: Execution Streams (ESs) and Work Units (WUs). ES is a sequential instruction stream that contains one or more WUs. It is bound to hardware resources and guarantee progress. WUs are lightweight execution unit, such as User-level Threads (ULTs) or tasklets. They are associated with function calls and each WU will execute to its completion. A WU can be associated with an ES and the ES runs all associated WUs according to the scheduling strategy. Basically, there is no parallel execution of WUs in a single ES. However, WUs in different ESs can be executed in parallel.

Argobots 支持两个级别的并行性：执行流 (ES) 和工作单元 (WU)。 ES 是包含一个或多个 WU 的顺序指令流。 它绑定硬件资源并保证进度。 WU 是轻量级执行单元，例如用户级线程 (ULT) 或 tasklet。 它们与函数调用相关联，每个 WU 都将执行直至完成。 一个WU可以关联一个ES，ES根据调度策略运行所有关联的WU。 基本上，单个 ES 中没有 WU 的并行执行。 但是，不同 ES 中的 WU 可以并行执行。

Two abstractions are envisioned in Argobots. The first abstraction supports lightweight concurrent execution, such as ULTs or tasklets, that can dynamically and efficiently adapt to the tug of requirements from applications and hardware resources. ULTs and tasklets must be efficiently scheduled based on power, resilience, memory locality, and capabilities. The second abstraction supports low-latency, lightweight, message-driven activation.

Argobots 设想了两种抽象。 第一个抽象支持轻量级并发执行，例如 ULT 或 tasklet，它们可以动态有效地适应来自应用程序和硬件资源的需求。 ULT 和 tasklet 必须根据功率、弹性、内存位置和功能进行有效调度。 第二个抽象支持低延迟、轻量级、消息驱动的激活。

The guiding design principle for scheduler is the cooperative, nonpreemptive activation of schedulable units of computation (ULTs or tasklets) on the managed hardware resources. Localized scheduling strategies such as those used in current runtime systems, while efficient for short execution, are unaware of global strategies and priorities. Adaptability and dynamic system behavior must be handled by scheduling strategies that can change over time or be customized for a particular algorithm or data structure. "Plugging" in a specialized scheduling strategy lets the OS/R handle the mechanism and utilize lightweight notification features while leaving the policy to higher levels of the system-software stack.

调度程序的指导设计原则是在托管硬件资源上协作、非抢占式地激活可调度计算单元（ULT 或 tasklet）。 本地化调度策略（例如当前运行时系统中使用的调度策略）虽然对短时间执行有效，但不了解全局策略和优先级。 适应性和动态系统行为必须通过可以随时间变化或针对特定算法或数据结构定制的调度策略来处理。 “插入”专门的调度策略让 OS/R 处理该机制并利用轻量级通知功能，同时将策略留给系统软件堆栈的更高级别。

## User-level Threads and Tasklets
User-level Threads (ULTs) and tasklets offer applications an abstraction for fine-grained parallelism. Each ULT or tasklet is an independent execution unit scheduled by the runtime system. They differ in subtle aspects that make each of them better suited for some programming motifs. For example, a ULT has its own persistent stack region, while a tasklet borrows the stack of its host ES's scheduler. We expect the higher-level programming models to decompose their computation in terms of these basic scheduling units.

用户级线程 (ULT) 和 tasklet 为应用程序提供了细粒度并行性的抽象。 每个ULT或tasklet都是一个独立的执行单元，由runtime系统调度。 它们在细微的方面有所不同，这使得它们中的每一个都更适合某些编程主题。 例如，ULT 有自己的持久堆栈区域，而 tasklet 借用其宿主 ES 调度程序的堆栈。 我们期望更高级别的编程模型根据这些基本调度单元分解它们的计算。

ULTs are excellent for expressing parallelism in terms of persistent contexts whose flow of control pauses and resumes based on the flow of data. A common example is an over-decomposed application that uses blocking receives (or futures) to wait for remote data. Unlike traditional OS threads, ULTs are not intended to be preempted. They cooperatively yield control to the scheduler, for example, when they wait for remote data or just wish to let other work progress.

ULT 非常适合根据持久性上下文表达并行性，其控制流根据数据流暂停和恢复。 一个常见的例子是过度分解的应用程序，它使用阻塞接收（或期货）来等待远程数据。 与传统的 OS 线程不同，ULT 不打算被抢占。 他们协作将控制权交给调度程序，例如，当他们等待远程数据或只是希望让其他工作继续进行时。

Tasklet in Argobots is an indivisible unit of work, with dependence only on its input data, and typically providing output data upon completion. Application-level examples of tasks include a force-evaluation object in molecular dynamics, which executes when two sets of atoms are available, or a trailing update in LU decomposition, which executes the matrix-matrix (GEMM) computation when the two blocks of matrices are available. Tasklets do not explicitly yield control but run to completion before returning control to the scheduler that invoked them.

Argobots 中的 Tasklet 是一个不可分割的工作单元，仅依赖于其输入数据，通常在完成时提供输出数据。 任务的应用级示例包括分子动力学中的力评估对象，当两组原子可用时执行，或 LU 分解中的尾随更新，当两个矩阵块可用时执行矩阵-矩阵 (GEMM) 计算 可用。 Tasklet 不会显式地让出控制权，而是在将控制权返回给调用它们的调度程序之前运行至完成。

ULTs and tasklets can be scheduled independently based on their own policies, including arrival of remote data, availability of memory, or data dependencies. Both ULTs and tasklets may maintain affinity to one or more data objects, which the scheduler will consider in the decision making. Once ready to execute, both ULTs and tasklets are managed by a scheduler with a common work pool.

ULT 和 tasklet 可以根据它们自己的策略独立调度，包括远程数据的到达、内存的可用性或数据依赖性。 ULT 和 tasklet 都可以保持与一个或多个数据对象的亲和力，调度程序将在决策制定中考虑这些对象。 一旦准备好执行，ULT 和 tasklet 都由具有公共工作池的调度程序管理。

## Work Unit Migration 工作单位迁移
Argobots supports migration of work units (ULTs or tasklets) between different locations, i.e., ESs or scheduler pools. Basically, all ULTs, which are created by the user, can be migrated unless they are terminated. However, tasklets can only be migrated if they have not started the execution. Migration operation can be blocking or non-blocking depending on the request. And, if a callback function is set, it will be invoked when the migration happens.

Argobots 支持在不同位置（即 ES 或调度程序池）之间迁移工作单元（ULT 或 tasklet）。 基本上，所有由用户创建的 ULT 都可以迁移，除非它们被终止。 但是，tasklet 只有在还没有开始执行的情况下才能被迁移。 迁移操作可以是阻塞的或非阻塞的，具体取决于请求。 并且，如果设置了回调函数，它将在迁移发生时被调用。

Migration operations are divided into two categories: work unit-centric migration and location-centric migration. Both operations can be used as push and pull interfaces for migration.

迁移操作分为两类：以工作单元为中心的迁移和以位置为中心的迁移。 这两个操作都可以用作迁移的推送和拉取接口。

Work unit-centric migration is to migrate one work unit to another ES or another pool. Depending on who requests the migration, it can be a push or pull operation. For example, if a work unit W0 requests to migrate a work unit W1 to an ES or a pool that is different from that of W0, it is a push operation. It pushes a work unit W1 to a different location. However, if W1 is migrated to the ES or the pool where the calling work unit W0 is residing, it is a pull operation. Work unit-centric migration also includes the interface to make a work unit orphaned, i.e., it makes the work unit have no association with any ES and any pool.

以工作单元为中心的迁移是将一个工作单元迁移到另一个 ES 或另一个池。 根据谁请求迁移，它可以是推送或拉取操作。 例如工作单元W0请求将工作单元W1迁移到与W0不同的ES或pool中，这就是push操作。 它将工作单元 W1 推到不同的位置。 但是，如果将W1迁移到调用工作单元W0所在的ES或pool中，则为pull操作。 以工作单元为中心的迁移还包括使工作单元成为孤立的接口，即它使工作单元与任何 ES 和任何池都没有关联。

Location-centric migration is to relocate a certain number of work units associated with a specific ES or pool to a different ES or pool. This approach focuses on locations where the migration happens rather than work units to be migrated. The migration routines of this category specify the source location, the target location, and the number of work units to be migrated, but they do not specify which work units to migrate. The location-centric migration can be viewed as a push operation if it is requested by a work unit whose associated ES or pool is different from the target. Otherwise, i.e., if the target is the same as the ES or the pool associated with the calling work unit, it behaves like a pull operation.

以位置为中心的迁移是将与特定 ES 或池关联的一定数量的工作单元迁移到不同的 ES 或池。 这种方法侧重于迁移发生的位置，而不是要迁移的工作单元。 此类迁移例程指定源位置、目标位置和要迁移的工作单元数，但它们不指定要迁移哪些工作单元。 如果由其关联的 ES 或池与目标不同的工作单元请求，则可以将以位置为中心的迁移视为推送操作。 否则，即如果目标与 ES 或与调用工作单元关联的池相同，则它的行为类似于拉操作。

## Stackable Scheduler with Pluggable Strategies 具有可插入策略的可堆叠调度器
The overall scheduling framework is centered on the notion of stackable or nested schedulers with multiple pluggable scheduling strategies. Plugging in custom strategies modifies the scheduling rules for all tasks managed by that scheduler instance. Pluggability also expects the input required for these decisions to be available at a single point in the OS/R software stack. In addition to pluggable strategies, the OS/R scheduler will permit nesting (stacking) scheduler engines specific to the programming language or application component. For example, the system could accept a scheduler for each module in an application; and based on dependencies or relative module priorities, the OS/R scheduler may invoke one of the stacked schedulers. This can now activate its tasks on the managed hardware and yield control back upon completion. Similarly stacked schedulers might be a result of multiple programming languages interacting in the context of the single OS/R scheduler. A framework based on task-dependency analysis might choose to schedule tasks once dependencies are satisfied. This will be expressed as a stacked scheduler that handles the DAG dependency analysis and schedules tasks or simply propagates them to the OS/R scheduler once they are "ready" to be activated.

整个调度框架以具有多个可插入调度策略的可堆叠或嵌套调度器的概念为中心。 插入自定义策略会修改该调度程序实例管理的所有任务的调度规则。 可插拔性还期望这些决策所需的输入在 OS/R 软件堆栈中的单个点可用。 除了可插拔策略之外，OS/R 调度程序将允许特定于编程语言或应用程序组件的嵌套（堆叠）调度程序引擎。 例如，系统可以为应用程序中的每个模块接受一个调度程序； 并且基于依赖关系或相关模块优先级，OS/R 调度器可以调用堆栈调度器之一。 现在，它可以在托管硬件上激活其任务，并在完成后交出控制权。 类似的堆栈调度器可能是多种编程语言在单个 OS/R 调度器的上下文中交互的结果。 基于任务依赖性分析的框架可能会选择在满足依赖性后安排任务。 这将被表示为一个堆栈式调度器，它处理 DAG 依赖性分析和调度任务，或者在它们“准备好”被激活时简单地将它们传播到 OS/R 调度器。

# Memory Model
Current communication libraries are not well integrated with the scheduler or the memory manager. All three must work together in order to support optimized data movement across nodes and provide the features needed by the higher layers that built the specific programming environment. Argobots provides an integrated memory model that supports eventual consistency.

当前的通信库没有很好地与调度程序或内存管理器集成。 这三者必须协同工作，以支持跨节点的优化数据移动，并提供构建特定编程环境的更高层所需的功能。 Argobots 提供了一个支持最终一致性的集成内存模型。

## Consistency Domains 一致性域
Current hardware couples consistency and coherence into one, but this can be expensive on NUMA machines or non-DRAM like memories. Argobots decouples consistency and coherence by explicitly dividing memory space into difference domains. There are three levels of consistency in Argobots: 1. Consistency Domains (CDs); 2. Non-coherent Load/Store Domains (NCLSDs); 3. Outside an NCLSD.

当前的硬件将一致性和一致性合二为一，但这在 NUMA 机器或非 DRAM（如内存）上可能代价高昂。 Argobots 通过明确地将内存空间划分为不同的域来解耦一致性和连贯性。 Argobots 中的一致性分为三个级别： 1. Consistency Domains (CDs)； 2. 非相干加载/存储域 (NCLSD)； 3. 在 NCLSD 之外。

A Consistency Domain is a region of memory where data becomes eventually consistent, which means if no new updates are made to a given data item, eventually all accesses to that item will return the last updated value. At the same time, immediate consistency can be enforced with memory barriers. In NCLSD, data are accessed using load/store, but not hardware consistency is provided. Outside an NCLSD, explicit Put/Get/Messaging models are used to move data.

一致性域是数据最终变得一致的内存区域，这意味着如果没有对给定数据项进行新更新，最终对该项目的所有访问都将返回最后更新的值。 同时，可以通过内存屏障强制执行即时一致性。 在 NCLSD 中，使用加载/存储访问数据，但不提供硬件一致性。 在 NCLSD 之外，显式 Put/Get/Messaging 模型用于移动数据。

# Organization of this Document
The remaining parts of this document begin with Terms and Conventions, which explains Argobots terms and conventions used throughout this document. It then describes Argobots components and their API.
本文档的其余部分以术语和约定开头，解释了本文档中使用的 Argobots 术语和约定。 然后描述了 Argobots 组件及其 API。
