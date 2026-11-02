# 附录 D libNUMA 简介

程序员为了最优地调度线程、恰当地分配内存等所需的许多信息虽然都是可获得的，但获取起来却十分繁琐。现有的 NUMA 支持库（libnuma，位于 RHEL/Fedora 系统上的 numactl 软件包中）远远不能提供足够的功能。

作为回应，作者提出了一个新库，它提供 NUMA 所需的全部功能。由于内存与缓存层次结构的处理存在重叠，这个库对于具有多线程与多核处理器的非 NUMA 系统同样有用——几乎当前可用的每台机器都属于此类。

要遵循本文件所给出的建议，就急需这个新库提供的功能，这才是此处提及它的唯一原因。该库（截至编写本文时）尚未完成、未经评审、未经打磨，也未（广泛）分发。它将来可能会有重大变化。它目前可从以下地址获得：

http://people.redhat.com/drepper/libNUMA.tar.bz2

本库的接口高度依赖于 /sys 文件系统所导出的信息。如果该文件系统未挂载，许多函数会直接失败或给出不准确的信息。如果进程在 chroot jail 中执行，记住这一点尤为重要。

该库的接口头文件目前包含以下定义：

```c
typedef memnode_set_t;

#define MEMNODE_ZERO_S(setsize, memnodesetp)
#define MEMNODE_SET_S(node, setsize, memnodesetp)
#define MEMNODE_CLR_S(node, setsize, memnodesetp)
#define MEMNODE_ISSET_S(node, setsize, memnodesetp)
#define MEMNODE_COUNT_S(setsize, memnodesetp)

#define MEMNODE_EQUAL_S(setsize, memnodesetp1, memnodesetp2)

#define MEMNODE_AND_S(setsize, destset, srcset1, srcset2)
#define MEMNODE_OR_S(setsize, destset, srcset1, srcset2)
#define MEMNODE_XOR_S(setsize, destset, srcset1, srcset2)

#define MEMNODE_ALLOC_SIZE(count)
#define MEMNODE_ALLOC(count)
#define MEMNODE_FREE(memnodeset)

int NUMA_cpu_system_count(void);
int NUMA_cpu_system_mask(size_t destsize, cpu_set_t *dest);

int NUMA_cpu_self_count(void);
int NUMA_cpu_self_mask(size_t destsize, cpu_set_t *dest);

int NUMA_cpu_self_current_idx(void);
int NUMA_cpu_self_current_mask(size_t destsize, cpu_set_t *dest);

ssize_t NUMA_cpu_level_mask(size_t destsize, cpu_set_t *dest,
                            size_t srcsize, const cpu_set_t *src,
                            unsigned int level);

int NUMA_memnode_system_count(void);
int NUMA_memnode_system_mask(size_t destsize, memnode_set_t *dest);

int NUMA_memnode_self_mask(size_t destsize, memnode_set_t *dest);

int NUMA_memnode_self_current_idx(void);
int NUMA_memnode_self_current_mask(size_t destsize, memnode_set_t *dest);

int NUMA_cpu_to_memnode(size_t cpusetsize, const cpu_set_t *cpuset,
                        size_t __memnodesize, memnode_set_t *memnodeset);
int NUMA_memnode_to_cpu(size_t memnodesize, const memnode_set_t *memnodeset,
                        size_t cpusetsize, cpu_set_t *cpuset);

int NUMA_mem_get_node_idx(void *addr);
int NUMA_mem_get_node_mask(void *addr, size_t size,
                           size_t destsize, memnode_set_t *dest);
```

MEMNODE_* 宏在形式与功能上与第 6.4.3 节引入的 CPU_* 宏相似。这些宏没有非 _S 版本，它们全都需要一个大小参数。memnode_set_t 类型相当于 cpu_set_t，只不过这次针对的是内存节点。注意，内存节点的数量与 CPU 的数量不必有任何关系，反之亦然。每个内存节点上可以有许多 CPU，甚至可以一个 CPU 都没有。因此，动态分配的内存节点位集，其大小不应由 CPU 数量来决定。

因此，应改用 NUMA_memnode_system_count 接口。它返回当前已注册的节点数量。该数字可能随时间增大或缩小。不过多数情况下它会保持不变，因此很适合用来确定内存节点位集的大小。分配同样与 CPU_ 宏类似，通过 MEMNODE_ALLOC_SIZE、MEMNODE_ALLOC 和 MEMNODE_FREE 完成。

作为与 CPU_* 宏的最后一处对应，该库还提供了用于比较内存节点位集是否相等以及执行逻辑运算的宏。

NUMA_cpu_* 函数提供处理 CPU 集合的功能。其中一部分接口只是把现有功能以新名字提供出来。NUMA_cpu_system_count 返回系统中 CPU 的数量，NUMA_CPU_system_mask 变体则返回一个设置了相应位的位掩码——这一功能在其他地方无法获得。

NUMA_cpu_self_count 与 NUMA_cpu_self_mask 返回关于当前线程此刻被允许在其上运行的 CPU 的信息。NUMA_cpu_self_current_idx 返回当前所用 CPU 的索引。由于内核可能做出调度决策，这一信息在返回时可能已经过时；必须始终假定它是不准确的。NUMA_cpu_self_current_mask 返回同样的信息，并在位集中设置相应的位。

NUMA_memnode_system_count 已经介绍过了。NUMA_memnode_system_mask 是对应的函数，它填充一个位集。NUMA_memnode_self_mask 则根据直接连接到该线程当前可运行的任一 CPU 的内存节点来填充位集。

NUMA_memnode_self_current_idx 与 NUMA_memnode_self_current_mask 函数返回更加专门化的信息。返回的信息是与该线程当前运行所在的处理器相连的内存节点。正如 NUMA_cpu_self_current_* 函数那样，这一信息在函数返回时可能已经过时；它只能用作提示。

NUMA_cpu_to_memnode 函数可用于把一组 CPU 映射到直接相连的内存节点集合。如果 CPU 集合中只设置了一位，就可以确定每个 CPU 属于哪个内存节点。目前 Linux 不支持单个 CPU 属于一个以上的内存节点；理论上将来这一点可能会改变。要朝相反方向映射，可以使用 NUMA_memnode_to_cpu 函数。

如果内存已经分配，有时知道它分配在哪里会很有用。这正是 NUMA_mem_get_node_idx 与 NUMA_mem_get_node_mask 让程序员能够确定的事情。前一个函数返回与参数所指定地址相对应的页面所在内存节点的索引——如果该页面尚未分配，则返回按照当前生效的策略它将分配到的内存节点。第二个函数可以对整个地址范围做同样的工作；它以位集的形式返回信息。该函数的返回值是被使用的不同内存节点的数量。

在本节的剩余部分，我们将看到这些接口用例的几个示例。所有示例都省略了错误处理，以及 CPU 和/或内存节点数量超出 cpu_set_t 与 memnode_set_t 类型各自容量的情形。使代码变得健壮留给读者作为练习。

## 确定给定 CPU 的线程兄弟

为了调度辅助线程，或其他受益于被调度到某个给定 CPU 的线程兄弟上的线程，可以使用如下代码序列。

```c
cpu_set_t cur;
CPU_ZERO(&cur);
CPU_SET(cpunr, &cur);
cpu_set_t hyperths;
NUMA_cpu_level_mask(sizeof(hyperths), &hyperths, sizeof(cur), &cur, 1);
CPU_CLR(cpunr, &hyperths);
```

这段代码首先为 cpunr 指定的 CPU 生成一个位集。然后把这个位集连同第五个参数一起传给 NUMA_cpu_level_mask，该参数指明我们要找的是超线程。结果返回到 hyperths 位集中。剩下要做的事情只是清除与原 CPU 对应的位。

## 确定给定 CPU 的核心兄弟

如果两个线程不应被调度到两个超线程上，但又能从缓存共享中获益，我们就需要确定该处理器的其他核心。下面的代码序列可以做到这一点。

```c
cpu_set_t cur;
CPU_ZERO(&cur);
CPU_SET(cpunr, &cur);
cpu_set_t hyperths;
int nhts = NUMA_cpu_level_mask(sizeof(hyperths), &hyperths, sizeof(cur), &cur, 1);
cpu_set_t coreths;
int ncs = NUMA_cpu_level_mask(sizeof(coreths), &coreths, sizeof(cur), &cur, 2);
CPU_XOR(&coreths, &coreths, &hyperths);
ncs -= nhts;
```

代码的第一部分与确定超线程的代码相同。这并非巧合，因为我们必须把给定 CPU 的超线程与其他核心区分开。第二部分实现了这一点：它再次调用 NUMA_cpu_level_mask，但这次 level 为 2。剩下要做的事情是把给定 CPU 的所有超线程从结果中移除。变量 nhts 与 ncs 用于记录相应位集中置位的位数。

得到的掩码可用于调度另一个线程。如果不需要显式调度其他线程，可以把使用哪个核心的决定留给操作系统。否则可以迭代地运行以下代码：

```c
while (ncs > 0) {
  size_t idx = 0;
  while (! CPU_ISSET(idx, &ncs))
    ++idx;
  CPU_ZERO(&cur);
  CPU_SET(idx, &cur);
  nhts = NUMA_cpu_level_mask(sizeof(hyperths), &hyperths, sizeof(cur), &cur, 1);
  CPU_XOR(&coreths, &coreths, hyperths);
  ncs -= nhts;

  ... schedule thread on CPU idx ...
}
```

该循环在每次迭代中从剩余的核心中挑选一个 CPU 编号。然后计算这个 CPU 的所有超线程。接着把得到的位集从可用核心位集中减去（使用 CPU_XOR）。如果 XOR 运算没有移除任何位，那就真出问题了。ncs 变量随之更新，我们就可以进入下一轮——但要先做出调度决策。最后，idx、cur 或 hyperths 中任何一个都可以用来调度线程，取决于程序的要求。通常最好是给操作系统尽可能多的自由度，因此使用 hyperths 位集，让操作系统去挑选最好的超线程。
