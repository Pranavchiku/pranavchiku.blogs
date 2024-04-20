+++
title = 'Running Openmp With C Api'
date = 2024-04-20T14:40:13+05:30
draft = false
+++

To support openmp in LFortran, we decided to first implement it in C using OpenMP APIs and then port it to Fortran.

Here is a minimal cpp code utilizing `omp` ( `gomp` ) APIs to initialise array values.

```cpp
#include <stdio.h>
// #include <omp.h>
#include "/System/Volumes/Data/opt/homebrew/Cellar/gcc/13.2.0/lib/libgomp/libgomp_g.h"

struct Node {
    int n;
    float *a;
    float val;
};

void subfunction(void *data) {
    Node *d = (Node *)data;
    int n = d->n;
    float *a = d->a;
    float val = d->val;

    for (int i = 0; i < n; i++) {
        a[i] = val;
    }
}

int main() {
    float a[1000000];

    Node *data = new Node;
    data->n = 1000000;
    data->a = a;
    data->val = 1.0;

    GOMP_parallel(subfunction, data, 4, 0);

    for (int i = 0; i < 1000; i++) {
        printf("%f\n", a[i]);
    }

    return 0;
}
```

## Setup 

We need `libgomp.h` header file, which I was unable to install with command line and found that it is present at https://github.com/gcc-mirror/gcc/blob/master/libgomp/libgomp.h

So I manually cloned it and put at `/System/Volumes/Data/opt/homebrew/Cellar/gcc/13.2.0/lib`

```
git clone https://github.com/gcc-mirror/gcc.git
cp -R ./gcc/libgomp /System/Volumes/Data/opt/homebrew/Cellar/gcc/13.2.0/lib
```

and then include header file as given in example.

## Why?

This needs to be done to support `GOMP_parallel(xx)`.

## Outcome

With, this we get 

```console
% /opt/homebrew/bin/g++-13 -fopenmp f-openmp-api.cpp -o f.out -L/System/Volumes/Data/opt/homebrew/Cellar/gcc/13.2.0/lib/gcc/current -lgomp 
ld: warning: ignoring duplicate libraries: '-lgomp'
Undefined symbols for architecture arm64:
  "__Z13GOMP_parallelPFvPvES_jj", referenced from:
      _main in cceYPRhv.o
ld: symbol(s) not found for architecture arm64
collect2: error: ld returned 1 exit status
```

We linked `/System/Volumes/Data/opt/homebrew/Cellar/gcc/13.2.0/lib/gcc/current/libgomp.dylib` that consists of `000000000000cdc0 T _GOMP_parallel` but somehow it still throws error. 

Thanks to Thirumalai for his help :))
