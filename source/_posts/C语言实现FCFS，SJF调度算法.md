---
title: C语言实现FCFS，SJF调度算法
date: 2022-05-11 18:43:40
tags: 操作系统
---

本次内容也是北京航空航天大学OS操作系统第五次课上作业。

![](https://note.youdao.com/yws/api/personal/file/69479141906C4090894C0F203F28D306?method=download&shareKey=79efbec86dd0a81e338db7172b17bb6b)

我所使用的变量顾名知义，不做过多的阐述。

```C
#include<stdio.h>
#include<stdlib.h>
int job_submitted_time[] = {0, 1, 2, 3, 12, 12, 12, 12, 200, 200};
int job_required_time[] = {4, 3, 2, 1, 6, 5, 4, 3, 1, 1};
int fcfs_result[10];
int sjf_result[10];
void FCFS (
    int number_of_jobs,
    const int job_submitted_time [],
    const int job_required_time [],
    int job_sched_start []
) {
    int currenttime = job_submitted_time[0];
    job_sched_start[0] = job_submitted_time[0];
    currenttime += job_required_time[0];
    for(int i=1;i<number_of_jobs;i++) {
        if(currenttime >= job_submitted_time[i]) {
            job_sched_start[i] = currenttime;
        } else {
            job_sched_start[i] = job_submitted_time[i];
            currenttime = job_submitted_time[i];
        }
        currenttime += job_required_time[i];
    }
}

void SJF (
    int number_of_jobs,
    const int job_submitted_time [],
    const int job_required_time [],
    int job_sched_start []
) {
    char record[2005];
    memset(record,0,sizeof(record));
    int currenttime = job_submitted_time[0];
    int k = number_of_jobs;
    int last = 0;
    //int flag = 0;
    while(k--){
        int mintime = (((unsigned int)(-1))>>1);
        int recordi = -1;
        
        for(int i = 0;i < number_of_jobs;i++) {
            if(job_submitted_time[i] > currenttime) {
                break;
            } else if(record[i] != '1') {
                if(job_required_time[i] < mintime) {
                    recordi = i;
                    mintime = job_required_time[i];
                }
            } else if(record[i] == '1') {
                if(i > last) {
                    last = i;
                }
            }
        }
        if(recordi == -1) {
            currenttime = job_submitted_time[last + 1];
            k++;
            continue;
        }
        record[recordi] = '1';
        job_sched_start[recordi] = currenttime;
        currenttime += job_required_time[recordi];
    }
}
int main() {
    int n = 10, i;
    FCFS(n, job_submitted_time, job_required_time, fcfs_result);
    puts("FCFS results:");
    for (i = 0; i < n; ++i) printf("job % 2d: [% 9d]\n", i, fcfs_result[i]);
    SJF(n, job_submitted_time, job_required_time, sjf_result);
    puts("SJF results:");
    for (i = 0; i < n; ++i) printf("job % 2d: [% 9d]\n", i, sjf_result[i]);
}

```

