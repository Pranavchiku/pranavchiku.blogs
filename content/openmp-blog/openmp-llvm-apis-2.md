+++
title = 'Openmp transformation using llvm-openmp runtime routines - 2'
date = 2024-05-03T00:27:48+05:30
draft = false
+++

## Introduction

In continuation to last blog, in this blog, we'll discuss array initialisation and reduction example.

## Array Initialisation

### openmp

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

int main() {
	float a[1000000];
	initialize_array(1000000, a, 12.91);

	for (int i = 0; i < 10; i++) {
		printf("%f\n", a[i]);
	}
}
```

```console
% time clang++ c.cpp -fopenmp && time ./a.out            
clang++ c.cpp -fopenmp  0.07s user 0.12s system 41% cpu 0.454 total
12.910000
12.910000
12.910000
12.910000
12.910000
12.910000
12.910000
12.910000
12.910000
12.910000
./a.out  0.00s user 0.00s system 103% cpu 0.005 total
```

### transformed code

At first, we will generate `llvm`:

<details>


<summary> llvm </summary>


```llvm
; ModuleID = 'c.cpp'
source_filename = "c.cpp"
target datalayout = "e-m:o-i64:64-i128:128-n32:64-S128"
target triple = "arm64-apple-macosx14.0.0"

%struct.ident_t = type { i32, i32, i32, i32, ptr }

@0 = private unnamed_addr constant [23 x i8] c";unknown;unknown;0;0;;\00", align 1
@1 = private unnamed_addr constant %struct.ident_t { i32 0, i32 514, i32 0, i32 22, ptr @0 }, align 8
@2 = private unnamed_addr constant %struct.ident_t { i32 0, i32 66, i32 0, i32 22, ptr @0 }, align 8
@3 = private unnamed_addr constant %struct.ident_t { i32 0, i32 2, i32 0, i32 22, ptr @0 }, align 8
@.str = private unnamed_addr constant [4 x i8] c"%f\0A\00", align 1

; Function Attrs: mustprogress noinline nounwind optnone ssp uwtable(sync)
define void @_Z16initialize_arrayiPff(i32 noundef %0, ptr noundef %1, float noundef %2) #0 {
  %4 = alloca i32, align 4
  %5 = alloca ptr, align 8
  %6 = alloca float, align 4
  %7 = alloca i32, align 4
  store i32 %0, ptr %4, align 4
  store ptr %1, ptr %5, align 8
  store float %2, ptr %6, align 4
  call void (ptr, i32, ptr, ...) @__kmpc_fork_call(ptr @3, i32 3, ptr @_Z16initialize_arrayiPff.omp_outlined, ptr %4, ptr %5, ptr %6)
  ret void
}

; Function Attrs: noinline norecurse nounwind optnone ssp uwtable(sync)
define internal void @_Z16initialize_arrayiPff.omp_outlined(ptr noalias noundef %0, ptr noalias noundef %1, ptr noundef nonnull align 4 dereferenceable(4) %2, ptr noundef nonnull align 8 dereferenceable(8) %3, ptr noundef nonnull align 4 dereferenceable(4) %4) #1 {
  %6 = alloca ptr, align 8
  %7 = alloca ptr, align 8
  %8 = alloca ptr, align 8
  %9 = alloca ptr, align 8
  %10 = alloca ptr, align 8
  %11 = alloca i32, align 4
  %12 = alloca i32, align 4
  %13 = alloca i32, align 4
  %14 = alloca i32, align 4
  %15 = alloca i32, align 4
  %16 = alloca i32, align 4
  %17 = alloca i32, align 4
  %18 = alloca i32, align 4
  %19 = alloca i32, align 4
  %20 = alloca i32, align 4
  store ptr %0, ptr %6, align 8
  store ptr %1, ptr %7, align 8
  store ptr %2, ptr %8, align 8
  store ptr %3, ptr %9, align 8
  store ptr %4, ptr %10, align 8
  %21 = load ptr, ptr %8, align 8
  %22 = load ptr, ptr %9, align 8
  %23 = load ptr, ptr %10, align 8
  %24 = load i32, ptr %21, align 4
  store i32 %24, ptr %13, align 4
  %25 = load i32, ptr %13, align 4
  %26 = sub nsw i32 %25, 0
  %27 = sdiv i32 %26, 1
  %28 = sub nsw i32 %27, 1
  store i32 %28, ptr %14, align 4
  store i32 0, ptr %15, align 4
  %29 = load i32, ptr %13, align 4
  %30 = icmp slt i32 0, %29
  br i1 %30, label %31, label %66

31:                                               ; preds = %5
  store i32 0, ptr %16, align 4
  %32 = load i32, ptr %14, align 4
  store i32 %32, ptr %17, align 4
  store i32 1, ptr %18, align 4
  store i32 0, ptr %19, align 4
  %33 = load ptr, ptr %6, align 8
  %34 = load i32, ptr %33, align 4
  call void @__kmpc_for_static_init_4(ptr @1, i32 %34, i32 34, ptr %19, ptr %16, ptr %17, ptr %18, i32 1, i32 1)
  %35 = load i32, ptr %17, align 4
  %36 = load i32, ptr %14, align 4
  %37 = icmp sgt i32 %35, %36
  br i1 %37, label %38, label %40

38:                                               ; preds = %31
  %39 = load i32, ptr %14, align 4
  br label %42

40:                                               ; preds = %31
  %41 = load i32, ptr %17, align 4
  br label %42

42:                                               ; preds = %40, %38
  %43 = phi i32 [ %39, %38 ], [ %41, %40 ]
  store i32 %43, ptr %17, align 4
  %44 = load i32, ptr %16, align 4
  store i32 %44, ptr %11, align 4
  br label %45

45:                                               ; preds = %59, %42
  %46 = load i32, ptr %11, align 4
  %47 = load i32, ptr %17, align 4
  %48 = icmp sle i32 %46, %47
  br i1 %48, label %49, label %62

49:                                               ; preds = %45
  %50 = load i32, ptr %11, align 4
  %51 = mul nsw i32 %50, 1
  %52 = add nsw i32 0, %51
  store i32 %52, ptr %20, align 4
  %53 = load float, ptr %23, align 4
  %54 = load ptr, ptr %22, align 8
  %55 = load i32, ptr %20, align 4
  %56 = sext i32 %55 to i64
  %57 = getelementptr inbounds float, ptr %54, i64 %56
  store float %53, ptr %57, align 4
  br label %58

58:                                               ; preds = %49
  br label %59

59:                                               ; preds = %58
  %60 = load i32, ptr %11, align 4
  %61 = add nsw i32 %60, 1
  store i32 %61, ptr %11, align 4
  br label %45

62:                                               ; preds = %45
  br label %63

63:                                               ; preds = %62
  %64 = load ptr, ptr %6, align 8
  %65 = load i32, ptr %64, align 4
  call void @__kmpc_for_static_fini(ptr @1, i32 %65)
  br label %66

66:                                               ; preds = %63, %5
  %67 = load ptr, ptr %6, align 8
  %68 = load i32, ptr %67, align 4
  call void @__kmpc_barrier(ptr @2, i32 %68)
  ret void
}

; Function Attrs: nounwind
declare void @__kmpc_for_static_init_4(ptr, i32, i32, ptr, ptr, ptr, ptr, i32, i32) #2

; Function Attrs: nounwind
declare void @__kmpc_for_static_fini(ptr, i32) #2

; Function Attrs: convergent nounwind
declare void @__kmpc_barrier(ptr, i32) #3

; Function Attrs: nounwind
declare !callback !6 void @__kmpc_fork_call(ptr, i32, ptr, ...) #2

; Function Attrs: mustprogress noinline norecurse optnone ssp uwtable(sync)
define noundef i32 @main() #4 {
  %1 = alloca i32, align 4
  %2 = alloca [1000000 x float], align 4
  %3 = alloca i32, align 4
  store i32 0, ptr %1, align 4
  %4 = getelementptr inbounds [1000000 x float], ptr %2, i64 0, i64 0
  call void @_Z16initialize_arrayiPff(i32 noundef 1000000, ptr noundef %4, float noundef 0x4029D1EB80000000)
  store i32 0, ptr %3, align 4
  br label %5

5:                                                ; preds = %15, %0
  %6 = load i32, ptr %3, align 4
  %7 = icmp slt i32 %6, 10
  br i1 %7, label %8, label %18

8:                                                ; preds = %5
  %9 = load i32, ptr %3, align 4
  %10 = sext i32 %9 to i64
  %11 = getelementptr inbounds [1000000 x float], ptr %2, i64 0, i64 %10
  %12 = load float, ptr %11, align 4
  %13 = fpext float %12 to double
  %14 = call i32 (ptr, ...) @printf(ptr noundef @.str, double noundef %13)
  br label %15

15:                                               ; preds = %8
  %16 = load i32, ptr %3, align 4
  %17 = add nsw i32 %16, 1
  store i32 %17, ptr %3, align 4
  br label %5, !llvm.loop !8

18:                                               ; preds = %5
  %19 = load i32, ptr %1, align 4
  ret i32 %19
}

declare i32 @printf(ptr noundef, ...) #5

attributes #0 = { mustprogress noinline nounwind optnone ssp uwtable(sync) "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #1 = { noinline norecurse nounwind optnone ssp uwtable(sync) "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #2 = { nounwind }
attributes #3 = { convergent nounwind }
attributes #4 = { mustprogress noinline norecurse optnone ssp uwtable(sync) "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #5 = { "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }

!llvm.module.flags = !{!0, !1, !2, !3, !4}
!llvm.ident = !{!5}

!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{i32 7, !"openmp", i32 51}
!2 = !{i32 8, !"PIC Level", i32 2}
!3 = !{i32 7, !"uwtable", i32 1}
!4 = !{i32 7, !"frame-pointer", i32 1}
!5 = !{!"Homebrew clang version 18.1.4"}
!6 = !{!7}
!7 = !{i64 2, i64 -1, i64 -1, i1 true}
!8 = distinct !{!8, !9}
!9 = !{!"llvm.loop.mustprogress"}
```


</details>



Here you can see use of `__kmpc_for_static_init_4`, `__kmpc_for_static_fini` and `__kmpc_barrier`. So, we'll have to port it into `kmp.h` file and thus it looks like:

```cpp
typedef int kmp_int32;

typedef struct ident {
  kmp_int32 reserved_1; /**<  might be used in Fortran; see above  */
  kmp_int32 flags; /**<  also f.flags; KMP_IDENT_xxx flags; KMP_IDENT_KMPC
                      identifies this union member  */
  kmp_int32 reserved_2; /**<  not really used in Fortran any more; see above */
#if USE_ITT_BUILD
/*  but currently used for storing region-specific ITT */
/*  contextual information. */
#endif /* USE_ITT_BUILD */
  kmp_int32 reserved_3; /**< source[4] in Fortran, do not use for C++  */
  char const *psource; /**< String describing the source location.
                       The string is composed of semi-colon separated fields
                       which describe the source file, the function and a pair
                       of line numbers that delimit the construct. */
  // Returns the OpenMP version in form major*10+minor (e.g., 50 for 5.0)
//   kmp_int32 get_openmp_version() {
//     return (((flags & KMP_IDENT_OPENMP_SPEC_VERSION_MASK) >> 24) & 0xFF);
//   }
} ident_t;

typedef struct ident ident_t;
typedef void (*kmpc_micro)(kmp_int32 *global_tid, kmp_int32 *bound_tid, ...);

extern "C" {
/*!
@ingroup PARALLEL
@param loc  source location information
@param argc  total number of arguments in the ellipsis
@param microtask  pointer to callback routine consisting of outlined parallel
construct
@param ...  pointers to shared variables that aren't global

Do the actual fork and call the microtask in the relevant number of threads.
*/
    void __kmpc_fork_call(ident_t *, kmp_int32 nargs,
                                kmpc_micro microtask, ...);

/*
@param    loc       Source code location
@param    gtid      Global thread id of this thread
@param    schedtype  Scheduling type
@param    plastiter Pointer to the "last iteration" flag
@param    plower    Pointer to the lower bound
@param    pupper    Pointer to the upper bound
@param    pstride   Pointer to the stride
@param    incr      Loop increment
@param    chunk     The chunk size

The functions compute the upper and lower bounds and stride to be used for the
set of iterations to be executed by the current thread from the statically
scheduled loop that is described by the initial values of the bounds, stride,
increment and chunk size.
*/

    void __kmpc_for_static_init_4(ident_t *loc, kmp_int32 gtid, kmp_int32 schedtype,
                                  kmp_int32 *plastiter, kmp_int32 *plower,
                                  kmp_int32 *pupper, kmp_int32 *pstride,
                                  kmp_int32 incr, kmp_int32 chunk);

/*
Mark the end of a statically scheduled loop.
*/

    void __kmpc_for_static_fini(ident_t *loc, kmp_int32 global_tid);

/*
Execute a barrier.
*/

    void __kmpc_barrier(ident_t *loc, kmp_int32 global_tid);
}
```

Now, translating `llvm` code to `cpp` we get:

> c-transformed.cpp

```cpp
#include <stdio.h>
#include <omp.h>
#include "kmp.h"

void kmp_initialize_array( kmp_int32 *global_tid, kmp_int32 *bound_tid, int *n, float *arr, float *value ) {
    // @1 = private unnamed_addr constant %struct.ident_t { i32 0, i32 514, i32 0, i32 22, ptr @0 }, align 8
    ident_t loc; loc.reserved_1 = 0; loc.flags = 514; loc.reserved_2 = 0; loc.reserved_3 = 22; loc.psource = ";unknown;unknown;0;0;;\00";

    int *lastiter = new int(0);
    int *lower = new int(0);
    int *upper = new int(*n);
    int *stride = new int(1);
    __kmpc_for_static_init_4(&loc, *global_tid, 34, lastiter, lower, upper, stride, 1, 1);
    while ( *lower < *upper ) {
      // set the value of arr[*lower] to *value
      arr[*lower] = *value;
      *lower += 1;
    }
    __kmpc_for_static_fini(&loc, *global_tid);
    __kmpc_barrier(&loc, *global_tid);
    return;
}


void initialize_array( int n, float *a, float val ) {
    ident_t loc;

    /*
        @0 = private unnamed_addr constant [23 x i8] c";unknown;unknown;0;0;;\00", align 1
        @1 = private unnamed_addr constant %struct.ident_t { i32 0, i32 514, i32 0, i32 22, ptr @0 }, align 8
        @2 = private unnamed_addr constant %struct.ident_t { i32 0, i32 66, i32 0, i32 22, ptr @0 }, align 8
        @3 = private unnamed_addr constant %struct.ident_t { i32 0, i32 2, i32 0, i32 22, ptr @0 }, align 8

        ident_t for this case is @3
    */

    loc.reserved_1 = 0; loc.flags = 2; loc.reserved_2 = 0; loc.reserved_3 = 22; loc.psource = ";unknown;unknown;0;0;;";

    printf("value here = %f\n", val);
    __kmpc_fork_call(&loc, 3, (kmpc_micro)kmp_initialize_array, &n, a, &val);
}


int main() {
    float a[1000000];
    float val = 12.91;

    initialize_array(1000000, a, val);

    for (int i = 0; i < 10; i++) {
        printf("%f\n", a[i]);
    }
}
```

```console
% time clang++ c-transformed.cpp -fopenmp && time ./a.out
clang++ c-transformed.cpp -fopenmp  0.08s user 0.03s system 119% cpu 0.093 total
value here = 12.910000
12.910000
12.910000
12.910000
12.910000
12.910000
12.910000
12.910000
12.910000
12.910000
12.910000
./a.out  0.00s user 0.00s system 188% cpu 0.003 total
```

## Reduction

### openmp

```cpp
#include <stdio.h>
#include <omp.h>
#include "kmp.h"

void initialize_array( int n, float *a, float val ) {
    int i;
    #pragma omp parallel 
    {
        #pragma omp for
        for (i = 0; i < n; i++) {
            a[i] = val;
        }
    }

    #pragma omp barrier
    return;
}

void parallel_Sum_created( kmp_int32 *global_tid, kmp_int32 *bound_tid, float *total_Sum, int *n, float *a ) {
    // @1 = private unnamed_addr constant %struct.ident_t { i32 0, i32 514, i32 0, i32 22, ptr @0 }, align 8
    ident_t loc; loc.reserved_1 = 0; loc.flags = 514; loc.reserved_2 = 0; loc.reserved_3 = 22; loc.psource = ";unknown;unknown;0;0;;\00";
    // @2 = private unnamed_addr constant %struct.ident_t { i32 0, i32 66, i32 0, i32 22, ptr @0 }, align 8
    ident_t loc2; loc2.reserved_1 = 0; loc2.flags = 66; loc2.reserved_2 = 0; loc2.reserved_3 = 22; loc2.psource = ";unknown;unknown;0;0;;\00";
    float partial_sum = 0;
    int *lastiter = new int(0);
    int *lower = new int(0);
    int *upper = new int(*n);
    int *stride = new int(1);
    __kmpc_for_static_init_4(&loc, *global_tid, 34, lastiter, lower, upper, stride, 1, 1);
    while ( *lower < *upper ) {
        partial_sum += a[*lower];
        *lower += 1;
    }
    __kmpc_for_static_fini(&loc, *global_tid);
    __kmpc_barrier(&loc2, *global_tid);
    *total_Sum += partial_sum;
    printf("Thread %d: partial sum: %f total sum: %f\n", omp_get_thread_num(), partial_sum, *total_Sum);
    return;
} 

void parallel_Sum( int n, float *a ) {
    float total_Sum;

    // @3 = private unnamed_addr constant %struct.ident_t { i32 0, i32 2, i32 0, i32 22, ptr @0 }, align 8
    ident_t loc; loc.reserved_1 = 0; loc.flags = 2; loc.reserved_2 = 0; loc.reserved_3 = 22; loc.psource = ";unknown;unknown;0;0;;\00";
    __kmpc_fork_call(&loc, 3, (kmpc_micro)parallel_Sum_created, &total_Sum, &n, a);

    printf("Total sum: %f\n", total_Sum);
}

int main() {
    float a[1000000];
    initialize_array(1000000, a, 1.0);
    parallel_Sum(1000000, a);
}
```
```console
% time clang++ e.cpp -fopenmp && time ./a.out        
clang++ e.cpp -fopenmp  0.09s user 0.03s system 118% cpu 0.095 total
Total sum: 1000000.000000
./a.out  0.01s user 0.00s system 253% cpu 0.003 total
```

### transformed code

Beginning with `llvm`:

<details>

<summary> llvm </summary>

```llvm
; ModuleID = 'e.cpp'
source_filename = "e.cpp"
target datalayout = "e-m:o-i64:64-i128:128-n32:64-S128"
target triple = "arm64-apple-macosx14.0.0"

%struct.ident_t = type { i32, i32, i32, i32, ptr }

@0 = private unnamed_addr constant [23 x i8] c";unknown;unknown;0;0;;\00", align 1
@1 = private unnamed_addr constant %struct.ident_t { i32 0, i32 514, i32 0, i32 22, ptr @0 }, align 8
@2 = private unnamed_addr constant %struct.ident_t { i32 0, i32 66, i32 0, i32 22, ptr @0 }, align 8
@3 = private unnamed_addr constant %struct.ident_t { i32 0, i32 2, i32 0, i32 22, ptr @0 }, align 8
@.str = private unnamed_addr constant [15 x i8] c"Total sum: %f\0A\00", align 1
@.str.1 = private unnamed_addr constant [11 x i8] c"a[i] = %f\0A\00", align 1

; Function Attrs: mustprogress noinline nounwind optnone ssp uwtable(sync)
define void @_Z16initialize_arrayiPff(i32 noundef %0, ptr noundef %1, float noundef %2) #0 {
  %4 = alloca i32, align 4
  %5 = alloca ptr, align 8
  %6 = alloca float, align 4
  %7 = alloca i32, align 4
  store i32 %0, ptr %4, align 4
  store ptr %1, ptr %5, align 8
  store float %2, ptr %6, align 4
  call void (ptr, i32, ptr, ...) @__kmpc_fork_call(ptr @3, i32 3, ptr @_Z16initialize_arrayiPff.omp_outlined, ptr %4, ptr %5, ptr %6)
  ret void
}

; Function Attrs: noinline norecurse nounwind optnone ssp uwtable(sync)
define internal void @_Z16initialize_arrayiPff.omp_outlined(ptr noalias noundef %0, ptr noalias noundef %1, ptr noundef nonnull align 4 dereferenceable(4) %2, ptr noundef nonnull align 8 dereferenceable(8) %3, ptr noundef nonnull align 4 dereferenceable(4) %4) #1 {
  %6 = alloca ptr, align 8
  %7 = alloca ptr, align 8
  %8 = alloca ptr, align 8
  %9 = alloca ptr, align 8
  %10 = alloca ptr, align 8
  %11 = alloca i32, align 4
  %12 = alloca i32, align 4
  %13 = alloca i32, align 4
  %14 = alloca i32, align 4
  %15 = alloca i32, align 4
  %16 = alloca i32, align 4
  %17 = alloca i32, align 4
  %18 = alloca i32, align 4
  %19 = alloca i32, align 4
  %20 = alloca i32, align 4
  store ptr %0, ptr %6, align 8
  store ptr %1, ptr %7, align 8
  store ptr %2, ptr %8, align 8
  store ptr %3, ptr %9, align 8
  store ptr %4, ptr %10, align 8
  %21 = load ptr, ptr %8, align 8
  %22 = load ptr, ptr %9, align 8
  %23 = load ptr, ptr %10, align 8
  %24 = load i32, ptr %21, align 4
  store i32 %24, ptr %13, align 4
  %25 = load i32, ptr %13, align 4
  %26 = sub nsw i32 %25, 0
  %27 = sdiv i32 %26, 1
  %28 = sub nsw i32 %27, 1
  store i32 %28, ptr %14, align 4
  store i32 0, ptr %15, align 4
  %29 = load i32, ptr %13, align 4
  %30 = icmp slt i32 0, %29
  br i1 %30, label %31, label %66

31:                                               ; preds = %5
  store i32 0, ptr %16, align 4
  %32 = load i32, ptr %14, align 4
  store i32 %32, ptr %17, align 4
  store i32 1, ptr %18, align 4
  store i32 0, ptr %19, align 4
  %33 = load ptr, ptr %6, align 8
  %34 = load i32, ptr %33, align 4
  call void @__kmpc_for_static_init_4(ptr @1, i32 %34, i32 34, ptr %19, ptr %16, ptr %17, ptr %18, i32 1, i32 1)
  %35 = load i32, ptr %17, align 4
  %36 = load i32, ptr %14, align 4
  %37 = icmp sgt i32 %35, %36
  br i1 %37, label %38, label %40

38:                                               ; preds = %31
  %39 = load i32, ptr %14, align 4
  br label %42

40:                                               ; preds = %31
  %41 = load i32, ptr %17, align 4
  br label %42

42:                                               ; preds = %40, %38
  %43 = phi i32 [ %39, %38 ], [ %41, %40 ]
  store i32 %43, ptr %17, align 4
  %44 = load i32, ptr %16, align 4
  store i32 %44, ptr %11, align 4
  br label %45

45:                                               ; preds = %59, %42
  %46 = load i32, ptr %11, align 4
  %47 = load i32, ptr %17, align 4
  %48 = icmp sle i32 %46, %47
  br i1 %48, label %49, label %62

49:                                               ; preds = %45
  %50 = load i32, ptr %11, align 4
  %51 = mul nsw i32 %50, 1
  %52 = add nsw i32 0, %51
  store i32 %52, ptr %20, align 4
  %53 = load float, ptr %23, align 4
  %54 = load ptr, ptr %22, align 8
  %55 = load i32, ptr %20, align 4
  %56 = sext i32 %55 to i64
  %57 = getelementptr inbounds float, ptr %54, i64 %56
  store float %53, ptr %57, align 4
  br label %58

58:                                               ; preds = %49
  br label %59

59:                                               ; preds = %58
  %60 = load i32, ptr %11, align 4
  %61 = add nsw i32 %60, 1
  store i32 %61, ptr %11, align 4
  br label %45

62:                                               ; preds = %45
  br label %63

63:                                               ; preds = %62
  %64 = load ptr, ptr %6, align 8
  %65 = load i32, ptr %64, align 4
  call void @__kmpc_for_static_fini(ptr @1, i32 %65)
  br label %66

66:                                               ; preds = %63, %5
  %67 = load ptr, ptr %6, align 8
  %68 = load i32, ptr %67, align 4
  call void @__kmpc_barrier(ptr @2, i32 %68)
  ret void
}

; Function Attrs: nounwind
declare void @__kmpc_for_static_init_4(ptr, i32, i32, ptr, ptr, ptr, ptr, i32, i32) #2

; Function Attrs: nounwind
declare void @__kmpc_for_static_fini(ptr, i32) #2

; Function Attrs: convergent nounwind
declare void @__kmpc_barrier(ptr, i32) #3

; Function Attrs: nounwind
declare !callback !6 void @__kmpc_fork_call(ptr, i32, ptr, ...) #2

; Function Attrs: mustprogress noinline optnone ssp uwtable(sync)
define void @_Z12parallel_SumiPf(i32 noundef %0, ptr noundef %1) #4 {
  %3 = alloca i32, align 4
  %4 = alloca ptr, align 8
  %5 = alloca i32, align 4
  %6 = alloca float, align 4
  %7 = alloca float, align 4
  store i32 %0, ptr %3, align 4
  store ptr %1, ptr %4, align 8
  call void (ptr, i32, ptr, ...) @__kmpc_fork_call(ptr @3, i32 3, ptr @_Z12parallel_SumiPf.omp_outlined, ptr %7, ptr %3, ptr %4)
  %8 = load float, ptr %7, align 4
  %9 = fpext float %8 to double
  %10 = call i32 (ptr, ...) @printf(ptr noundef @.str, double noundef %9)
  ret void
}

; Function Attrs: noinline norecurse nounwind optnone ssp uwtable(sync)
define internal void @_Z12parallel_SumiPf.omp_outlined(ptr noalias noundef %0, ptr noalias noundef %1, ptr noundef nonnull align 4 dereferenceable(4) %2, ptr noundef nonnull align 4 dereferenceable(4) %3, ptr noundef nonnull align 8 dereferenceable(8) %4) #1 {
  %6 = alloca ptr, align 8
  %7 = alloca ptr, align 8
  %8 = alloca ptr, align 8 // total_Sum
  %9 = alloca ptr, align 8 // n
  %10 = alloca ptr, align 8 // a
  %11 = alloca float, align 4
  %12 = alloca i32, align 4
  %13 = alloca i32, align 4
  %14 = alloca i32, align 4
  %15 = alloca i32, align 4
  %16 = alloca i32, align 4
  %17 = alloca i32, align 4
  %18 = alloca i32, align 4
  %19 = alloca i32, align 4
  %20 = alloca i32, align 4
  %21 = alloca i32, align 4
  store ptr %0, ptr %6, align 8
  store ptr %1, ptr %7, align 8
  store ptr %2, ptr %8, align 8 // total_Sum
  store ptr %3, ptr %9, align 8 // n
  store ptr %4, ptr %10, align 8 // a
  %22 = load ptr, ptr %8, align 8
  %23 = load ptr, ptr %9, align 8
  %24 = load ptr, ptr %10, align 8
  store float 0.000000e+00, ptr %11, align 4 // partial_Sum = 0.0
  store float 0.000000e+00, ptr %22, align 4 // total_Sum = 0.0
  %25 = load i32, ptr %23, align 4
  store i32 %25, ptr %14, align 4
  %26 = load i32, ptr %14, align 4
  %27 = sub nsw i32 %26, 0
  %28 = sdiv i32 %27, 1
  %29 = sub nsw i32 %28, 1
  store i32 %29, ptr %15, align 4
  store i32 0, ptr %16, align 4
  %30 = load i32, ptr %14, align 4
  %31 = icmp slt i32 0, %30
  br i1 %31, label %32, label %69

32:                                               ; preds = %5
  store i32 0, ptr %17, align 4
  %33 = load i32, ptr %15, align 4
  store i32 %33, ptr %18, align 4
  store i32 1, ptr %19, align 4
  store i32 0, ptr %20, align 4
  %34 = load ptr, ptr %6, align 8
  %35 = load i32, ptr %34, align 4
  call void @__kmpc_for_static_init_4(ptr @1, i32 %35, i32 34, ptr %20, ptr %17, ptr %18, ptr %19, i32 1, i32 1)
  %36 = load i32, ptr %18, align 4
  %37 = load i32, ptr %15, align 4
  %38 = icmp sgt i32 %36, %37
  br i1 %38, label %39, label %41

39:                                               ; preds = %32
  %40 = load i32, ptr %15, align 4
  br label %43

41:                                               ; preds = %32
  %42 = load i32, ptr %18, align 4
  br label %43

43:                                               ; preds = %41, %39
  %44 = phi i32 [ %40, %39 ], [ %42, %41 ]
  store i32 %44, ptr %18, align 4
  %45 = load i32, ptr %17, align 4
  store i32 %45, ptr %12, align 4
  br label %46

46:                                               ; preds = %62, %43
  %47 = load i32, ptr %12, align 4
  %48 = load i32, ptr %18, align 4
  %49 = icmp sle i32 %47, %48
  br i1 %49, label %50, label %65

50:                                               ; preds = %46
  %51 = load i32, ptr %12, align 4
  %52 = mul nsw i32 %51, 1
  %53 = add nsw i32 0, %52
  store i32 %53, ptr %21, align 4
  %54 = load ptr, ptr %24, align 8
  %55 = load i32, ptr %21, align 4
  %56 = sext i32 %55 to i64
  %57 = getelementptr inbounds float, ptr %54, i64 %56
  %58 = load float, ptr %57, align 4
  %59 = load float, ptr %11, align 4
  %60 = fadd float %59, %58
  store float %60, ptr %11, align 4
  br label %61

61:                                               ; preds = %50
  br label %62

62:                                               ; preds = %61
  %63 = load i32, ptr %12, align 4
  %64 = add nsw i32 %63, 1
  store i32 %64, ptr %12, align 4
  br label %46

65:                                               ; preds = %46
  br label %66

66:                                               ; preds = %65
  %67 = load ptr, ptr %6, align 8
  %68 = load i32, ptr %67, align 4
  call void @__kmpc_for_static_fini(ptr @1, i32 %68)
  br label %69

69:                                               ; preds = %66, %5
  %70 = load ptr, ptr %6, align 8
  %71 = load i32, ptr %70, align 4
  call void @__kmpc_barrier(ptr @2, i32 %71)
  %72 = load float, ptr %11, align 4
  %73 = load float, ptr %22, align 4
  %74 = fadd float %73, %72
  store float %74, ptr %22, align 4
  ret void
}

declare i32 @printf(ptr noundef, ...) #5

; Function Attrs: mustprogress noinline nounwind optnone ssp uwtable(sync)
define void @_Z5checkiPff(i32 noundef %0, ptr noundef %1, float noundef %2) #0 {
  %4 = alloca i32, align 4
  %5 = alloca ptr, align 8
  %6 = alloca float, align 4
  %7 = alloca i32, align 4
  store i32 %0, ptr %4, align 4
  store ptr %1, ptr %5, align 8
  store float %2, ptr %6, align 4
  call void (ptr, i32, ptr, ...) @__kmpc_fork_call(ptr @3, i32 3, ptr @_Z5checkiPff.omp_outlined, ptr %4, ptr %5, ptr %6)
  ret void
}

; Function Attrs: noinline norecurse nounwind optnone ssp uwtable(sync)
define internal void @_Z5checkiPff.omp_outlined(ptr noalias noundef %0, ptr noalias noundef %1, ptr noundef nonnull align 4 dereferenceable(4) %2, ptr noundef nonnull align 8 dereferenceable(8) %3, ptr noundef nonnull align 4 dereferenceable(4) %4) #1 personality ptr @__gxx_personality_v0 {
  %6 = alloca ptr, align 8
  %7 = alloca ptr, align 8
  %8 = alloca ptr, align 8
  %9 = alloca ptr, align 8
  %10 = alloca ptr, align 8
  %11 = alloca i32, align 4
  %12 = alloca i32, align 4
  %13 = alloca i32, align 4
  %14 = alloca i32, align 4
  %15 = alloca i32, align 4
  %16 = alloca i32, align 4
  %17 = alloca i32, align 4
  %18 = alloca i32, align 4
  %19 = alloca i32, align 4
  %20 = alloca i32, align 4
  store ptr %0, ptr %6, align 8
  store ptr %1, ptr %7, align 8
  store ptr %2, ptr %8, align 8
  store ptr %3, ptr %9, align 8
  store ptr %4, ptr %10, align 8
  %21 = load ptr, ptr %8, align 8
  %22 = load ptr, ptr %9, align 8
  %23 = load ptr, ptr %10, align 8
  %24 = load i32, ptr %21, align 4
  store i32 %24, ptr %13, align 4
  %25 = load i32, ptr %13, align 4
  %26 = sub nsw i32 %25, 0
  %27 = sdiv i32 %26, 1
  %28 = sub nsw i32 %27, 1
  store i32 %28, ptr %14, align 4
  store i32 0, ptr %15, align 4
  %29 = load i32, ptr %13, align 4
  %30 = icmp slt i32 0, %29
  br i1 %30, label %31, label %78

31:                                               ; preds = %5
  store i32 0, ptr %16, align 4
  %32 = load i32, ptr %14, align 4
  store i32 %32, ptr %17, align 4
  store i32 1, ptr %18, align 4
  store i32 0, ptr %19, align 4
  %33 = load ptr, ptr %6, align 8
  %34 = load i32, ptr %33, align 4
  call void @__kmpc_for_static_init_4(ptr @1, i32 %34, i32 34, ptr %19, ptr %16, ptr %17, ptr %18, i32 1, i32 1)
  %35 = load i32, ptr %17, align 4
  %36 = load i32, ptr %14, align 4
  %37 = icmp sgt i32 %35, %36
  br i1 %37, label %38, label %40

38:                                               ; preds = %31
  %39 = load i32, ptr %14, align 4
  br label %42

40:                                               ; preds = %31
  %41 = load i32, ptr %17, align 4
  br label %42

42:                                               ; preds = %40, %38
  %43 = phi i32 [ %39, %38 ], [ %41, %40 ]
  store i32 %43, ptr %17, align 4
  %44 = load i32, ptr %16, align 4
  store i32 %44, ptr %11, align 4
  br label %45

45:                                               ; preds = %71, %42
  %46 = load i32, ptr %11, align 4
  %47 = load i32, ptr %17, align 4
  %48 = icmp sle i32 %46, %47
  br i1 %48, label %49, label %74

49:                                               ; preds = %45
  %50 = load i32, ptr %11, align 4
  %51 = mul nsw i32 %50, 1
  %52 = add nsw i32 0, %51
  store i32 %52, ptr %20, align 4
  %53 = load ptr, ptr %22, align 8
  %54 = load i32, ptr %20, align 4
  %55 = sext i32 %54 to i64
  %56 = getelementptr inbounds float, ptr %53, i64 %55
  %57 = load float, ptr %56, align 4
  %58 = load float, ptr %23, align 4
  %59 = fcmp une float %57, %58
  br i1 %59, label %60, label %69

60:                                               ; preds = %49
  %61 = load ptr, ptr %22, align 8
  %62 = load i32, ptr %20, align 4
  %63 = sext i32 %62 to i64
  %64 = getelementptr inbounds float, ptr %61, i64 %63
  %65 = load float, ptr %64, align 4
  %66 = fpext float %65 to double
  %67 = invoke i32 (ptr, ...) @printf(ptr noundef @.str.1, double noundef %66)
          to label %68 unwind label %81

68:                                               ; preds = %60
  br label %69

69:                                               ; preds = %68, %49
  br label %70

70:                                               ; preds = %69
  br label %71

71:                                               ; preds = %70
  %72 = load i32, ptr %11, align 4
  %73 = add nsw i32 %72, 1
  store i32 %73, ptr %11, align 4
  br label %45

74:                                               ; preds = %45
  br label %75

75:                                               ; preds = %74
  %76 = load ptr, ptr %6, align 8
  %77 = load i32, ptr %76, align 4
  call void @__kmpc_for_static_fini(ptr @1, i32 %77)
  br label %78

78:                                               ; preds = %75, %5
  %79 = load ptr, ptr %6, align 8
  %80 = load i32, ptr %79, align 4
  call void @__kmpc_barrier(ptr @2, i32 %80)
  ret void

81:                                               ; preds = %60
  %82 = landingpad { ptr, i32 }
          catch ptr null
  %83 = extractvalue { ptr, i32 } %82, 0
  call void @__clang_call_terminate(ptr %83) #8
  unreachable
}

declare i32 @__gxx_personality_v0(...)

; Function Attrs: noinline noreturn nounwind ssp uwtable(sync)
define linkonce_odr hidden void @__clang_call_terminate(ptr noundef %0) #6 {
  %2 = call ptr @__cxa_begin_catch(ptr %0) #2
  call void @_ZSt9terminatev() #8
  unreachable
}

declare ptr @__cxa_begin_catch(ptr)

declare void @_ZSt9terminatev()

; Function Attrs: mustprogress noinline norecurse optnone ssp uwtable(sync)
define noundef i32 @main() #7 {
  %1 = alloca i32, align 4
  %2 = alloca [1000000 x float], align 4
  store i32 0, ptr %1, align 4
  %3 = getelementptr inbounds [1000000 x float], ptr %2, i64 0, i64 0
  call void @_Z16initialize_arrayiPff(i32 noundef 1000000, ptr noundef %3, float noundef 1.000000e+00)
  %4 = getelementptr inbounds [1000000 x float], ptr %2, i64 0, i64 0
  call void @_Z12parallel_SumiPf(i32 noundef 1000000, ptr noundef %4)
  %5 = getelementptr inbounds [1000000 x float], ptr %2, i64 0, i64 0
  call void @_Z5checkiPff(i32 noundef 1000000, ptr noundef %5, float noundef 1.000000e+00)
  ret i32 0
}

attributes #0 = { mustprogress noinline nounwind optnone ssp uwtable(sync) "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #1 = { noinline norecurse nounwind optnone ssp uwtable(sync) "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #2 = { nounwind }
attributes #3 = { convergent nounwind }
attributes #4 = { mustprogress noinline optnone ssp uwtable(sync) "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #5 = { "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #6 = { noinline noreturn nounwind ssp uwtable(sync) "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #7 = { mustprogress noinline norecurse optnone ssp uwtable(sync) "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #8 = { noreturn nounwind }

!llvm.module.flags = !{!0, !1, !2, !3, !4}
!llvm.ident = !{!5}

!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{i32 7, !"openmp", i32 51}
!2 = !{i32 8, !"PIC Level", i32 2}
!3 = !{i32 7, !"uwtable", i32 1}
!4 = !{i32 7, !"frame-pointer", i32 1}
!5 = !{!"Homebrew clang version 18.1.4"}
!6 = !{!7}
!7 = !{i64 2, i64 -1, i64 -1, i1 true}
```

</details>


No use of any new llvm openmp routines and thus we can translate it as shown:

```cpp
#include <stdio.h>
#include <omp.h>
#include "kmp.h"

void initialize_array( int n, float *a, float val ) {
    int i;
    #pragma omp parallel 
    {
        #pragma omp for
        for (i = 0; i < n; i++) {
            a[i] = val;
        }
    }

    #pragma omp barrier
    return;
}

void parallel_Sum_created( kmp_int32 *global_tid, kmp_int32 *bound_tid, float *total_Sum, int *n, float *a ) {
    // @1 = private unnamed_addr constant %struct.ident_t { i32 0, i32 514, i32 0, i32 22, ptr @0 }, align 8
    ident_t loc; loc.reserved_1 = 0; loc.flags = 514; loc.reserved_2 = 0; loc.reserved_3 = 22; loc.psource = ";unknown;unknown;0;0;;\00";
    // @2 = private unnamed_addr constant %struct.ident_t { i32 0, i32 66, i32 0, i32 22, ptr @0 }, align 8
    ident_t loc2; loc2.reserved_1 = 0; loc2.flags = 66; loc2.reserved_2 = 0; loc2.reserved_3 = 22; loc2.psource = ";unknown;unknown;0;0;;\00";
    float partial_sum = 0;
    int *lastiter = new int(0);
    int *lower = new int(0);
    int *upper = new int(*n);
    int *stride = new int(1);
    __kmpc_for_static_init_4(&loc, *global_tid, 34, lastiter, lower, upper, stride, 1, 1);
    while ( *lower < *upper ) {
        partial_sum += a[*lower];
        *lower += 1;
    }
    __kmpc_for_static_fini(&loc, *global_tid);
    __kmpc_barrier(&loc2, *global_tid);
    *total_Sum += partial_sum;
    printf("Thread %d: partial sum: %f total sum: %f\n", omp_get_thread_num(), partial_sum, *total_Sum);
    return;
} 

void parallel_Sum( int n, float *a ) {
    float total_Sum;

    // @3 = private unnamed_addr constant %struct.ident_t { i32 0, i32 2, i32 0, i32 22, ptr @0 }, align 8
    ident_t loc; loc.reserved_1 = 0; loc.flags = 2; loc.reserved_2 = 0; loc.reserved_3 = 22; loc.psource = ";unknown;unknown;0;0;;\00";
    __kmpc_fork_call(&loc, 3, (kmpc_micro)parallel_Sum_created, &total_Sum, &n, a);

    printf("Total sum: %f\n", total_Sum);
}

int main() {
    float a[1000000];
    initialize_array(1000000, a, 1.0);
    parallel_Sum(1000000, a);
}
```


```console
% time clang++ e-transformed.cpp -fopenmp && time ./a.out
clang++ e-transformed.cpp -fopenmp  0.09s user 0.03s system 120% cpu 0.095 total
Thread 2: partial sum: 124999.000000 total sum: 374999.000000
Thread 0: partial sum: 125000.000000 total sum: 374999.000000
Thread 3: partial sum: 124999.000000 total sum: 374999.000000
Thread 4: partial sum: 124999.000000 total sum: 499998.000000
Thread 7: partial sum: 124999.000000 total sum: 624997.000000
Thread 1: partial sum: 124999.000000 total sum: 749996.000000
Thread 6: partial sum: 124999.000000 total sum: 874995.000000
Thread 5: partial sum: 124999.000000 total sum: 999994.000000
Total sum: 999994.000000
./a.out  0.01s user 0.00s system 206% cpu 0.004 total

```

Although, there looks slight divergence in value that needs to be debugged and we need to fully understand how is `#pragma critical` being handled in llvm openmp runtime routine calls.

----

That's it for this blog, we'll continue to push and get LFortran support openmp code :)
