---
title:  "Linux内核设计和实现-进程状态"
date:   2017-05-05 17:03:44
categories: LINUXKERNEL
---

## **Linux进程状态**

在Linux中每个进程都必然处于以下五种状态中的一种。进程状态描述如下：

- TASK_RUNNIN(运行) --- 进程是可执行的；表示进程正在处于执行，或者在运行队列中等待执行。
- TASK_INTERRUPTIBLE(可中断) --- 进程正在睡眠（即处于被阻塞状态）。等待某些条件达成，内核就会把进程状态设置为运行。处于此状态的进程也会因为接收到信号而提前被唤醒并随时准备投入运行。
- TASK_UNINTERRUPTIBLE(不可中断) --- 这个状态通常在进程必须在等待时不受干扰，就算接收到信号也不会被唤醒或准备投入运行中。比如进程的执行依赖与IO完成为前提，如果IO未完成，进程设置为运行状态也没有意义。
- _TASK_TRACED --- 被其它进程跟踪的进程，例如通过ptrace对调试程序进行跟踪。
- _TASK_STOPPED(停止) --- 进程停止执行。通常这种状态发生在接收到SIGSTOP,SIGTSTP,SIGTTIN,SIGTTOU等信号的时候。此外在调试期间接收到任何信号，都会使进程进入这种状态。

## **进程状态机**

![process-states]({{ site.url }}/images/linux/process-states.png)