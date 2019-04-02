#lec11: 进程／线程概念spoc练习

## 视频相关思考题

### 11.1 进程的概念

1. 什么是程序？什么是进程？  
程序是一个静态的文件，本质上是由很多指令组成。而进程是执行中的程序，包括程序和执行状态，是操作系统对处于执行状态的程序的抽象。

2. 进程有哪些组成部分？  
进程包括程序、数据和进程控制块。

3. 请举例说明进程的独立性和制约性的含义。  
进程的独立性指不同进程之间互不影响。制约性指不同进程间因为访问共享资源或进程间同步而产生制约。

4. 程序和进程联系和区别是什么？  
程序是静态的、永久的，进程是动态的、暂时的。进程等于程序加执行状态。

### 11.2 进程控制块

1. 进程控制块的功能是什么？  
进程控制块为操作系统提供管理控制进程运行的信息。

2. 进程控制块中包括什么信息？  
包括进程标识信息、处理机现场保存、进程控制信息。

3. ucore的进展控制块数据结构定义中哪些字段？有什么作用？  
```
struct proc_struct {
    enum proc_state state;                      // Process state
    int pid;                                    // Process ID
    int runs;                                   // the running times of Proces
    uintptr_t kstack;                           // Process kernel stack
    volatile bool need_resched;                 // bool value: need to be rescheduled to release CPU?
    struct proc_struct *parent;                 // the parent process
    struct mm_struct *mm;                       // Process's memory management field
    struct context context;                     // Switch here to run process
    struct trapframe *tf;                       // Trap frame for current interrupt
    uintptr_t cr3;                              // CR3 register: the base addr of Page Directroy Table(PDT)
    uint32_t flags;                             // Process flag
    char name[PROC_NAME_LEN + 1];               // Process name
    list_entry_t list_link;                     // Process link list 
    list_entry_t hash_link;                     // Process hash list
};
```
  
### 11.3 进程状态

1. 进程生命周期中的相关事件有些什么？它们对应的进程状态变化是什么？  
进程创建：从创建到就绪。  
进程执行：从就绪到运行。  
进程等待：从运行到等待。  
进程抢占：从运行到就绪。  
进程唤醒：从等待到就绪。  
进程结束：从运行到结束。

### 11.4 三状态进程模型

1. 运行、就绪和等待三种状态的含义？7个状态转换事件的触发条件是什么？  
NULL到创建：一个新进程被产生出来执行一个程序  
创建到就绪：当进程被创建完成并初始化后，一切就绪准备运行时，变为就绪状态  
就绪到运行：处于就绪状态的进程被进程调度程序选中后，就分配到处理机上来运行  
运行到结束：当进程表示它已经完成或者因出错，当前运行进程会由操作系统作结束处理  
运行到就绪：处于运行状态的进程在其运行过程中，由于分配给它的处理机时间片用完而让出处理机  
运行到等待：当进程请求某资源且必须等待时  
等待到就绪：当进程要等待某事件到来时，它从阻塞状态变到就绪状态

### 11.5 挂起进程模型

1. 引入挂起状态的目的是什么？  
处在挂起状态的进程映像在磁盘上， 目的是减少进程占用内存
2. 引入挂起状态后，状态转换事件和触发条件有什么变化？  
增加了就绪挂起状态和等待挂起状态。  
等待到等待挂起：没有进程处于就绪状态或就绪进程要求更多内存资源  
就绪到就绪挂起：当有高优先级等待（系统认为会很快就绪的）进程和低优先级就绪进程     
运行到就绪挂起：对抢先式分时系统，当有高优先级等待挂起进程因事件出现而进入就绪挂起  
等待挂起到就绪挂起：当有等待挂起进程因相关事件出现  
就绪挂起到就绪：没有就绪进程或挂起就绪进程优先级高于就绪进程  
等待挂起到等待：当一个进程释放足够内存，并有高优先级等待挂起进程  

3. 内存中的什么内容放到外存中，就算是挂起状态？  
进程PCB
### 11.6 线程的概念

1. 引入线程的目的是什么？  
解决进程间共享数据困难以及进程切换开销较大的问题。  
2. 什么是线程？  
线程是进程的一部分，描述指令流执行状态。它是进程中的指令执行流的最小单元，是CPU调度的基本单位。

3. 进程与线程的联系和区别是什么？  
资源分配单位，线程是CPU调度单位。  
线程具有就绪、等待和运行三种基本状态和状态间的转换关系。  
进程拥有一个完整的资源平台，而线程只独享指令流执行的必要资源，如寄存器和栈。
### 11.7 用户线程

1. 什么是用户线程？  
由一组用户级的线程库函数来完成线程的管理，包括线程的创建、终止、同步和调度等。
2. 用户线程的线程控制块保存在用户地址空间还是在内核地址空间？  
用户地址空间。

### 11.8 内核线程

1. 用户线程与内核线程的区别是什么？  
实现方式：用户线程由一组用户级的线程库函数来完成线程的管理，包括线程的创建、终止、同步和调度等；而内核线程由内核通过系统调用实现的线程机制，由内核完成线程的创建、终止和管理。  
内核线程执行系统调用而被阻塞不影响其他线程，用户线程执行系统调用整个进程都会被阻塞。同一进程内的用户线程切换速度快，而内核线程切换速度慢。  
用户线程只能以进程为单位分配CPU时间，而内核线程可以以线程为单位来分配。  


2. 同一进程内的不同线程可以共用一个相同的内核栈吗？  
可以
3. 同一进程内的不同线程可以共用一个相同的用户栈吗？  
可以

## 选做题
1. 请尝试描述用户线程堆栈的可能维护方法。

## 小组思考题
(1) 熟悉和理解下面的简化进程管理系统中的进程状态变化情况。
 - [简化的三状态进程管理子系统使用帮助](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep7-process-run.md)
 - [简化的三状态进程管理子系统实现脚本](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep7-process-run.py)

(2) (spoc)设计一个简化的进程管理子系统，可以管理并调度如下简化进程。在理解[参考代码](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab4/process-concept-homework.py)的基础上，完成＂YOUR CODE"部分的内容。然后通过测试用例和比较自己的实现与往届同学的结果，评价自己的实现是否正确。可２个人一组。

### 进程的状态 

 - RUNNING - 进程正在使用CPU
 - READY   - 进程可使用CPU
 - DONE    - 进程结束

### 进程的行为
 - 使用CPU, 
 - 发出YIELD请求,放弃使用CPU


### 进程调度
 - 使用FIFO/FCFS：先来先服务,
   - 先查找位于proc_info队列的curr_proc元素(当前进程)之后的进程(curr_proc+1..end)是否处于READY态，
   - 再查找位于proc_info队列的curr_proc元素(当前进程)之前的进程(begin..curr_proc-1)是否处于READY态
   - 如都没有，继续执行curr_proc直到结束

### 关键模拟变量
 - 进程控制块
```
PROC_CODE = 'code_'
PROC_PC = 'pc_'
PROC_ID = 'pid_'
PROC_STATE = 'proc_state_'
```
 - 当前进程 curr_proc 
 - 进程列表：proc_info是就绪进程的队列（list），
 - 在命令行（如下所示）需要说明每进程的行为特征：（１）使用CPU ;(2)等待I/O
```
   -l PROCESS_LIST, --processlist= X1:Y1,X2:Y2,...
   X 是进程的执行指令数; 
   Ｙ是执行CPU的比例(0..100) ，如果是100，表示不会发出yield操作
```
 - 进程切换行为：系统决定何时(when)切换进程:进程结束或进程发出yield请求

### 进程执行
```
instruction_to_execute = self.proc_info[self.curr_proc][PROC_CODE].pop(0)
```

### 关键函数
 - 系统执行过程：run
 - 执行状态切换函数:　move_to_ready/running/done　
 - 调度函数：next_proc

### 执行实例

#### 例１
```
$./process-simulation.py -l 5:50
Process 0
  yld
  yld
  cpu
  cpu
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD
Time     PID: 0 
  1     RUN:yld 
  2     RUN:yld 
  3     RUN:cpu 
  4     RUN:cpu 
  5     RUN:yld 

```

   
#### 例２
```
$./process-simulation.py  -l 5:50,5:50
Produce a trace of what would happen when you run these processes:
Process 0
  yld
  yld
  cpu
  cpu
  yld

Process 1
  cpu
  yld
  cpu
  cpu
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD
Time     PID: 0     PID: 1 
  1     RUN:yld      READY 
  2       READY    RUN:cpu 
  3       READY    RUN:yld 
  4     RUN:yld      READY 
  5       READY    RUN:cpu 
  6       READY    RUN:cpu 
  7       READY    RUN:yld 
  8     RUN:cpu      READY 
  9     RUN:cpu      READY 
 10     RUN:yld      READY 
 11     RUNNING       DONE 
```
