---
title: "HEFT算法 静态调度"
date: 2021-08-31T15:33:49+08:00
draft: false
description: "经典heft调度算法阅读"
tags:
- 论文阅读
- DAG
- 算法
- 静态调度
categories:
- 论文阅读
- 算法
- 静态调度
mathjax: true
---

# 论文

《[Performance-effective and low-complexity task scheduling for heterogeneous computing](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=993206) 》

> 本文是异构平台的两种静态调度方法，分别是heft和cpop算法。

### 模型符号

| 符号                                                         | 意义                           |
| ------------------------------------------------------------ | ------------------------------ |
| $\overline {w_i}$                                            | 任务i的平均执行时间            |
| $w_{i,j}$                                                    | 任务i在处理器j上的执行时间     |
| $L$                                                          | 处理器通信模块启动时间         |
| $L_m$                                                        | 处理器m的通信模块启动时间      |
| $c_{i,k} = L_m + \frac{data_{i,k}}{B_{m,n}}$                 | 任务i和任务j的通信时间         |
| $B_{m,n}$                                                    | 处理器m，n的单位时间通信量     |
| $data_{i,k}$                                                 | 任务i和任务j的通信量           |
| $EST(n_i,p_j) = max\{avail[j], max_{n_m\in pred(n_i)} (AFT(n_m)+c_{m,j})\}$ | 任务i在处理器j上的最早启动时间 |
| $EFT(n_i,p_j) = w_{i,j} + EST(n_i,p_j)$                      | 任务i在处理器j上的最早完成时间 |
| $avail[j]$                                                   | 处理器j最早可用时间            |
| $makespan = max\{AFT(n_{exit})\}$                            | dag的工作完成时间              |
| $rank_u(n_i) = \overline {w_i} + max_{n_j \in succ(n_i)}(\overline {c_{i,j}}+rank_u(n_j)) $ | 作为权重                       |
| $rank_d(n_i) = max_{n_j \in pred(ni)} \{rankd(n_j) + \overline {w_j} + \overline {c_{j,i}}\}$ | 作为权重                       |

### 算法

#### 两个重要指标

##### $rank_{u}$

> $rank_{u}$是从任务ni到出口节点关键路径的长度，是包括任务ni的计算耗时的。对于出口节点$rank_{u}$的值等于它的平均执行时间。

##### $rank_{d}$

> $rank_{d}$是从入口节点到任务ni的关键路径长度，不包括任务ni的执行时间，对于入口节点其$rank_{d}$的值等于0。

#### heft算法

> heft算法主要有两个阶段，第一个阶段计算所有任务的权重优先级，第二个阶段按照优先级顺序调度任务到最适合他们的处理器，以达到最小化任务的完成时间。

##### 计算任务优先级

> heft算法会将任务的优先度设置为$rank_u$，之后按照$rank_u$降低的顺序排列生成任务执行序列，如果两个任务的$rank_u$是相同的则可以有很多打破这种局面的策略，比如选择直接后继任务的$rank_u$更大的任务。最后生成的序列是一个拓扑排序所以肯定满足前序约束。

##### 处理器选择

> 对于大多数调度算法，处理器的最早启动时间都是在该处理完成最后一个被分配任务之后。但是heft算法是考虑一个插入策略，考虑将任务插入到一个最早的空间时间间隔在两个已经完成调度的任务。空闲时隙时间的长度，即，在同一处理器上连续调度的两个任务的执行开始时间和完成时间之间的差异，应该至少能够满足计算要调度的任务的成本。并且必须要满足前置要求。

##### 算法伪代码

```python
# set the computation costs of tasks and communication costs of edges with mean values
# Compute ranku for all tasks by traversing graph upward, starting from the exit task
# Sort tasks in a scheduling list by noincreasing order of ranku values
while there are unscheduled tasks in the list do:
    Select the first task ni from the list for scheduling
    for each processor pk in the processor-set (pk in Q) do:
        Comput EFT(ni,pk) value using the insertion-based scheduling policy
    Assign task ni to processor pj that minimizes EFT of task ni
endwhile
```

#### cpop算法

>cpop算法和heft算法结构相同都有两部分，不过使用的算法不相同。使用不同的属性设置任务的优先级以及不同的策略来选择最佳的处理器。

##### 计算任务优先级

> 对于cpop算法需要将每个任务的$rank_u,rank_d$使用平均执行时间和平均通信时间进行计算。cpop算法使用dag的关键路径，路径的长度是沿着这条路径计算时间以及任务之间通信时间的总和。这条路径上所有计算时间总和是调度长度的下限（所有的任务都在同一个处理器上进行处理）。而每个任务的优先度是$rank_u,rank_d$的总和。入口的优先度等于关键路径的长度。对于平等的情况，选择第一个直系后继有更高的优先级的任务。具体选择在伪代码部分给出。

##### 处理器选择

> 选择一个处理器作为关键路径专用处理器，这个处理器能够最小化执行时间。如果一个任务在关键路径上，直接调度到关键路径处理器。除此之外选择能够让该任务执行时间最短的其他处理器，注意不能调度到关键路径专用处理器。

##### 算法伪代码

```python
# set the computation costs of tasks and communication costs of edges with mean values
# Compute ranku for all tasks by traversing graph upward, starting from the exit task
# Compute rankd for all tasks by traversing graph downward, starting from the entry task
# Compute priority(ni) = ranku(ni) + rankd(ni) for each task ni in the graph
# SET_CP = {n_entry}, where SET_CP is the set of tasks on the critical path
nk <- n_entry
while nk is not the exit task do:
    Select nj where ((nj in succ(nk)) and (priority(nj)== |CP|))
    SET_CP = SET_CP.append(nj)
    nk <- nj
endwhile
Select the critical path processor(p_cp) which minizes sum(w_i,j)
Initialize the priority queue whith entry task
while ther is an unscheduled task in the priority queue do:
    Select the highest priority task ni from priority queue
    if ni in SET_CP:
        Assign the task ni on p_cp
    else:
        Assign the task ni to processor pj which minimizes the EFT(ni,pj)
    Update the priority-queue with the sucessors of ni if they ready tasks
endwhile
```

