> # Malloc Lab

项目链接：<http://csapp.cs.cmu.edu/3e/labs.html>

完成此项目需要阅读的内容：

* Lecture 17 VM concepts
* Lecture 18 VM systems
* Lecture 19 malloc basis
* Lecture 20 malloc advanced

扩展阅读的内容：

* 《Linux/Unix系统编程手册》上册的第6章 进程
* 《Linux/Unix系统编程手册》上册的第7章 内存分配
* 《Linux/Unix系统编程手册》上册的第24章 进程的创建
* 《Linux/Unix系统编程手册》上册的第25章 进程的终止
* 《Linux/Unix系统编程手册》下册的第49章 内存映射
* 《Linux/Unix系统编程手册》下册的第50章 虚拟内存操作

## Main Files

`mm.{c,h}`

Your solution malloc package. mm.c is the file that you will be handing in, and is the only file you should modify.

`mdriver.c`

The malloc driver that tests your mm.c file

`short{1,2}-bal.rep`

Two tiny tracefiles to help you get started.

`Makefile`

Builds the driver

## Other support files for the driver

* `config.h`	Configures the malloc lab driver
* `fsecs.{c,h}`	Wrapper function for the different timer packages
* `clock.{c,h}`	Routines for accessing the Pentium and Alpha cycle counters
* `fcyc.{c,h}`	Timer functions based on cycle counters
* `ftimer.{c,h}`	Timer functions based on interval timers and gettimeofday()
* `memlib.{c,h}`	Models the heap and sbrk function

## Building and running the driver

To build the driver, type "make" to the shell.

To run the driver on a tiny test trace:

```bash
unix> mdriver -V -f short1-bal.rep
```

The -V option prints out helpful tracing and summary information.

To get a list of the driver flags:

```c
unix> mdriver -h
```





































