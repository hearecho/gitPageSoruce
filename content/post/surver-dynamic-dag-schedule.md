---
title: "Surver Dynamic Dag Schedule"
date: 2021-08-22T09:58:40+08:00
draft: false
description: "动态DAG调度综述"
tags:
- 论文阅读
- DAG
- 算法
categories:
- 论文阅读
- 算法
mathjax: true
---

## 动态DAG调度综述

> 在异构系统中调度的核心问题是：由于异构系统中处理器的性能不尽相同，所以为了获得更高的性能。系统调度需要将应用程序的任务分配给合适的处理器，并对每个资源上的任务执行进行排序。给定一个有向无环图（DAG）建模的应用程序，调度问题处理的是在异构环境中映射每个任务以最小化执行时间（makspan）。

#### 动态调度基础

> 动态调度，任务在到达时分配给处理器，调度决策必须在运行时做出。调度决策基于在运行时可能更改的动态参数。在动态调度中，任务可以在运行时重新分配给其他处理器。动态调度相比静态调度灵活且速度更快。由于我们异构系统处理同一个任务的时间是不相同的，所以我们每次调度的时候应该考虑将任务放在最合适的处理器中。

### 相关的DAG调度算法

#### Monte Carlo 算法

> Stochastic DAG  scheduling  using  a  Monte Carlo  approach
>
> 该算法使用静态调度方法。主要目的是最大限度地缩短完工时间。该算法避免了适用于任意随机分布的随机变量的复杂计算。阈值用于优化调度。概率分布用于最小化完工时间。

#### CBHD 算法

> 基于集群的复制权重，用于异构系统上任务的高效调度和映射，将程序分解为子任务，同时与其他任务一起工作[12]。该算法将三重聚类算法与HEFT算法相结合，将任务分为相互关联的组，并对每个簇中的任务进行排序，以提高负载均衡调度算法的性能。其主要目标是最小化执行时间，最大化处理器利用率和处理器之间的负载平衡。

#### P-HEFT算法

> P-HEFT（并行异构最早完成时间）的算法。该算法在不影响最大完工时间的前提下，以很高的效率处理异构集群中的并行任务。该算法的主要目标是最小化具有不同到达时间的作业的最大完工时间和批处理时间，从而在作业执行期间更改分配给作业的处理器(看描述也不是动态算法)

####  High Performance and Energy Efficient Task Scheduling Algorithm

> 该算法关注调度长度和能量/功耗的最小化。该算法分为编译阶段和运行阶段。编译时间阶段有三个阶段，例如级别排序、任务优先级和处理器选择。在运行期间，为了节省能量，该算法将任务从繁忙节点重新调度到理想节点。该算法的性能明显优于传统算法。 
>
> 该算法为动态调度算法。

#### Constrained Earliest Finish Time (CEFT) Algorithm

> 受约束的最早完成时间。这种新方法是使用受约束关键路径（CCPs）的概念为异构系统提供更好的调度。一旦发现DAG中的CCPs，任务将使用整个CCPs的完成时间进行调度。这种方法有助于以较短的完工时间生成时间表，并且工作复杂度很低。

#### Sorted Nodes in Leveled DAG Division (SNLDD) Algorithm  

> 该算法称为高性能任务调度算法。这种类型的算法将DAG划分为不同的级别，每个级别根据计算时间按降序排序，从而减少任务之间的依赖性。静态任务调度适用于处理器数量有限的异构系统。这种类型可能会产生高质量的任务调度。DAG中所有任务的计算时间只计算一次，因此消除了运行时开销。处理器的平滑时间也被最小化，因为DAG被调平并分配给处理器。

#### Multi – Queue Balancing Algorithm

> 在功能单元中引入了调度概念。该算法具有各种类型的功能单元，不使用有限数量的硬件和软件。这种类型的算法称为离线非确定性算法。该算法的主要目标是最小化作业完成时间，根据资源的可用性将任务分配给机器，并最大限度地利用异构系统。每个任务都在其匹配的处理器上执行。也是静态调度算法。

#### Clustering Scheduling Strategies Algorithm

> 以减少应用程序的配置数量和执行时间，同时提高现场可编程门路（FPGA）器件的利用率。在可重构计算系统中，引入启发式调度策略和动态规划调度策略，将有向无环图（DAG）划分为多个簇。

#### SD-based Algorithm for Task Scheduling (SDBATS)

> 基于SD的算法是一种标准偏差方法，用于计算异构计算环境中可用资源上给定任务的预期执行时间。该算法将各种条件应用于标准任务图应用，如高斯消去和快速傅立叶变换应用。这种方法产生了高质量的计划，并产生了低成本的计划和系统效率。 可以用于动态调度算法。

#### Online Scheduling of Dynamic Task Graphs 

> 与传统方案相比，动态任务图的在线调度更为现实，任务图在运行时会发生变化，处理器间的通信也会固定数量。在线调度采用广播和点对点通信。该算法的主要目标是减少完工时间。

#### Node Duplication Modified Genetic Algorithm Approach (NMGA)

> 该方法采用了顶层和底层方法。它展示了节点复制技术的效率。主要目的是最小化完成时间并增加系统吞吐量。复制任务以减少总时间。

#### Heuristic based for Genetic Algorithm

> 该算法需要较少的计算时间[22]。该算法基于多处理器系统中的任务调度，通过选择合适的处理器来达到次优解。底层方法是通过选择合格的处理器来分配任务，以获得最小的完成时间。

#### List based Heuristic Task Scheduling

> 该方法具有不同类型的任务优先级，以获得最小的调度长度和速度。这只继承静态调度，静态调度进一步分为作业调度和任务调度。该算法的主要目的是最小化执行时间和通信延迟，最大限度地提高资源利用率。

#### Efficient Genetic Algorithm

> 该算法继承了定制的遗传算法，为HEDCS生成高质量的任务调度。新算法的性能由两种调度算法实现，即HEFT算法和DLS算法。这适用于随机生成的任务图和某些实际数值应用的任务图。该算法的主要目标是增加调度长度、加速比和效率。

#### BNP Scheduling Algorithms 

> 它将一个由边有向无环图（DAG）表示的并行程序引入一组同质处理器。主要目标是尽可能缩短完成时间。几类算法和一类调度的性能称为有界处理器数（BNP）。比较基于不同的调度参数，如最大完工时间、速度、处理器利用率和调度长度比。主要重点是增加处理器上的任务数量，以提高这些算法的性能

#### Predict Earliest Finish Time (PEFT) 

> 该算法具有相同的时间复杂度，即v任务和p处理器的时间复杂度为O（v2:p），在不增加与计算乐观代价表（OCT）相关的时间复杂度的情况下引入了一个特性。设计值是一个乐观的成本，因为计算中未测量处理器可用性。此算法仅基于用于对任务进行排序和处理器选择的OCT表。PEFT算法在调度长度比、效率和频率方面为异构系统执行基于列表的算法。

#### Hybrid Genetic Scheduling Algorithm

> 该算法为每个任务分配一个耦合因子，并通过避免大量通信时间将其调度到同一处理器上。该算法的目标是通过将相互强耦合的任务调度到同一个处理器上来生成高质量的初始解，并通过使用耦合初始解、随机解、，通过交叉和变异算子中的列表调度算法获得近似最优解。

#### Fork – Join Method (TSFJ)

> 该算法继承了多处理机系统中的任务调度，目的是最小化总执行时间，从而达到最大的速度和效率。应用程序由有向无环图（DAG）表示，任务根据fork-join结构分配给处理器。该算法的主要性能基于调度长度、加速效率和负载平衡。

#### Non – Dominated Sorting Genetic Algorithm – II (NSGA-II)

> 该算法在异构多处理器系统上引入了调度应用程序，主要用于单一目的，如执行时间、成本或总数据传输时间。所提出的算法是利用进化技术开发一种多目标调度算法，用于在多处理器环境中调度一组依赖于可用资源的任务。主要目标是最大限度地缩短完工时间。

#### Task Scheduling and Energy Conservation Techniques

> 该算法解决了多处理器系统中依赖任务的能量感知调度技术中的“最新技术”，以减少总体完工时间和能源消耗。

#### Critical Path Scheduling with T – level (CPST)

> 在CPST中，使用任务的t级值生成基于关键路径的任务序列，其中使用基于方差的计算和通信成本。任务按优先级的非递增顺序排序和选择，并在处理器上调度，以优化各种性能指标。目标是将任务映射到合适的资源上，并最小化最大完工时间。

#### Multi – Core Scheduling 

> 引入异构协处理器（即一个处理器和多个异构协处理器）,处理器处理控制信号&异构协处理器用于数据计算，因为需要多个协处理器调度器来确定哪些子任务传输到哪个协处理器。该算法采用迁移策略来提高资源的利用率。

#### Task Scheduling Algorithm using Merge Conditions

> 该算法在保留调度长度的同时减少了处理器数量。该算法根据条件找到合并对，并在不增加调度长度的情况下合并它们。函数用于查找最大对以减少合并次数。

#### Linear Programming based Scheduling Algorithms

> 该算法应用于具有不同特征的五种不同应用中的各种邻近查询，并通过额外的计算资源有力地提高了性能。所提出的算法是为并行邻近计算而设计的，以最小化计算资源所花费的最大时间。

#### Comparative Study of Scheduling Algorithm

> 一种称为调度算法比较研究的算法，用于在静态资源集上映射具有不同截止日期的多个工作流的任务。主要目的是将任务映射到处理器，最大限度地提高资源利用效率，同时满足所有工作流的最后期限。

#### 算法比较以及未来提升

| 算法                                                         | 目的                                         | 优点                                                         | 未来提升方向                                       |
| ------------------------------------------------------------ | -------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| Monte Carlo based DAG scheduling  approach                   | 最大限度缩短完工时间                         | 避免了随机变量的复杂计算，适用于人任何随机分布               | 动态调度                                           |
| Clustering based HEFT with duplication                       | 最小化执行时间                               | 最大限度的提高处理器利用率和复杂平衡                         | 更好的优化方法                                     |
| P-HEFT                                                       | 尽可能缩短完成时间                           | 高效和最佳完成时间                                           | 多用户环境的优化                                   |
| High performance and energy  efficient task scheduling algorithm | 最小化完成时间                               | 调度长度和能量消耗                                           | 大规模任务图                                       |
| Constrained  Earliest Finish Time                            | 最小化执行时间                               | 明细表长度的最小阈值                                         | 用于在机器的后续操作中选择多条受约束的关键路径     |
| Sorted Nodes in Leveled DAG Division                         | 加速、效率、复杂性和质量                     | 处理器的最小完成时间和最大利用率                             | 在异构系统上调度更多作业                           |
| Multi Queue Balancing Algorithm                              | 最大限度地缩短完成时间和最大限度地利用资源   | 处理器的最小完成时间和最大利用率                             | 在异构系统上调度更多作业                           |
| Heuristic and Dynamic programming scheduling strategy        | 尽量减少执行时间                             | 高度并行而不丧失通用性                                       | 热力性能                                           |
| SD-Based Algorithm for Task Scheduling                       | 时间表长度和速度                             | 减少执行时间并分配任务优先级。                               | 同质系统                                           |
| Online Scheduling of Dynamic Task Graphs                     | 使制造跨度最小化                             | 固定通道数的处理器间通信                                     | 离线                                               |
| Node duplication Modified Genetic Algorithm Approach         | 使完成时间和吞吐量最小化                     | 复制任务以减少总时间                                         | 非确定性同质系统                                   |
| Heuristic based for genetic algorithm                        | 使制造跨度最小化                             | 通过选择合格的处理器来分配任务的底层方法                     | 非确定性同质系统                                   |
| List based heuristic task Scheduling                         | 执行时间、通信延迟和最大化资源利用率         | 并行架构的效率。任务执行时间、通信成本和任务相关性在执行前可用。 | 动态调度                                           |
| An Efficient Genetic Algorithm                               | 调度长度、加速和效率。                       |                                                              | 异构处理器的部分连接网络。                         |
| BNP Scheduling Algorithms                                    | 尽可能缩短完成时间                           | 在性能上增加任务和处理器的数量                               | 异构系统                                           |
| Predict Earliest Finish Time                                 | 使制造跨度最小化                             | 调度长度比率、效率和频率                                     | 异构系统                                           |
| A Hybrid Genetic Scheduling Algorithm                        | 将任务分配给可用的处理器并生成跨度           | 避免过多的沟通时间。通信成本长，更有效，图形结构更灵活       | 更好的优化方法                                     |
| Fork-Join Method                                             | 以最小化总执行时间                           | 调度长度、加速比、效率和负载平衡                             | 更多节点数。                                       |
| Non-dominated sorting Genetic Algorithm-II                   | 使完工时间和可靠性成本最小化                 | 用于在最短时间内选择最佳计划                                 | 动态调度                                           |
| Task Scheduling & Energy Conservation Techniques             | 减少总完工时间和能源消耗                     | 高性能计算系统用途广泛，性能价格低廉，能耗巨大。             | 异构系统是一组关于不同任务图特征的随机和随机集合。 |
| Critical path scheduling with t-level                        | 最小化总体执行时间和最大完工时间             | 任务按其优先级的非递增顺序排序和选择                         | 同质系统                                           |
| Multi-core Scheduling                                        | 尽量缩短响应时间，提高资源利用率             | 它使用调度机制来调度不同的子任务和迁移策略，以减少响应时间   | 同质系统                                           |
| Task Scheduling Algorithm using Merge Conditions             | 要最小化时间表长度                           | 减少合并的次数。                                             | 预处理调度方法                                     |
| Scheduling in Heterogeneous Computing Environments for Proximity Queries | 在异构计算系统中最小化最大完工时间和邻近计算 | 鲁棒性强，专为并行邻近计算而设计                             | 针对更多种类作业的最佳优化方法                     |
| Comparative study of scheduling algorithm                    | 最大限度地提高资源利用效率                   | 提高效率，按时完成任务。                                     | 动态和电子科学基础设施平台                         |

