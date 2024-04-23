+++
title = 'Multithreading With Omp Tools'
date = 2024-04-23T11:40:39+05:30
draft = false
+++

## Introduction

In the last post, we discussed about parallel construct and left at `Execution Model Events`. I read it and found that it consists a lot thread dispatching and hence in this blog, I'll discuss about multithreading and how a small openmp example can be solved using it.

Also, I have a few questions in my mind, that I'll post at the end.

## Multithreading

We all know why and when to use threading, it is provided under `<thread>` header file for `c++` and can be called following the syntax:

```cpp
std::thread thread_object(callable);
```

std::thread is the thread class that represents a single thread in C++. To start a thread we simply need to create a new thread object and pass the executing code to be called (i.e, a callable object) into the constructor of the object. Once the object is created a new thread is launched which will execute the code specified in callable. A callable can be any of the five:

- A Function Pointer
- A Lambda Expression
- A Function Object
- Non-Static Member Function
- Static Member Function

### Launching Thread Using Function Pointer

A function pointer can be a callable object to pass to the std::thread constructor for initializing a thread. The following code snippet demonstrates how it is done.

```cpp
void foo(param)
{ 
  Statements; 
}
// The parameters to the function are put after the comma
std::thread thread_obj(foo, params);
```

## Openmp example

For our understanding, I'll use the following toy example:

```cpp
#include <stdio.h>
#include <omp.h>

int main() {
    int thread_id;

    printf("omp_get_max_threads(): %d\n", omp_get_max_threads());
    #pragma omp parallel private(thread_id)
    {
        thread_id = omp_get_thread_num();
        printf("Hello from thread: %d\n", thread_id);
    }
    return 0;
}
```

One can execute this using:

```console
% /opt/homebrew/bin/g++-13 -fopenmp a.cpp && ./a.out
omp_get_max_threads(): 8
Hello from thread: 1
Hello from thread: 5
Hello from thread: 6
Hello from thread: 3
Hello from thread: 2
Hello from thread: 4
Hello from thread: 0
Hello from thread: 7

```

This can be very easily translated to:

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <omp.h>
using namespace std;

std::mutex mtx;

void helloFromThread( int thread_id ) {
	mtx.lock();
	std::cout << "Hello from thread: " << thread_id << std::endl;
	mtx.unlock();
}

int main() {
	const int num_threads = omp_get_max_threads();

	cout << "omp_get_max_threads(): "<<omp_get_max_threads()<<"\n";

	std::thread threads[num_threads];

	for (int i = 0; i < num_threads; ++i) {
        	threads[i] = std::thread(helloFromThread, i);
    	}

    	for (int i = 0; i < num_threads; ++i) {
        	threads[i].join();
    	}

	return 0;
}
```

To execute this you will have to use `-pthread` flag and command will be:

```console
% /opt/homebrew/bin/g++-13 a-openmp.cpp -fopenmp -pthread && ./a.out
omp_get_max_threads(): 8
Hello from thread: 0
Hello from thread: 1
Hello from thread: 3
Hello from thread: 4
Hello from thread: 5
Hello from thread: 6
Hello from thread: 7
Hello from thread: 2
```

To ensure that the output from different threads doesn't get mixed up, we use `mutex`. `mutex` synchronize access to the output stream.

## Using Posix threads

One may definetly think, we already achieved multithreading using `std::thread` then what is the need to read about `Posix threads`. So, as per what [stackoverflow](https://stackoverflow.com/questions/13134186/c11-stdthread-vs-posix-threads) suggests:

_If you want to run code on many platforms, go for Posix Threads. They are available almost everywhere and are quite mature. On the other hand if you only use Linux/gcc std::thread is perfectly fine - it has a higher abstraction level, a really good interface and plays nicely with other C++11 classes._

_The C++11 std::thread class unfortunately doesn't work reliably (yet) on every platform, even if C++11 seems available. For instance in native Android std::thread or Win64 it just does not work or has severe performance bottlenecks (as of 2012)._

_A good replacement is boost::thread - it is very similar to std::thread (actually it is from the same author) and works reliably, but, of course, it introduces another dependency from a third party library._

Now, having known requirement of posix threads, this is how the modified code looks like:

```cpp
#include <iostream>
#include <pthread.h>
#include <omp.h>

pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;

void* helloFromThread(void* arg) {
    int thread_id = *((int*)arg);
    pthread_mutex_lock(&mtx);
    std::cout << "Hello from thread: " << thread_id << std::endl;
    pthread_mutex_unlock(&mtx);
    return NULL;
}

int main() {
    const int num_threads = omp_get_max_threads();
    pthread_t threads[num_threads];
    int thread_ids[num_threads];

    std::cout << "omp_get_max_threads(): " << omp_get_max_threads() << "\n";

    for (int i = 0; i < num_threads; ++i) {
        thread_ids[i] = i;
        pthread_create(&threads[i], NULL, helloFromThread, (void*)&thread_ids[i]);
    }

    for (int i = 0; i < num_threads; ++i) {
        pthread_join(threads[i], NULL);
    }

    return 0;
}
```

You have to now just include `<pthread.h>` and use `PTHREAD_MUTEX_INITIALIZER` instead and apply changes at consequent places.

```console
% time /opt/homebrew/bin/g++-13 a-pthread-openmp.cpp -fopenmp -pthread && ./a.out
/opt/homebrew/bin/g++-13 a-pthread-openmp.cpp -fopenmp -pthread  0.17s user 0.16s system 78% cpu 0.432 total
omp_get_max_threads(): 8
Hello from thread: 0
Hello from thread: 3
Hello from thread: 2
Hello from thread: 4
Hello from thread: 1
Hello from thread: 5
Hello from thread: 6
Hello from thread: 7
```

## Question

- I see no builtin way to execute a parallel construct in [`runtime library`](https://www.openmp.org/spec-html/5.0/openmp.html) routines provided by `omp`. `gnu` and `llvm` guys have developed their own wrappers over these runtime library, so I think we'll have to do that, and if yes, was the above example a correct direction to proceed?

## References
- https://www.openmp.org/spec-html/5.0/openmp.html
- https://www.geeksforgeeks.org/multithreading-in-cpp/
