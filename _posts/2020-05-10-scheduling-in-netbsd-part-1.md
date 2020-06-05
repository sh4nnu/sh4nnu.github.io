---
title: "Scheduling in NetBSD - Part 1"
date: "2020-05-10"
layout: post
summary: 
author: Shannu
category: 
    - BSD
    - Scheduling
    - Unix
thumbnail: /assets/img/posts/netbsd-1.png

---

## INTRODUCTION

In this blog, we will discuss about the **4.4BSD Thread scheduler** one of the two schedulers in NetBSD and a few OS APIs that can be used to control the schedulers and get information while executing. 

## Few basics

### What is a scheduler? 

A Process Scheduler schedules the processes of different states namely ready, waiting, and running. There are different scheduling algorithms like :

1. Shortest Job first
2. First Come First Serve
3. Shortest Remaining time first
4. Priority scheduling
5. Round robin 
6. Multi-level queue scheduling etc.,

These algorithms are either **non-preemptive** or **preemptive**. In preemptive scheduling is based on priority where a scheduler may preempt a low priority running process anytime when a high priority process enters into a ready state, whereas in non-preemptive scheduling once a process enters running state it cannot be preempted until it completes its allotted time.

## NetBSD's internals

Before going into the schedulers in NetBSD lets learn about few internals of NetBSD.

### Processes:

NetBSD supports traditional processes created by _fork()_ and native POSIX threads (_pthreads_). The threads are implemented with a **1:1 threading model** where each user thread (_pthread_) has a kernel thread called a light-weight process (_LWP_). So, Inside the kernel, both processes and threads are implemented as LWPs and are served the same by the scheduler. 

### Scheduling classes:

According to the POSIX standard, at least the following three scheduling policies (classes) should be provided to support the POSIX real-time scheduling extensions: 

-  **SCHED OTHER**: Time-sharing (TS) scheduling policy, the default policy in NetBSD.
-  **SCHED FIFO**: First in, first-out (FIFO) scheduling policy. \*
-  **SCHED RR**: Round-robin scheduling policy.  \*

(\* - strictly provided by the standard.)

### CSF:

CFS is  a  Common Scheduler Framework which was introduced in NetBSD 5.0. It provides an Interface for implementing different thread scheduling algorithms and one among those schedulers can be selected at compile-time.  Currently, there are 2 schedulers available, they are

- **4.4BSD thread scheduler**: The traditional BSD's scheduler.
-  **M2 scheduler**: A more advanced thread scheduler.

## Controlling the scheduling of processes and threads

NetBSD provides utilities to control and change scheduling policies and [thread affinity](https://computing.llnl.gov/tutorials/openMP/ProcessThreadAffinity.pdf). We are going to see about **schedctl** and **cpuctl** in detail.

### **Schedctl** 

The schedctl command can be used to control the scheduling of processes and threads.  It also returns information about the current scheduling parameters of the process or thread. This enables us to monitor and control scheduling in NetBSD, including changing the priority , affinity, scheduler’s - policy. Etc,  
  
1\. We can see the scheduling information of a process by it's PID.
{% highlight bash %}
#schedctl -p 18467
  LID:              1
  Priority:         43
  Class:            SCHED_OTHER
  Affinity (CPUs):  <none>

{% endhighlight %}
The '**Class**' in the above prompt refers to the scheduling policy the process is being scheduled with. We have 3 scheduling policies  SCHED_OTHER, SCHED_FIFO,SCHED_RR   

2\. We can change the scheduling policy for a process or a command.
{% highlight bash %}
    # schedctl -C SCHED_FIFO top
{% endhighlight %}

And the `top` runs with FIFO policy whenever called.

### **Cpuctl** 

This can be used to control the CPU of the Computer. We can perform several operations like sending a cpu offline and bringing it back online, setting microcode and listing available cpus. Let's see some examples.

1\. This argument `list` can be used to see the available CPUs and their status. 
{% highlight bash %}
$ cpuctl list
Num  HwId Unbound LWPs Interrupts Last change               Intr
---- ---- ------------ ---------- ------------------------ --------
0    0    online       intr       Tue Apr  7 23:17:32 2020 8
1    1    online       intr       Tue Apr  7 23:17:32 2020 0
{% endhighlight %}
2\. We can see specific information regarding a cpu. It provides a lot of information, only a few interesting stats are mentioned here. 

{% highlight bash %}
$ cpuctl identify 0

cpu0: highest basic info 00000016
cpu0: highest hypervisor info 00000000
cpu0: highest extended info 80000008
cpu0: Running on hypervisor: unknown
cpu0: "Intel(R) Core(TM) i5-8259U CPU @ 2.30GHz"
cpu0: Intel 7th or 8th gen Core (Kaby Lake, Coffee Lake) or Xeon E (Coffee Lake) (686-class), 2304.00 MHz
cpu0: L2 cache 256KB 64B/line 4-way
cpu0: L3 cache 6MB 64B/line 12-way
cpu0: 64B prefetching
cpu0: L1 1GB page DTLB 4 1GB entries 4-way
{% endhighlight %}

3\. We can send specific CPUs offline and online.
{% highlight bash %}

$ cpuctl offline 1

$ cpuctl list
Num  HwId Unbound LWPs Interrupts Last change               Intr
---- ---- ------------ ---------- ------------------------ -------
0    0    online       intr       Tue Apr  7 23:17:32 2020 8
1    1    offline      intr       Wed Apr  8 03:27:12 2020 0

$ cpuctl online 1

$ cpuctl list
Num  HwId Unbound LWPs Interrupts Last change               Intr
---- ---- ------------ ---------- ------------------------ -------
0    0    online       intr       Tue Apr  7 23:17:32 2020 8
1    1    online       intr       Wed Apr  8 03:27:59 2020 0
{% endhighlight %}

In addition to these we can change affinity of a process and many other commands are available that can come handy to examine the scheduling policies and study threads and processes in NetBSD.

### Process structure

As discussed before threads and processes are treated the same as LWPs by the scheduler. So the BSD associates a process ID with each thread, rather than with a collection of threads sharing an address space. For more detailed information regarding Process Structure please go through the book 'The Design and Implementation of 4.4BSD Operating System'.

## WORKING OF BSD SCHEDULER  

### 4.4BSD scheduler

The traditional **4.4BSD scheduler** employs a **multi-level feedback queues** algorithm, favoring interactive, short-running threads to CPU-bound ones. 

![](/assets/img/posts/slide-1.png){:class="img-fluid"}
src: https://images.slideplayer.com/16/5138416/slides/slide\_1.jpg

### **Multilevel feedback queue:**

The multi-level feedback queue is an excellent example of a system that learns from the past to predict the future. This type of scheduler maintains several queues of ready-to-run threads, where each queue holds threads with a different priority as shown in the figure. At any given time, the scheduler chooses a thread from the highest-priority non-empty queue. If the highest-priority queue contains multiple threads, then they run in the **Round-robin** order. The time quantum used in 4.4BSD is **0.1 seconds**. 

### **Niceness**

Each thread has an integer nice value, that determines how nice a thread is treated than other threads. A nice of zero does not affect thread priority. A positive nice, to the maximum of 20, decreases the priority of a thread and causes it to give up some CPU time it would otherwise receive. On the other hand, a negative nice, to the minimum of -20, tends to take away CPU time from other threads.

### **Priority**

CPU time is made available to processes according to their scheduling priority. A process has two scheduling priorities, one for scheduling **user-mode** execution and one for scheduling **kernel-mode** execution. The _p\_usrpri_ field in the process structure contains the user-mode scheduling priority, whereas the _p\_priority_ field holds the current kernel-mode scheduling priority.  Priorities are divided as shown in the figure in the range between 0 and 127, with a lower value interpreted as a higher priority.

![](https://lh3.googleusercontent.com/VDh0Z0lvz7iPw2icPLr_QIN42w0PEMNtZI3T9vyQ8XcNWL7n8O3N6U7Yhz2vRmtst6guNJ69pQyEQfdmjP4qYyK6HQydtbeD49vfLIqZT4siF8TFvskFOc0yGN4fUYMTRys-ib5F){:class="img-fluid"}

Priority distribution

### Calculating priority

As we have seen how priorities are divided let's get into how actually are they calculated. The processes' priority is determined by two values:

1. [_p\_estcpu_](https://nxr.netbsd.org/xref/src/sys/sys/proc.h#283)  : Time average value of CPU ticks (_p\_cpticks_)
2. [_p\_nice_](https://nxr.netbsd.org/xref/src/sys/sys/proc.h#325)     :  Process "nice" value with range of \[ -20 to +20 \]  

A process's _user-mode_ scheduling priority is calculated every four clock cycles (~= 40ms) using the following equation.

>_EQ1: p\_usrpri = PUSER + (  p\_estcpu  /  4 ) + 2 p\_nice_

Values less than PUSER are set to PUSER (see Table) values greater than 127 are set to 127. This calculation causes the priority to decrease linearly based on recent CPU utilization. The user-controllable _p\_nice_ parameter acts as a limited **weighting factor**. Negative values retard the effect of heavy CPU utilization by offsetting the additive term containing _p\_estcpu_. Otherwise, if we ignore the second term, _p\_nice_ simply shifts the priority by a constant factor. 

The _p\_estcpu_ is adjusted for every one second via a **digital decay filter** (which reduces the effect of previous values by mathematical operations like a decay). The decay causes about 90 percent of the CPU usage accumulated in a 1-second interval to be forgotten over a period of time that is dependent on the system load average. To be exact, p\_estcpu is adjusted according to EQ2 . Where the _load_ is a sampled average of the sum of the lengths of the run queue and of the short-term sleep queue over the previous 1-minute interval of system operation.  We will discuss about run queues below.

>_EQ2: p\_estcpu = ( 2 load ) / ( 2 load + 1 )  p\_estcpu + p\_nice_

The working of the decay filter is explained in detail in the book \[The Design and Implementation of the 4.4BSD Operating System\] but we are not going to cover it here.

### Assigning priorities

As discussed the runnable processes get their priorities adjusted periodically, but what about the processes that were blocked awaiting an event?

The system ignores such processes, so an estimate of their filtered **CPU-usage** is calculated, and the priority adjusted accordingly.  The system recomputes a process's priority when that process is awakened and has been sleeping for longer than 1 second.  This optimization can reduce the system’s overhead when many blocked processes are present. For this we use **_p\_slptime_**  and estimation of the process's filtered CPU-usage can  be done in one step with the help of this.

### What is **_p\_slptime_**

This is an estimate of the time a process has spent blocked waiting for an event. When a process calls sleep() this p\_slptime is set to 0, while the process stays in SSLEEP or SSTOP state it is incremented once per second. When the process is awakened, the system calculates the _p\_estcpu_  with EQ3

![](https://lh6.googleusercontent.com/xQor5LGQzyyYYNxsdVLSqE_A2l18_aysYkaE54FUqC4NJxVFI7vI1iTdgBDcTs4jnMM_he2Proaz0a6RFPo82D0Wi_Mt_mA6ruxQWjzhEXbIVJoNNugl3e80XK56puFfidFZ_F4q){:class="img-fluid"}

And then recalculates the scheduling priority using EQ1. This analysis ignores the influence of _p\_nice,_ and the _load_ used is current avg load rather than the load at the time the process was blocked.

### Process priority routines

![](https://lh6.googleusercontent.com/aay29BucTPSorBXybGfneMax9VWkJRGeqTPpE14I8ldUG66vqqewHz8SopB2Wk7MXTzY6se8HLC28hAzxgjLci1uowG25s46qO6cCSlEjAqgV2M8r6zMhLbDPFB6_NRXvj-KiHHr){:class="img-fluid"}

These are the procedural interface to the priority calculation.

### Process Run Queues and Context Switching 

The scheduling priority ranges between 0 and 127, with 0 to 49 reserved for processes executing in kernel mode, and 50 to 127 reserved for processes executing in user mode. 

The system uses 32 run queues, selecting a run queue for a process by dividing the process's priority by 4.  The run queues contain all the runnable processes in main memory except the currently running process.  The head of each run queue is kept in an array; associated with this array is a bit map, _whichqs_, that is used in identifying the nonempty run queues. 

Two main routines for managing the queues:

- _setrunqueue_() 
- _rmrq_()

The context switch is divided into two types

- _miswitch_() — machine-independent
- _Cpuswitch_() — machine dependent

Let's observe the importance of each function.

**_cpu\_switch_()**: the heart of the scheduling algorithm and is responsible for selecting a new process to run, and it operates as follows:

1. Block interrupts, then look for a nonempty run queue. Locate a nonempty queue by finding the location of the first nonzero bit in the _whichqs_ bit vector. If _whichqs_ is zero, there are no processes to run, so unblock interrupts and loop; this loop is the _idle loop_.  
    
2. Given a nonempty run queue, remove the first process on the queue.  
    
3. If this run queue is now empty as a result of removing the process, reset the appropriate bit in _whichqs_.  
    
4. Clear the _curproc_ pointer and the _want\_resched_ flag. The _curproc_ pointer references the currently running process. Clear it to show that _no process is currently running_. The _want\_resched_ flag shows that a context switch should take place; it is explained in next part  
    
5. Set the new process running and unblock interrupts.  
    

**_mi\_switch_():** It contains the machine independent part of the context-switching code. Which is called when a process wants to _sleep()_.

Now we have _mi\_switch_(), priority calculations, but we need a mechanism for forcing an involuntary context switching. 

- Voluntary context switches occur when a process calls the _sleep_() routine.  
    _Sleep_() can be invoked by only a runnable process, so _sleep_() needs only to place the process on a sleep queue and to invoke _mi\_switch_() to schedule the next process to run.  _mi\_switch()_ can't be called by a process that is in _interrupt_ state, because it must be called with in the context of a _running_ process.
- The new mechanism is handled by the machine-dependent _need\_resched_() routine, which generally sets a global _reschedule request_ flag, named _want\_resched_, and then posts an _asynchronous system trap_ (_AST_) for the current process.  
      
    when a process is woken up and if it comes across an AST trap it requests for reschedule instead of resuming the process execution.

The 4.4 BSD scheduler doesn’t support preemptions in kernel mode. So the drawback here is it is not. Real Time scheduler.  
This is about the 4.4BSD Scheduler, Let's learn about the advanced **M2 Scheduler** in the second part.

**References**:

- [http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-mlfq.pdf](http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-mlfq.pdf)
- [https://www.netbsd.org/docs/internals/en/chap-processes.html](https://www.netbsd.org/docs/internals/en/chap-processes.html)
- [http://www.netbsd.org/~rmind/pub/netbsd\_5\_scheduling\_apis.pdf](http://www.netbsd.org/~rmind/pub/netbsd_5_scheduling_apis.pdf)
