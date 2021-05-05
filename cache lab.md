---
title: cache lab
date: 2021-05-05 21:45:47
tags:
	csapp
---

cache lab 学习记录

<!--more-->

Prat A: 高速缓存模拟器

讲义里说可以用malloc动态分配cache大小，但这东西规模很小，我直接预定义成了个大二维数组。唯一能说的新东西就是getopt了，解答了我一直以来对程序参数处理的标准方案的疑惑

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <getopt.h>
#include "cachelab.h"
#define MAX_CACHE_SET 32
#define MAX_CACHE_LINE 32
#define DEBUG 0
int hit_cnt;
int miss_cnt;
int eviction_cnt;
int cmd_cnt;
int s,E,b;
struct cache_line
{
    int valid_bit;
    int tag_bit;
    int last_time;
}cache[MAX_CACHE_SET][MAX_CACHE_LINE];

void init();
void args_parse(int argc, char *argv[]);
void cmd_parse(char *cmd,long long addr);
void exec_cmd(long long addr);
void addr_parse(long long addr,int *tag_bit,int *set_id);
int main(int argc,char *argv[]);

void init(){
    hit_cnt=miss_cnt=eviction_cnt=cmd_cnt=0;
    memset(cache,0,sizeof(cache));
    return;
}
void args_parse(int argc, char *argv[]){
    char ch;
    while((ch=getopt(argc, argv,"s:E:b:t:"))!=-1){
        switch (ch)
        {
        case 's':
            s=atoi(optarg);
            break;
        case 'E':
            E=atoi(optarg);
            break;
        case 'b':
            b=atoi(optarg);
            break;
        case 't':
            freopen(optarg, "r", stdin);
        }
    }
    return;
}
void cmd_parse(char *cmd,long long addr){
    switch (cmd[0])
    {
    case 'I':
        break;
    case 'L':
        exec_cmd(addr);
        break;
    case 'S':
        exec_cmd(addr);
        break;
    case 'M':
        exec_cmd(addr);
        exec_cmd(addr);
        break;
    }
    return;
}
void addr_parse(long long addr,int *tag_bit,int *set_id){
    int tmp=0;
    for(int i=0;i<s;i++){
        tmp=(tmp<<1)+1;
    }
    *set_id=((int)(addr>>b)&tmp)%(1<<s);
    *tag_bit=(int)(addr>>(b+s));
    return;
}
void exec_cmd(long long addr){
    int tag_bit,set_id;
    cmd_cnt++;
    addr_parse(addr,&tag_bit,&set_id);
    if(DEBUG) printf("%d %d ",set_id,tag_bit);
    for(int i=0;i<E;i++){
        if((cache[set_id][i].valid_bit)
        &&(cache[set_id][i].tag_bit)==tag_bit
        ){
            cache[set_id][i].last_time=cmd_cnt;
            hit_cnt++;
            if(DEBUG) printf("hit\n");
            return;
        }
    }
    miss_cnt++;
    for(int i=0;i<E;i++){
        if(!cache[set_id][i].valid_bit){
            cache[set_id][i].valid_bit=1;
            cache[set_id][i].tag_bit=tag_bit;
            cache[set_id][i].last_time=cmd_cnt;
            if(DEBUG) printf("miss\n");
            return;
        }
    }
    eviction_cnt++;
    int victim_id=0;
    for(int i=0;i<E;i++){
        if(cache[set_id][i].last_time<cache[set_id][victim_id].last_time){
            victim_id=i;
        }
    }
    cache[set_id][victim_id].tag_bit=tag_bit;
    cache[set_id][victim_id].last_time=cmd_cnt;
    if(DEBUG) printf("miss eviction\n");
    return;
}
int main(int argc,char *argv[])
{
    init();
    args_parse(argc,argv);
    char cmd[10];
    long long addr;
    int blocksize;
    while(~scanf("%s %llx,%d",cmd,&addr,&blocksize)){
        cmd_parse(cmd,addr);
    }
    printSummary(hit_cnt, miss_cnt, eviction_cnt);
    return 0;
}


```

Part B:优化矩阵转置函数

这题用到了一种名为分块(blocking)的技术，指将要处理的大块数据分割为可以放入L1高速缓存的小块，把与这一小块的全部相关操作一次性处理完再切换下一小块，以此来提高L1高速缓存的利用率。块的尺寸可以通过枚举测试或者数学方法分析来决定。

```c
void transpose_submit(int M, int N, int A[N][M], int B[M][N])
{
    int rr,cc,r,c;
    int bsize;
    if(M==32) bsize=8;
    else if(M==64) bsize=4;
    else if(M==61) bsize=16;
    for(rr=0;rr<N;rr+=bsize){
        for(cc=0;cc<M;cc+=bsize){
            for(r=rr;r<N&&r<rr+bsize;r++){
                for(c=cc;c<M&&c<cc+bsize;c++){
                    B[c][r]=A[r][c];
                }
            }
        }
    }
    return;
}

```

得分情况：

```
Part A: Testing cache simulator
Running ./test-csim
                        Your simulator     Reference simulator
Points (s,E,b)    Hits  Misses  Evicts    Hits  Misses  Evicts
     3 (1,1,1)       9       8       6       9       8       6  traces/yi2.trace
     3 (4,2,4)       4       5       2       4       5       2  traces/yi.trace
     3 (2,1,4)       2       3       1       2       3       1  traces/dave.trace
     3 (2,1,3)     167      71      67     167      71      67  traces/trans.trace
     3 (2,2,3)     201      37      29     201      37      29  traces/trans.trace
     3 (2,4,3)     212      26      10     212      26      10  traces/trans.trace
     3 (5,1,5)     231       7       0     231       7       0  traces/trans.trace
     6 (5,1,5)  265189   21775   21743  265189   21775   21743  traces/long.trace
    27


Part B: Testing transpose function
Running ./test-trans -M 32 -N 32
Running ./test-trans -M 64 -N 64
Running ./test-trans -M 61 -N 67

Cache Lab summary:
                        Points   Max pts      Misses
Csim correctness          27.0        27
Trans perf 32x32           6.9         8         343
Trans perf 64x64           1.2         8        1891
Trans perf 61x67          10.0        10        1992
          Total points    45.1        53

```



