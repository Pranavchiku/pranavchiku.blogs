+++
title = 'Openmp transformation using llvm-openmp runtime routines'
date = 2024-05-02T23:18:30+05:30
draft = false
+++

## Introduction

Following the plan outlined at [lfortran#3777](https://github.com/lfortran/lfortran/issues/3777#issuecomment-2079723728), next step is to figure out how to do two examples ( array assignments and parallel sum ) using the LLVM OpenMP routines.

On page 12 of [llvm-openmp-reference](https://raw.githubusercontent.com/llvm/llvm-project/main/openmp/runtime/doc/Reference.pdf), llvm guys have mentioned how to translate a simple openmp code to cpp code using LLVM OpenMp routines. Although, it is too complex to understand, we'll start with a small example.

## Example 1

Let's begin with fairly small example that prints corresponding thread it:

### openmp pragma

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

```console
% time clang++ a.cpp -fopenmp && time ./a.out
clang++ a.cpp -fopenmp  0.07s user 0.03s system 97% cpu 0.097 total
omp_get_max_threads(): 8
Hello from thread: 0
Hello from thread: 4
Hello from thread: 6
Hello from thread: 5
Hello from thread: 3
Hello from thread: 2
Hello from thread: 7
Hello from thread: 1
./a.out  0.00s user 0.00s system 75% cpu 0.003 total
```

#### translated code

To write the code that uses llvm openmp routines, we first need to understand how this gets compiled, and this can be done by emitting llvm. To do this, one shall use the following code:

```
clang++ -S -emit-llvm a.cpp -fopenmp
```

Generated llvm looks like:

> a.ll

<details>


<summary> llvm </summary>

```llvm
; ModuleID = 'a.cpp'
source_filename = "a.cpp"
target datalayout = "e-m:o-i64:64-i128:128-n32:64-S128"
target triple = "arm64-apple-macosx14.0.0"

%struct.ident_t = type { i32, i32, i32, i32, ptr }

@.str = private unnamed_addr constant [27 x i8] c"omp_get_max_threads(): %d\0A\00", align 1
@.str.1 = private unnamed_addr constant [23 x i8] c"Hello from thread: %d\0A\00", align 1
@0 = private unnamed_addr constant [23 x i8] c";unknown;unknown;0;0;;\00", align 1
@1 = private unnamed_addr constant %struct.ident_t { i32 0, i32 2, i32 0, i32 22, ptr @0 }, align 8

; Function Attrs: mustprogress noinline norecurse optnone ssp uwtable(sync)
define noundef i32 @main() #0 {
  %1 = alloca i32, align 4
  %2 = alloca i32, align 4
  store i32 0, ptr %1, align 4
  %3 = call i32 @omp_get_max_threads()
  %4 = call i32 (ptr, ...) @printf(ptr noundef @.str, i32 noundef %3)
  call void (ptr, i32, ptr, ...) @__kmpc_fork_call(ptr @1, i32 0, ptr @main.omp_outlined)
  ret i32 0
}

declare i32 @printf(ptr noundef, ...) #1

declare i32 @omp_get_max_threads() #1

; Function Attrs: noinline norecurse nounwind optnone ssp uwtable(sync)
define internal void @main.omp_outlined(ptr noalias noundef %0, ptr noalias noundef %1) #2 personality ptr @__gxx_personality_v0 {
  %3 = alloca ptr, align 8
  %4 = alloca ptr, align 8
  %5 = alloca i32, align 4
  store ptr %0, ptr %3, align 8
  store ptr %1, ptr %4, align 8
  %6 = invoke i32 @omp_get_thread_num()
          to label %7 unwind label %11

7:                                                ; preds = %2
  store i32 %6, ptr %5, align 4
  %8 = load i32, ptr %5, align 4
  %9 = invoke i32 (ptr, ...) @printf(ptr noundef @.str.1, i32 noundef %8)
          to label %10 unwind label %11

10:                                               ; preds = %7
  ret void

11:                                               ; preds = %7, %2
  %12 = landingpad { ptr, i32 }
          catch ptr null
  %13 = extractvalue { ptr, i32 } %12, 0
  call void @__clang_call_terminate(ptr %13) #5
  unreachable
}

declare i32 @omp_get_thread_num() #1

declare i32 @__gxx_personality_v0(...)

; Function Attrs: noinline noreturn nounwind ssp uwtable(sync)
define linkonce_odr hidden void @__clang_call_terminate(ptr noundef %0) #3 {
  %2 = call ptr @__cxa_begin_catch(ptr %0) #4
  call void @_ZSt9terminatev() #5
  unreachable
}

declare ptr @__cxa_begin_catch(ptr)

declare void @_ZSt9terminatev()

; Function Attrs: nounwind
declare !callback !6 void @__kmpc_fork_call(ptr, i32, ptr, ...) #4

attributes #0 = { mustprogress noinline norecurse optnone ssp uwtable(sync) "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #1 = { "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #2 = { noinline norecurse nounwind optnone ssp uwtable(sync) "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #3 = { noinline noreturn nounwind ssp uwtable(sync) "frame-pointer"="non-leaf" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="apple-m1" "target-features"="+aes,+complxnum,+crc,+dotprod,+fp-armv8,+fp16fml,+fullfp16,+jsconv,+lse,+neon,+pauth,+ras,+rcpc,+rdm,+sha2,+sha3,+v8.1a,+v8.2a,+v8.3a,+v8.4a,+v8.5a,+v8a,+zcm,+zcz" }
attributes #4 = { nounwind }
attributes #5 = { noreturn nounwind }

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

This makes it easy to understand how code is being translated internally.

We can see calls to internal routines i.e. `__kmpc_fork_call`, which is declared in `kmp.h` file and due to some reasons we cannot directly include it. So I decided to port in required routine functions from `kmp.h` and create a new file `kmp.h` in same directory and later we can include it in `a-transformed.cpp`

> kmp.h

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
}
```

Now with this, we can transform the code as shown:

> a-transformed.cpp

```cpp
#include <stdio.h>
#include <omp.h>
#include "kmp.h"

void foo( void ) {
    int thread_id = omp_get_thread_num();
    printf("Hello from thread: %d\n", thread_id);
}

int main() {
    int thread_id;

    printf("omp_get_max_threads(): %d\n", omp_get_max_threads());

    ident_t loc;

    /*
        This is how ident is set in llvm:
        @0 = private unnamed_addr constant [23 x i8] c";unknown;unknown;0;0;;\00", align 1
        @1 = private unnamed_addr constant %struct.ident_t { i32 0, i32 2, i32 0, i32 22, ptr @0 }, align 8
    */

    loc.reserved_1 = 0;
    loc.flags = 2;
    loc.reserved_2 = 0;
    loc.reserved_3 = 22;
    loc.psource = ";unknown;unknown;0;0;;";

    __kmpc_fork_call(&loc, 1, (kmpc_micro)foo);

    return 0;
}

```

and this can be run using:

```console
% time clang++ a-transformed.cpp /Users/pranavchiku/repos/llvm-project/openmp/build/runtime/src/libomp.dylib -Wl,-rpath,/Users/pranavchiku/repos/llvm-project/openmp/build/runtime/src/ && time ./a.out
clang++ a-transformed.cpp    0.07s user 0.07s system 39% cpu 0.362 total
omp_get_max_threads(): 8
Hello from thread: 4
Hello from thread: 0
Hello from thread: 2
Hello from thread: 1
Hello from thread: 3
Hello from thread: 6
Hello from thread: 5
Hello from thread: 7
./a.out  0.00s user 0.00s system 58% cpu 0.005 total
```

Tadaaa! we now know how to use llvm openmp routines in cpp and get it working.

---

In next blog, I'll increase complexity of examples and explain how it functions.

---

## Read also

- [Openmp transformation using llvm-openmp runtime routines-2](https://pranavchiku.github.io/pranavchiku.blogs/openmp-blog/openmp-llvm-apis-2/)
