---
title: 'Trace Kernel Function'
layout: post
tags:
 - Ftrace
 - Linux Kernel
category: Linux 
---

In this report, we will learn the basic usage of "ftrace", a linux kernel tracing tools to trace kernel events. Also, we will learn who to change the kernel source code to add custom kernel functions.
<!--more-->

## 0. Environment
### Host Machine
**Machine:** MacBook Pro (13-inch, Late 2016, Four Thunderbolt 3 Ports)  
**Processor:** 3.1 GHz Intel Core i5 (2-core)  
**Memory:** 16 GB 2133 MHz LPDDR3  
**Operating System:** MacOS Sierra10.12.3  


### Guest Machine Settings
**Software:** Virtual Box 5.1.15, VMware 8.5.3 (just one of them is enough)  
**CPU Core Number:** 2  
**Memory Size:** 2GB  
**Disk Size:** 35GB  
**Operating System:** Ubuntu16.04.1 LTS **Kernel:** 4.4.0-31-generic   
**GCC:** 5.4.0

## Task 1
- **a.** Ftrace uses memory to keep the trace record. Before starting a new trace, the old traces are still kept in the memory by ftrace has to be emptied. How can this be done? (1 mark)  
**My Answer:** To empty the ring buffer in the memory. We can firstly trace 'nop' for a few seconds and then start the new trace. So the command we use is `$ echo nop > current_tracer`. Notice that `$ echo 0 > tracing_on` may **not work** since it just stops pushing back records into the ring buffer but may not empty it.

 
- **b.** What is the command to change the maximum size of the trace file of ftrace? (1 mark)  
**My Answer:** 
We can use the command `$ echo 14008 > buffer_size_kb` to change the size of the buffer. 
The default value of `buffer_size_kb` is `1408`, so after the running of above command the file `trace` will be 10 times bigger.


- **c.** Assuming you are changing the kernel code to insert a printk‐like code to output hello world in ftrace and the message given by the print code is the only thing you want to trace, show the code and also the tracer that should use in ftrace. (2 marks)  
**My Answer:**  
When changing the kernel code, using `trace_printk` instead of `printk`:

```
#include <linux/kernel.h>   /* for printk */
#include <linux/syscalls.h>  /* for SYSCALL_DEFINE1 macro */

SYSCALL_DEFINE1(printmsg, int, i)
{
    trace_printk(KERN_DEBUG "A0159369L say hello to you! from %d", i);
    return 1;
}

```
Then by looking at the trace file, we will see the output.

    [tracing]# cat trace
    # tracer: nop
    #
    #           TASK-PID    CPU#    TIMESTAMP  FUNCTION
    #              | |       |          |         |
               <...>-10690 [003] 17279.332920: : A0159369L say hello to you! from 668

The tracer used here is `nop`.  

- **d.** Use the function tracer to trace the kernel for about a few seconds. Give the screenshots of the first 20 lines of the trace file and analyze the basic structure of it. (2 marks)  
**My Answer:** 
![function_trace_first20lines](function_trace_first20lines.png)
The header line explained the trace file quite well. The first collum shows the task traced and the PID of the process. The second collum tells which CPU the process is running on. In this case, all the records shown in the picture are running on CPU0. The timestamp is the time since boot, with precision of micro-second. The last collum is the function being traced with its parent following the "<-" symbol.


## Task 2

Firstly set the max depth of stack to 10. And then set the `vfs_open`,`vsf_read` and `vfs_write` as the "trigger" of start tracing:

```
echo 10 > max_graph_depth
echo vfs_open vfs_read vfs_write > set_graph_function
echo 1 > tracing_on
```
![img_function_graph](function_graph.png)

This tracer is similar to the function tracer except that it
probes a function on its entry and its exit. [^quote] For example, in the line39 of the picture, there is a `vfs_read()` function call. Next in the line40, `vfs_read()` called another function `rw_verify_area()` and more other functions. These functions nesting together and will return one by one as the trace graph shows. The first collum and the second collum of the trace file show the CPU number and the duration of the execution time of the function on the right.

## Task 3

- **a.** Finish the steps for adding a kernel function into kernel in the tutorial. Supply the two C files. (2 marks)  
**Answer:** Done.

- **b.** What is the full path of the system call table? Show the line you added in the system call table. (2 marks)  
**Answer:** 
The full path of the system call table is:
```
./arch/x86/entry/syscalls/syscall_64.tbl
```  
I added:  
```
...
329        64        printmsg        sys_printmsg
...
```  
to the syscall table.

![img_task_3b](task3b.png)

- **c.** Show the screenshot of the “dmesg | tail” containing the print message of your new kernel function. (2 marks)  
**Answer:**
![img_task_3c](task3c.png)


- **d.** Give the first 20 lines of the trace file of ftrace and analyse it. (2 marks)  
**Answer:**
I tried the method metioned in **Task 1c**, and it worked.
![img_trace_printk](trace_printk.png)
In the picture above, there are totally 3 records of the message printed by `trace_printk()`, each with thier task pid and timestamp, as weel as the CPU they were running on.

[^quote]: [Ftrace Documentation](https://www.kernel.org/doc/Documentation/trace/ftrace.txt)
