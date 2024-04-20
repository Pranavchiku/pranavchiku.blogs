+++
title = 'LFortran Openmp'
date = 2024-04-17T20:41:15+05:30
draft = false
+++

## Introduction

The next step in development of LFortran is to add support for `openmp`, we have to support it in AST and ASR.

## Approach

Let's take the following and document approach on how shall we work on this.

```fortran
subroutine simple(n, a, b)
integer :: i, n
real :: a(n), b(n)

!$omp parallel private(i)
!$omp do
do i = 2, n
    b(i) = (a(i) + a(i-1)) / 2.0
end do
!$omp end do

!$omp end parallel

end subroutine
```

### GFortran parse tree

When we do `gfortran e.f90 -fopenmp -fdump-tree-all`, in `a-e.f90.017t.ompexp` we get the following 

<details>


<summary> GFortran Parse tree </summary>

```
OMP region tree

bb 2: gimple_omp_parallel
    bb 3: gimple_omp_for
    bb 4: GIMPLE_OMP_CONTINUE
    bb 5: GIMPLE_OMP_RETURN
bb 6: GIMPLE_OMP_RETURN

Added new low gimple function simple_._omp_fn.0 to callgraph
Introduced new external node (__builtin_omp_get_num_threads/2).
Introduced new external node (__builtin_omp_get_thread_num/3).

;; Function simple_._omp_fn.0 (simple_._omp_fn.0, funcdef_no=1, decl_uid=3028, cgraph_uid=2, symbol_order=1)

void simple_._omp_fn.0 (struct .omp_data_s.8 & restrict .omp_data_i)
{
  real(kind=4)[0:D.3037] * restrict b [value-expr: .omp_data_i->b];
  real(kind=4)[0:D.3038] * restrict a [value-expr: .omp_data_i->a];
  integer(kind=4) & restrict n [value-expr: .omp_data_i->n];
  real(kind=4)[0:D.3037] * D.3091;
  real(kind=4) D.3090;
  integer(kind=8) D.3089;
  integer(kind=8) D.3088;
  real(kind=4) D.3087;
  real(kind=4) D.3086;
  real(kind=4) D.3085;
  real(kind=4)[0:D.3038] * D.3084;
  integer(kind=8) D.3083;
  integer(kind=8) D.3082;
  integer(kind=4) D.3081;
  real(kind=4) D.3080;
  real(kind=4)[0:D.3038] * D.3079;
  integer(kind=8) D.3078;
  integer(kind=8) D.3077;
  integer(kind=4) D.3076;
  integer(kind=4) i;
  integer(kind=4) D.3074;
  integer(kind=4) D.3073;
  integer(kind=4) D.3072;
  integer(kind=4) tt.11;
  integer(kind=4) q.10;
  integer(kind=4) D.3069;
  integer(kind=4) D.3068;
  integer(kind=4) D.3067;
  integer(kind=4) D.3066;
  integer(kind=4) D.3065;
  integer(kind=4) D.3064;
  integer(kind=4) & D.3063;

  <bb 11> :

  <bb 3> :
  D.3063 = .omp_data_i->n; // n address
  D.3064 = *D.3063; // n value
  D.3065 = D.3064; // n value
  D.3066 = __builtin_omp_get_num_threads (); // num_threads
  D.3067 = __builtin_omp_get_thread_num (); // thread_id
  D.3068 = D.3065 + 1; // n + 1
  D.3069 = D.3068 + -2; // n - 1
  q.10 = D.3069 / D.3066; // n - 1 / num_threads --> how much portion to each thread
  tt.11 = D.3069 % D.3066; // n - 1 % num_threads --> portion left
  if (D.3067 < tt.11) // --> not sure what this is doing
    goto <bb 9>; [25.00%]
  else
    goto <bb 10>; [75.00%]

  <bb 10> :
  D.3072 = q.10 * D.3067; // chunk * thread_id
  D.3073 = D.3072 + tt.11;
  D.3074 = D.3073 + q.10;
  if (D.3073 >= D.3074)
    goto <bb 5>; [INV]
  else
    goto <bb 8>; [INV]

  <bb 8> :
  i = D.3073 + 2;
  D.3076 = D.3074 + 2;

  <bb 4> :
  D.3077 = (integer(kind=8)) i;
  D.3078 = D.3077 + -1;
  D.3079 = .omp_data_i->a; // a
  D.3080 = MEM <real(kind=4)[0:D.3040]> [(real(kind=4)[0:D.3040] *)D.3079][D.3078]; // a(i)
  D.3081 = i + -1; // i - 1
  D.3082 = (integer(kind=8)) D.3081;
  D.3083 = D.3082 + -1;
  D.3084 = .omp_data_i->a;
  D.3085 = MEM <real(kind=4)[0:D.3040]> [(real(kind=4)[0:D.3040] *)D.3084][D.3083]; // a(i-1)
  D.3086 = D.3080 + D.3085; // a(i) + a(i-1)
  D.3087 = ((D.3086));
  D.3088 = (integer(kind=8)) i;
  D.3089 = D.3088 + -1;
  D.3090 = D.3087 / 2.0e+0; // (a(i) + a(i-1)) / 2.0
  D.3091 = .omp_data_i->b; // b
  MEM <real(kind=4)[0:D.3041]> [(real(kind=4)[0:D.3041] *)D.3091][D.3089] = D.3090; // b = (a(i) + a(i-1)) / 2.0
  i = i + 1;
  if (i < D.3076) // if iterator less than a chunk in thread
    goto <bb 4>; [INV]
  else
    goto <bb 5>; [INV]

  <bb 5> :

  <bb 6> :
  return;

  <bb 9> :
  tt.11 = 0;
  q.10 = q.10 + 1;
  goto <bb 10>; [100.00%]
}

......

// called as 
D.3011 = size.7_8 * 4;
.omp_data_o.9.n = n;
.omp_data_o.9.a = a;
.omp_data_o.9.b = b;
__builtin_GOMP_parallel (simple_._omp_fn.0, &.omp_data_o.9, 0, 0);
.omp_data_o.9 = {CLOBBER};
```


</details>

### OpenMP APIs


In runtime library, we can see these functions `0000000000073128 T _GOMP_parallel`, `00000000000805d0 T _omp_get_num_threads` and `0000000000080ae8 T _omp_get_thread_num` which can be simply C interfaces with arguments as ( `CPtr`? )


I have added comments to understand what `gfortran` guys are doing, although I did not get what they are doing at `if (D.3067 < tt.11)`. Rest code logic is simply splitting range ( 2, n ) into `num_threads` and then applying loop normally. All these inside `__builting_GOMP_parallel`.

## DoConcurrent version

Let's take `DoConcurrent` version of this and brainstorm design for pass.

DoConcurrent version of this will look like

```fortran
subroutine simple(n, a, b)
integer :: i, n
real :: a(n), b(n)

do concurrent (i = 2:n)
b(i) = (a(i) + a(i-1)) / 2.0
end do

end subroutine
```

## Compiler Flag

We may need to add a compiler flag `--openmp` to link, if enabled will execute the pass, else skip it.

We do have one `--openmp Enable openmp`, that is good.

## Pass `replace_openmp`

> Pass: `replace_openmp`

`DoConcurrent` is a statement, we will create a simple `ASR::BaseWalkVisitor` with special handling for `DoConcurrent` statements.

`visit_DoConcurrent` will do the following:
- will trap all do concurrent loops
- identify `private` and `shared` variables
- create function like `_omp_fn` and populate it
- create function `_GOMP_parallel` ( BindC ) with required arguments
- add a new subroutine call to `_GOMP_parallel` in body

## Execution and Testing

To test it in integration test, we can generate executable using

```
lfortran -c a.f90 -o a.o
```

and then link `libopenmp.dylib`, LFortran runtime library and execute it. We can follow `symbolics` there.
