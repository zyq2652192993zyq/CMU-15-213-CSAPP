> # Shell Lab

Students implement their own simple Unix shell program with job control, including the ctrl-c and ctrl-z keystrokes, fg, bg, and jobs commands. This is the students' first introduction to application level concurrency, and gives them a clear idea of Unix process control, signals, and signal handling.

## Files

```bash
Makefile	# Compiles your shell program and runs the tests
README		# This file
tsh.c		# The shell program that you will write and hand in
tshref		# The reference shell binary.

# The remaining files are used to test your shell
sdriver.pl	# The trace-driven shell driver
trace*.txt	# The 15 trace files that control the shell driver
tshref.out 	# Example output of the reference shell on all 15 traces

# Little C programs that are called by the trace files
myspin.c	# Takes argument <n> and spins for <n> seconds
mysplit.c	# Forks a child that spins for <n> seconds
mystop.c        # Spins for <n> seconds and sends SIGTSTP to itself
myint.c         # Spins for <n> seconds and sends SIGINT to itself
```

## Trace file

`eval`函数的框架在课程`lecture`里有，然后根据不同的`trace`做一些增加。

```c
void eval(char *cmdline) 
{
    char *argv[MAXARGS]; /* Argument list execve() */
    char buf[MAXLINE];   /* Holds modified command line */
    int bg;              /* Should the job run in bg or fg? */
    pid_t pid;           /* Process id */
    
    strcpy(buf, cmdline);
    bg = parseline(buf, argv);       //解析命令行函数都提供好了
    if (argv[0] == NULL)  
    return;   /* Ignore empty lines */

    if (!builtin_command(argv)) { 
        if ((pid = Fork()) == 0) {   /* 子进程来执行job */
            if (execve(argv[0], argv, environ) < 0) {
                printf("%s: Command not found.\n", argv[0]);
                exit(0);
            }
        }
    /* 如果是前台作业，主进程需要等待子进程运行完毕 */
    if (!bg) {
        int status;
        if (waitpid(pid, &status, 0) < 0)
        unix_error("waitfg: waitpid error");
    }
    else
        printf("%d %s", pid, cmdline);
    }
    return;
}
```

框架有了，然后根据`trace`来完善这个框架。

* `trace01.txt`的部分只要`makefile`正常运行即可通过：

```bash
$ make test01
./sdriver.pl -t trace01.txt -s ./tsh -a "-p"
#
# trace01.txt - Properly terminate on EOF.
#
```

* `trace02.txt`需要支持`built-in commands`中的`quit`命令，对应着`exit(0);`，需要修改`int builtin_cmd(char **argv);`这个函数

```c
/* 
 * builtin_cmd - If the user has typed a built-in command then execute
 *    it immediately.  
 */
int builtin_cmd(char **argv) 
{
    if (!strcmp(argv[0], "quit")) exit(0); /* quit command */

    return 0;     /* not a builtin command */
}

```

```bash
$ make test02
./sdriver.pl -t trace02.txt -s ./tsh -a "-p"
#
# trace02.txt - Process builtin quit command.
#
```

如果程序并没有退出，仍然需要`ctrl  + c`才能退出的话，意味着代码存在问题。

* `trace03.txt`在前台执行`quit`，无需添加任何内容，可以正常通过

```bash
$ make test03
./sdriver.pl -t trace03.txt -s ./tsh -a "-p"
#
# trace03.txt - Run a foreground job.
#
tsh> quit
```

* `trace04.txt`需要处理一个后台进程，当输入`./myspin 1 &`的时候，`job`的数量要增加一个，并且打印`job`的信息。主进程通过`fork`出的子进程来处理这个后台任务。需要父进程先往任务列表里添加一个`job`，然后等待子进程执行完毕，最后打印后台任务的信息。此时需要用到信号处理程序。 此部分需要把书中`8.5.6, 8.5.7`的程序看懂。主要需要修改`eval`函数，另外程序还对阻塞信号的相关函数进行了封装，可以直接从`csapp.c`文件内提取。

```c
void eval(char *cmdline) 
{
    char *argv[MAXARGS]; /* Argument list execve() */
    char buf[MAXLINE];   /* Holds modified command line */
    int bg;              /* Should the job run in bg or fg? */
    pid_t pid;           /* Process id */
    sigset_t mask_all, mask_one, prev_one;

    strcpy(buf, cmdline);
    bg = parseline(buf, argv);
    if (argv[0] == NULL) return; /* Ignore empty lines */

    Sigfillset(&mask_all);
    Sigemptyset(&mask_one);
    Sigaddset(&mask_one, SIGCHLD);

    if (!builtin_cmd(argv)) {
        Sigprocmask(SIG_BLOCK, &mask_one, &prev_one); /* Block SIGCHLD */
        if ((pid = Fork()) == 0) { /* Child runs user job */
            Sigprocmask(SIG_SETMASK, &prev_one, NULL);  /* Unblock SIGCHLD */ 
            Setpgid(0,0);
            Execve(argv[0], argv, environ);
        }

        Sigprocmask(SIG_BLOCK, &mask_all, NULL); /* Parent process */
        addjob(jobs, pid, bg + 1, cmdline); /* Add the child to the job list FG 1 or BG 2*/
        Sigprocmask(SIG_SETMASK, &prev_one, NULL); /* Unblock SIGCHLD */
        
        /* Parent waits for foreground job to terminate */
        if (!bg) waitfg(pid);
        else printf("[%d] (%d) %s", pid2jid(pid), pid, cmdline);
    }
}
```

































