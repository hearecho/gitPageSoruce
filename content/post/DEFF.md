---
title: "DEFF算法论文阅读"
date: 2021-08-22T15:19:32+08:00
draft: false
description: "Dynamic_scheduling_algorithm_for_parpllel_real_time_jobs_in_heterogeneous_system论文阅读"
tags:
- 论文阅读
- DAG
- 算法
- 动态调度
categories:
- 论文阅读
- 算法
- 动态调度
mathjax: true
---

# 论文介绍

《Dynamic_scheduling_algorithm_for_parpllel_real_time_jobs_in_heterogeneous_system》

> 异构系统中并行实时作业的动态任务调度仍然是一些研究人员正在研究的具有挑战性的问题。但是基于DAG的实时任务调度还没有得到足够的重视。提出了一种基于DAG的实时任务调度模型和一种时间复杂度较低的实时调度算法DEFF。仿真实验表明，该调度模型和调度算法是可行的，在中小型并行作业的情况下，该算法可以获得较高的调度成功率

### 模型符号

| 符号                              | 意义                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| $V$                               | 实时任务集合                                                 |
| $E$                               | 任务之间通信                                                 |
| $dl(v_i)$                         | 任务$v_i$的截至时间                                          |
| $cv_i$                            | 任务$v_i$的计算量                                            |
| $e_{i,j}=(v_i,v_j)$               | 表示任务$v_i,v_j$之间的通信量                                |
| $P$                               | 处理器集合                                                   |
| $p_i$                             | 拥有本地存储的处理器                                         |
| $C:V*P \rightarrow R$             | 表示不同的计算能力                                           |
| $w_k$                             | 表示处理器$p_k$在单位时间内的计算量                          |
| $cv_i/w_k$                        | 表示任务$v_i$在处理器$p_k$上的计算时间                       |
| $M:E * P * P \rightarrow R$       | 表示异构通信能力。                                           |
| $w_{km}$                          | 表示处理器$p_k,p_m$之间单位长度信息的传输时间                |
| $w_{km}*e_{i,j}$                  | 表示$e_{i,j}$的传输时间                                      |
| queue-Global  Job  Queue （GJQ）  | 全局作业队列，所有到达系统的任务首先都要进入这个队列中，然后再进入中心调度器。进入这个队列的是DAG任务 |
| queue-Task Dispatch  Queue  (TDQ) | 和中心调度器进行交互的，分解之后的dag子任务                  |
| Local Scheduling Queue (LSQ)      | 每个处理器拥有的本地任务队列                                 |
| $at(p_k)$                         | 处理器$p_k$的最早空闲时间                                    |
| $st_k(v_i)$                       | 实时任务$v_i$的最早开始时间                                  |
| $ft_k(v_i)$                       | 映射到处理器$p_k$实时任务$v_i$的最早完成时间                 |

#### 调度算法

##### 定义1

> 如果映射到处理器$p_k$实时任务$v_i$的最早完成时间$ft_k(v_i)$小于等于任务$v_i$的截至时间$dl(v_i)$，则实时任务$v_i$可以被调度到处理器$p_k$中。

##### 定义2

> 映射到处理器$p_k$实时任务$v_i$的最早完成时间$ft_k(v_i)$被定义为:$$ft_k(v_i) = cv_i/w_k + max(st_k(v_i),at(p_k))$$

##### 定义3

> 实时任务$v_i$的最早开始时间$st_k(v_i)$被定义为:$st_k(v_i)=max_{v_j\in pred(v_i)}( ft_m(v_j+w_{mk}*e_{i,j}))$;其中$pred(v_i)$表示任务$v_i$的前一个集合。

### 算法实现

#### deff调度算法

```python
"""
论文算法实现
deff
AST 本次deff调度算法启动的时间
AFT 上次deff调度算法结束的时间
DIFT 中间的插值  DIFT = AST-DIFT
"""
import time
import sys
def deff(AFT, processors, tasks_dtq, tasks_scheduled, w, e, cp_w, deL_dag_tasks):
    """
    :param deL_dag_tasks: 无法进行调度的dag任务
    :param cp_w: 处理器的计算能力一个数组
    :param e: 表示任务之间的通信量，一个字典  e[task_i][task_j]表示两个任务之间的通信量
    :param w: 表示处理器之间单位长度信息的传输时间  二维矩阵
    :param tasks_scheduled: 已经被调度过的任务
    :param tasks_dtq: dtq中的准备被调度的任务队列
    :param AFT: 上次deff调度算法结束的时间
    :param processors: 处理器数组，每个处理器元素都是一个字段储存处理器相关的信息
    :return:
    """
    DIFT = time.time() - AFT

    for p in processors:
        if p['at'] - DIFT < 0:
            p['at'] = 0
        else:
            p['at'] -= DIFT
    for v in tasks_scheduled:
        if v['ft'][v['mapped_p']['index']] - DIFT < 0:
            v['ft'][v['mapped_p']['index']] = 0
        else:
            v['ft'][v['mapped_p']['index']] = v['ft'][v['mapped_p']['index']] - DIFT
    while len(tasks_dtq) > 0:
        task = tasks_dtq[0]
        # 如果该子任务的其他任务已经存在不可调度的子任务 则直接舍弃
        if task['father'] in deL_dag_tasks:
            continue
        sps = []
        for i in range(len(processors)):
            if len(task['pred']) > 0:
                max_st = -1
                for pred_task in task['pred']:
                    temp = pred_task['ft'][pred_task['mapped_p']['index']] + w[pred_task['mapped_p']['index']][i] * e[task['name']][
                        pred_task['name']]
                    if temp > max_st:
                        max_st = temp
                task['st'][i] = max_st
            else:
                task['st'][i] = 0
            task['ft'][i] = task['c']/cp_w[i] + max(task['st'][i], processors[i]['at'])
            if task['ft'][i] <= task['dl']:
                sps.append(processors[i])
        if len(sps) > 0:
            min_p = None
            min_p_ft = sys.maxsize
            for p in sps:
                if task['ft'][p['index']] < min_p_ft:
                    min_p = p
                    min_p_ft = task['ft'][p['index']]
            task['mapped_p'] = min_p
            min_p['at'] = min_p_ft
        else:
            # 无法进行调度（也就是调度最后实时性不满足，直接舍弃）
            # 需要删除的是这个子任务所属dag任务的所有任务，但是不包括已经调度。
            deL_dag_tasks.append(task['father'])
            # 我的做法是不在这里删除，只是将这个任务所属的dag任务加入到一个内存中进行存储
            # 之后再进行再从gjq向tdq调度的时候进行判断 然后执行算法的时候进行判断不对他们进行调度即可
        tasks_dtq = tasks_dtq[1:]
    # 返回算法结束时间
    return time.time()
    pass
```

#### 触发器算法

```python
"""
论文调度器的触发算法实现
触发器的触发条件，一个新的dag任务图到达，或者是一个一个子任务完成则会调用触发器算法
调用触发器的时刻，当GJQ新到达一个dag任务或者是一个子任务完成了
GJQ 全局作业队列
TDQ 任务分发队列
"""
def trigger(completed_tasks, task, trigger_type, remain_tasks, TDQ, completed_job, GJQ):
    """
    :param GJQ: job队列
    :param completed_job: 完成的并行任务
    :param TDQ:
    :param remain_tasks: 剩余任务
    :param trigger_type: 0 新job到达  1 任务完成
    :param completed_tasks: 已经完成的子任务，字典格式，key为dag任务的图
    :param task: 完成的任务或者是新到的dag任务的开头任务，
    :return:
    """
    if trigger_type == 0:
        # 将所有的入口任务都加进去 也就是没有前继的任务
        for task in remain_tasks:
            if len(task['pred']) == 0:
                TDQ.append(task)
                # 从剩余任务中删除这个任务
                remain_tasks.remove(task)
    else:
        if len(remain_tasks) == 0:
            # 任务完成
            completed_job.append({task['father']: task['ft'][task['mapped_p']['index']]})
            return
        completed_tasks.append(task)
        for task in remain_tasks:
            if len(task['pred']) == 0:
                TDQ.append(task)
                # 从剩余任务中删除这个任务
                remain_tasks.remove(task)
            else:
                flag = True
                for pred_t in task['pred']:
                    if pred_t not in completed_tasks:
                        flag = False
                if flag:
                    TDQ.append(task)
                    # 从剩余任务中删除这个任务
                    remain_tasks.remove(task)
```

