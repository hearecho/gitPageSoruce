---
title: "Go垃圾收集"
date: 2022-03-04T17:04:41+08:00
draft: false
description: "go垃圾收集的过程"
---

## Go 垃圾收集

Go垃圾收集使用的标记清除的方式，并且是并行GC，即和用户程序是一起运行的。标记方式使用的则是三色收集法。

### 垃圾收集步骤

Go垃圾收集总共是分为四部分：

1. Sweep Termination: 对未清扫的span进行清扫, 只有上一轮的GC的清扫工作完成才可以开始新一轮的GC
2. Mark: 扫描所有根对象, 和根对象可以到达的所有对象, 标记它们不被回收
3. Mark Termination: 完成标记工作, 重新扫描部分根对象
4. Sweep: 按标记结果清扫span

在GC过程中会有两种后台任务(G), 一种是标记用的后台任务, 一种是清扫用的后台任务。清扫任务会在程序启动时就启动，但是在进入清扫阶段之后才被唤醒。其中在标记阶段开始的时候和标记阶段结束的时候需要停止用户程序即STW（Stop The World）。两次STW的作用如下：

1. 第一次STW会准备根对象的扫描, 启动写屏障(Write Barrier)和辅助GC(mutator assist).
2. 第二次STW会重新扫描部分根对象, 禁用写屏障(Write Barrier)和辅助GC(mutator assist).

### Gc的触发条件

源码给出的触发条件主要有三种，如果不是这三种则是一直进行GC：

1. gcTriggerHeap: 当前分配的内存达到一定值就触发GC
2. gcTriggerTime: 当一定时间没有执行过GC就触发GC
3. gcTriggerCycle: 要求启动新一轮的GC, 已启动则跳过, 手动触发GC的`runtime.GC()`会使用这个条件

go源代码如下：

```go
func (t gcTrigger) test() bool {
	if !memstats.enablegc || panicking != 0 || gcphase != _GCoff {
		return false
	}
	switch t.kind {
	case gcTriggerHeap:
		// Non-atomic access to heap_live for performance. If
		// we are going to trigger on this, this thread just
		// atomically wrote heap_live anyway and we'll see our
		// own write.
		return memstats.heap_live >= memstats.gc_trigger
	case gcTriggerTime:
		if gcpercent < 0 {
			return false
		}
		lastgc := int64(atomic.Load64(&memstats.last_gc_nanotime))
		return lastgc != 0 && t.now-lastgc > forcegcperiod
	case gcTriggerCycle:
		// t.n > work.cycles, but accounting for wraparound.
		return int32(t.n-work.cycles) > 0
	}
	return true
}
```

### 三色标记法

三色标记法就是通过堆内存中不同状态的下的对象进行标价来为后续的清除做准备。三色的含义如下：

1. 黑色: 对象在这次GC中已标记, 且这个对象包含的子对象也已标记
2. 灰色: 对象在这次GC中已标记, 但这个对象包含的子对象未标记
3. 白色: 对象在这次GC中未标记

三色标记的过程：

1. 先将Gc root标记为灰色，加入到灰色队列中
2. 从灰色队列中取出对象，将其标记为黑色，并将其直接引用对象标记为灰色放入到灰色队列中
3. 重复步骤2，直到没有灰色对象的存在，此时剩余的白色对象就是需要进行回收的对象。

Gc  root对象

- Fixed Roots: 特殊的扫描工作
  - fixedRootFinalizers: 扫描析构器队列
  - fixedRootFreeGStacks: 释放已中止的G的栈
- Flush Cache Roots: 释放mcache中的所有span, 要求STW
- Data Roots: 扫描可读写的全局变量
- BSS Roots: 扫描只读的全局变量
- Span Roots: 扫描各个span中特殊对象(析构器列表)
- Stack Roots: 扫描各个G的栈

### 写屏障

写屏障的存在是因为go的垃圾收集过程是和用户程序并行的。并行的过程就会产生一些对象依赖上的变化，造成错误回收。例如开始扫描的时候发现根对象A和B，B拥有C的指针，GC先扫描A, 然后B把C的指针交给A, GC再扫描B, 这时C就不会被扫描到。写屏障就是为了避免这种类型的问题而存在的。

**启用了写屏障(Write Barrier)后, 当B把C的指针交给A时, GC会认为在这一轮的扫描中C的指针是存活的,
即使A可能会在稍后丢掉C, 那么C就在下一轮回收.**

也就是说，写屏障启动之后，用户程序的改动对于GC将不会造成任何影响，相互独立。写屏障只针对指针启用, 而且只在GC的标记阶段启用, 平时会直接把值写入到目标地址.

混合写屏障会同时标记指针写入目标的"原指针"和“新指针"。标记原指针的原因是,，其他运行中的线程有可能会同时把这个指针的值复制到寄存器或者栈上的本地变量,因为**复制指针到寄存器或者栈上的本地变量不会经过写屏障**, 所以有可能会导致指针不被标记，记新指针的原因是, 其他运行中的线程有可能会转移指针的位置。混合写屏障可以让GC在并行标记结束后不需要重新扫描各个G的堆栈, 可以减少Mark Termination中的STW时间。如果只是单独的写屏障，需要排除那些不经过写屏障的读写操作。

### 辅助GC

为了防止heap增速太快, 在GC执行的过程中如果同时运行的G分配了内存, 那么这个G会被要求辅助GC做一部分的工作.
在GC的过程中同时运行的G称为"mutator", "mutator assist"机制就是G辅助GC做一部分工作的机制.

辅助GC做的工作有两种类型, 一种是标记(Mark), 另一种是清扫(Sweep).
