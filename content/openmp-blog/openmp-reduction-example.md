+++
title = 'Openmp Reduction Example'
date = 2024-04-27T14:38:33+05:30
draft = false
+++

## Introduction

Having looked at an example of array intialization, in this blog, I'll provide a simple reduction example that does parallel sum of an array of fairly large dimension using openmp pragma and posix threads.

## Example

### Openmp

In this example, we will intialize an array and then perform parallel sum using `pragma omp parallel for` and `pragma critical`, we need to setup `private` and `shared` variables correctly to avoid any overwriting.

```cpp
#include <stdio.h>
#include <omp.h>

void initialize_array( int n, float *a, float val ) {
	int i;
	#pragma omp parallel
	{
		#pragma omp for
		for (i = 0; i < n; i++) {
			a[i] = val;
		}
	}
}

void parallel_Sum( int n, float *a ) {
	int i;
	float partial_sum, total_sum;

	#pragma omp parallel private(partial_sum) shared(total_sum)
	{
		partial_sum = 0;
		total_sum = 0;

		#pragma omp for
		for (i = 0; i < n; i++) {
			partial_sum += a[i];
		}

		#pragma critical
		{
			total_sum += partial_sum;
		}
	}

	printf("Total sum: %f\n", total_sum);
}

int main() {
	float a[1000000];

	initialize_array(1000000, a, 1.0);

	parallel_Sum(1000000, a);

    return 0;
}
```

This can be executed using

```console
% time /opt/homebrew/bin/g++-13 reduction.cpp -fopenmp && ./a.out
/opt/homebrew/bin/g++-13 reduction.cpp -fopenmp  0.08s user 0.04s system 121% cpu 0.101 total
Total sum: 1000000.000000
```

### pthreads

To write this using `pthreads` one just have to handle `partialSums` correctly and then the job is done.

```cpp
#include <iostream>
#include <pthread.h>
#include <omp.h>

pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;

struct ThreadArgs {
    int start;
    int end;
    float* array;
    float* partialSum;
};

void initialize_array( int n, float *a, float val ) {
    int i;
    #pragma omp parallel
    {
        #pragma omp for
        for (i = 0; i < n; i++) {
            a[i] = val;
        }
    }
}

void* parallelSum(void *arg) {
    ThreadArgs* args = static_cast<ThreadArgs*>(arg);

    float partialSum = 0.0f;

    for (int i = args->start; i < args->end; ++i) {
        partialSum += args->array[i];
    }

    pthread_mutex_lock(&mtx);
    *args->partialSum += partialSum;
    pthread_mutex_unlock(&mtx);

    delete args;
    return nullptr;
}

int main() {
    float a[1000000];

    initialize_array(1000000, a, 1.0);

    int num_threads = omp_get_max_threads();
    pthread_t threads[num_threads];
    float partialSums[num_threads];

    for ( int i = 0; i < num_threads; i++ ) {
        partialSums[i] = 0.0f;
    }

    int chunk_size = 1000000 / num_threads;

    for (int i = 0; i < num_threads; ++i) {
        ThreadArgs* args = new ThreadArgs();
        args->array = a;
        args->partialSum = &partialSums[i];
        args->start = i * chunk_size;
        args->end = (i == num_threads - 1) ? 1000000 : (i + 1) * chunk_size;
        pthread_create(&threads[i], nullptr, parallelSum, static_cast<void*>(args));
    }

    for (int i = 0; i < num_threads; ++i) {
        pthread_join(threads[i], nullptr);
    }

    float totalSum = 0.0f;
    for (int i = 0; i < num_threads; ++i) {
        totalSum += partialSums[i];
    }

    std::cout << "Total sum: " << totalSum << std::endl;

    return 0;
}
```

To execute this you'll need to use `-pthread` flag as shown

```console
% time /opt/homebrew/bin/g++-13 reduction-pthread-openmp.cpp -pthread -fopenmp && ./a.out
/opt/homebrew/bin/g++-13 reduction-pthread-openmp.cpp -pthread -fopenmp  0.18s user 0.05s system 111% cpu 0.202 total
Total sum: 1e+06
```

## Conclusion

That's it for this blog, thank you :)

----


