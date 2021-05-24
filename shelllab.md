---
title: shelllab
date: 2021-05-24 10:05:31
tags:
    -csapp
---

shell lab 学习记录

<!--more-->

shell lab要求我们实现一个简化版的shell，需要能够实现子程序的前台与后台运行以及切换，同时实现四条内建命令。

```
quit： 退出shell

jobs： 打印子进程列表与运行状态

fg： 切换进程到前台运行

bg： 切换进程到后台运行
```



eval()

```c
void eval(char *cmdline)
{
    char *argv[MAXARGS];
    char buf[MAXLINE];
    int bg;
    pid_t pid;
    sigset_t mask_one, mask_all, pre_mask;
    sigfillset(&mask_all);
    sigemptyset(&mask_one);
    sigaddset(&mask_one, SIGCHLD);
    strcpy(buf, cmdline);
    bg = parseline(buf, argv);
    if (argv[0] == NULL)
        return;
    if (!builtin_cmd(argv))
    {
        sigprocmask(SIG_BLOCK, &mask_one, &pre_mask);//阻断SIGCHLD
        if ((pid = fork()) == 0)
        {
            setpgid(0, 0); //*******很重要
            sigprocmask(SIG_SETMASK, &pre_mask, NULL);
            if (execve(argv[0], argv, environ) < 0)
            {
                printf("%s: Command not found
", argv[0]);
                exit(0);
            }
        }
        sigprocmask(SIG_BLOCK, &mask_all, NULL);//addjob涉及全局变量访问
        if (bg)
            addjob(jobs, pid, BG, buf);
        else
            addjob(jobs, pid, FG, buf);
        if (!bg)
            waitfg(pid);
        else
        {
            struct job_t *curr_bgmask = getjobpid(jobs, pid);
            printf("[%d] (%d) %s", curr_bgmask->jid, curr_bgmask->pid, curr_bgmask->cmdline);
        }
        sigprocmask(SIG_SETMASK, &pre_mask, NULL);
    }
    return;
}

```

这个函数不难，书上的代码拿来改改就能用，但是有问题需要注意

![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/aHR0cHM6Ly95dWxhbi5uZXQuY24vZmlsZXMvdXBsb2FkLzkxMzEwMzAwMjBmMzQ2NjRhZjZkYzVkNGIxOWY3ZWU4LnBuZw)

大体意思就是在Unix shell上运行我们自己的shell时，需要在创建出一个子进程之后调用`setpgid(0, 0) `将他们放到一个新的进程组，否则我们的`ctrl-c `与`ctrl-z `会把信号发送到包括我们的shell在内的全部进程中，就是因为没注意这个使得我的shell产生的很多迷幻的错误，以至于调试很久却不知道原因



waitfg()

按照书上的思路实现即可

```c
void waitfg(pid_t pid)
{
    sigset_t mask_empty, mask_all, mask_pre;
    sigemptyset(&mask_empty);
    sigfillset(&mask_all);
    sigprocmask(SIG_SETMASK, &mask_all, &mask_pre); //fgpid涉及全局变量的访问
    while (pid == fgpid(jobs))
        sigsuspend(&mask_empty);
    sigprocmask(SIG_SETMASK, &mask_pre, NULL);
    return;
```



 sigchld_handler()

```c
void sigchld_handler(int sig)
{
    pid_t pid;
    int status;
    int old_errno = errno;
    sigset_t mask_empty, mask_all, mask_pre;
    sigemptyset(&mask_empty);
    sigfillset(&mask_all);
    while ((pid = waitpid(-1, &status, WNOHANG | WUNTRACED)) > 0)
    {
        sigprocmask(SIG_BLOCK, &mask_all, &mask_pre);
        if (WIFSTOPPED(status)) //暂停
        {
            struct job_t *job = getjobpid(jobs, pid);
            job->state = ST;
            printf("Job [%d] (%d) stopped by signal %d
",job->jid, pid, WSTOPSIG(status));
        }
        else
        {
            if (WIFSIGNALED(status)) //被信号退出
            {
                struct job_t *job = getjobpid(jobs, pid);
                printf("Job [%d] (%d) terminated by signal %d
", job->jid, job->pid, WTERMSIG(status));
            } 
            deletejob(jobs, pid);
        }
        sigprocmask(SIG_SETMASK, &mask_pre, NULL);
    }
    errno = old_errno;
    return;
}
```



sigint_handler() 与 sigtstp_handler()

```c
void sigint_handler(int sig)
{
    pid_t pid;
    int old_errno = errno;
    if ((pid = fgpid(jobs)) != 0)
    {
        kill(-pid, SIGINT);
    }
    errno = old_errno;
    return;
}

void sigtstp_handler(int sig)
{
    pid_t pid;
    int old_errno = errno;
    pid = fgpid(jobs);
    if (pid)
    {
        kill(-pid, sig);
    }
    errno = old_errno;
    return;
}

```



builtin_cmd()

```c
int builtin_cmd(char **argv)
{
    if (!strcmp(argv[0], "quit"))
        exit(0);
    else if (!strcmp(argv[0], "jobs"))
    {
        listjobs(jobs);
        return 1;
    }
    else if (!strcmp(argv[0], "bg") || !strcmp(argv[0], "fg"))
    {
        do_bgfg(argv);
        return 1;
    }
    return 0; /* not a builtin command */
}


```

这个函数与书上的几乎一模一样



do_bgfg()

```c
void do_bgfg(char **argv)
{
    struct job_t *job;
    if(argv[1] == NULL){
        printf("%s command requires PID or %%jobid argument
",argv[0]);
        return;
    }
    char * idstring = argv[1];
    int isjid = 0;
    int id;
    if(idstring[0]=='%'){   //以jobid作为目标
        isjid = 1;
        id = atoi(idstring+1);
        if(!id){
            printf("%s: argument must be a PID or %%jobid
",argv[0]);
            return;
        }
    }
    else{   //以pid作为目标
        id = atoi(idstring);
        if(!id){
            printf("%s: argument must be a PID or %%jobid
",argv[0]);
            return;
        }
    }
    sigset_t mask_empty, mask_all, mask_pre;
    sigemptyset(&mask_empty);
    sigfillset(&mask_all);
    sigprocmask(SIG_SETMASK, &mask_all, &mask_pre); //与job相关操作涉及全局变量的访问
    if (isjid)  //寻找目标job
    {
        job = getjobjid(jobs, id);
        if (!job){
            printf("%%%d: No such job
", id);
            sigprocmask(SIG_SETMASK, &mask_pre, NULL);
            return;
        }
    }
    else 
    {
        job = getjobpid(jobs, id);
        if (!job){
            printf("(%d): No such process
", id);
            sigprocmask(SIG_SETMASK, &mask_pre, NULL);
            return;
        }
    }
    pid_t pid = job->pid;
    if (!strcmp(argv[0], "bg")) //bg fg切换
    {
        job->state = BG;
        printf("[%d] (%d) %s",job->jid,job->pid,job->cmdline);
        kill(-pid, SIGCONT);
    }
    else if (!strcmp(argv[0], "fg"))
    {
        job->state = FG;
        kill(-pid, SIGCONT);
        sigprocmask(SIG_SETMASK, &mask_pre, NULL);
        waitfg(pid);
        return;
    }
    sigprocmask(SIG_SETMASK, &mask_pre, NULL);
    return;
}

```

主要是要对参数是否符合要求进行判断