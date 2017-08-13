---
title: Python基础复习-GIL(一)
date: 2017-08-13 11:00:48
category: Python
tags: GIL
---

### GIL
>GIL，即全局解释器锁（Global Interpreter Lock），是计算机程序设计语言解释器用于同步线程的工具，使得任何时刻仅有一个线程在执行。

### Python的GIL
>CPython的线程是操作系统的原生线程。在Linux上为pthread，在Windows上为Win thread，完全由操作系统调度线程的执行。一个python解释器进程内有一条主线程，以及多条用户程序的执行线程。即使在多核CPU平台上，由于GIL的存在，所以禁止多线程的并行执行。

>Python解释器进程内的多线程是合作多任务方式执行。当一个线程遇到I/O任务时，将释放GIL。计算密集型（CPU-bound）的线程在执行大约100次解释器的计步（ticks）时，将释放GIL。计步（ticks）可粗略看作Python虚拟机的指令。计步实际上与时间片长度无关。可以通过sys.setcheckinterval()设置计步长度。

>在单核CPU上，数百次的间隔检查才会导致一次线程切换。在多核CPU上，存在严重的线程颠簸（thrashing）。

>Python 3.2开始使用新的GIL。在新的GIL实现中，用一个固定的超时时间来指示当前的线程放弃全局锁。在当前线程保持这个锁，且其他线程请求这个锁的时候，当前线程就会在5ms后被强制释放掉这个锁。

>可以创建独立的进程来实现并行化。Python 2.6引进了multiprocessing这个多进程包。或者把关键部分用C/C++写成 Python 扩展，通过cytpes使Python程序直接调用C语言编译的动态库的导出函数。
