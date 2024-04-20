+++
title = 'Understanding Parallel Construct'
date = 2024-04-20T16:23:12+05:30
draft = false
+++

OpenMP C example using omp runtime APIs

## Introduction

In the last attempt to parallelize code using `libgomp` we encountered that we were not able to link `gomp_parallel` even when it was present in runtime library and upon discussion with [Ondrej Certik]() we decided not to proceed with `libgomp` and instead use standard OpenMP runtime APIs.

So, to begin, I picked up a minimal code with only a `parallel` construct and understand how it works. The code is as follows:

```c
#include <stdio.h>
#include <omp.h>

int main() {
    int thread_id;

    #pragma omp parallel private(thread_id)
    {
        thread_id = omp_get_thread_num();
        printf("Hello from thread: %d\n", thread_id);
    }
    return 0;
}
```

Which gives the following output: ( num_threads = 8 )

```console
% /opt/homebrew/bin/g++-13 a.cpp -fopenmp && ./a.out
Hello from thread: 1
Hello from thread: 2
Hello from thread: 3
Hello from thread: 5
Hello from thread: 4
Hello from thread: 6
Hello from thread: 7
Hello from thread: 0
```

The code is simple, it prints the thread id of each thread that is spawned by the `parallel` construct. The `omp_get_thread_num()` function returns the thread id of the calling thread.

I am following [OPENMP API Specification: Version 5.0 November 2018](https://www.openmp.org/spec-html/5.0/openmpse14.html) to understand the APIs and their usage.

## Parallel construct

The [parallel construct](https://www.openmp.org/spec-html/5.0/openmpse14.html) creates a team of OpenMP threads that execute the region.

### Description

When a thread encounters a parallel construct, a team of threads is created to execute the parallel region. The number of threads is determined according to [Algorithm 2.1](https://www.openmp.org/spec-html/5.0/openmpsu35.html#x55-880002.6.1).

> Algorithm 2.1: Number of Threads in a Parallel Region

```c
unsigned get_num_threads() {
    // let ThreadsBusy be the number of OpenMP threads currently executing in this contention group;
    unsigned ThreadsBusy = 0;

    // let ActiveParRegions be the number of enclosing active parallel regions
    unsigned ActiveParRegions = omp_get_active_level();

    // assuming num_threads clause is not present
    unsigned ThreadsRequested = omp_get_max_threads();

    // let ThreadsAvailable = (thread-limit-var - ThreadsBusy + 1);
    unsigned ThreadsAvailable = omp_get_thread_limit() - ThreadsBusy + 1;

    // if clause is not present thus

    bool IfClauseValue = true;

    if (IfClauseValue) return 1;
    if (ActiveParRegions == omp_get_max_active_levels()) return 1;
    if (omp_get_dynamic() && ThreadsRequested <= ThreadsAvailable) return ThreadsRequested;
    if (omp_get_dynamic() && ThreadsRequested > ThreadsAvailable) return ThreadsAvailable;
    if (!omp_get_dynamic() && ThreadsRequested <= ThreadsAvailable) return ThreadsRequested;
    if (!omp_get_dynamic() && ThreadsRequested > ThreadsAvailable) return ThreadsAvailable;
}
```

There is an implied barrier at the end of a parallel region. After the end of a parallel region, only the master thread of the team resumes execution of the enclosing task region.

A thread in a team executing a parallel region encounters another parallel directive, it creates a new team of threads to execute the new parallel region. The new team of threads is created by the encountering thread, not by any of the threads in the original team.


### Execution Model Events

TBD in the next post.

