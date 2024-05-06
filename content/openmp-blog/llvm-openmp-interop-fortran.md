+++
title = 'LLVM Openmp Interop Fortran'
date = 2024-05-06T15:46:50+05:30
draft = false
+++

## Introduction

Moving ahead to fortran, we got `llvm-openmp` APIs working in C, now is time to write interface and interop modue in fortran and get it initially working with GFortran. In this blog, I'll discuss a small working example.

## Example

### openmp pragma

```fortran
program openmp
use omp_lib

integer :: thread_id

!$omp parallel private(thread_id)

thread_id = omp_get_thread_num()
print *, "hello from process: ", thread_id
!$omp end parallel
end program
```

```console
% gfortran a.f90 -fopenmp && ./a.out
 hello from process:            1
 hello from process:            0
 hello from process:            2
 hello from process:            3
 hello from process:            5
 hello from process:            6
 hello from process:            4
 hello from process:            7
```

### transformed to

Here, instead of `kmp.h`, I'll be defining `openmpinterop` module that will contain all `bind(c)` interfaces.

> openmpinterop

```fortran
module openmpinterop
use, intrinsic :: iso_c_binding
interface
    subroutine kmpc_fork_call(ident, nargs, microtask) bind(c, name="__kmpc_fork_call")
    import :: c_ptr, c_funptr, c_int
    type(c_ptr), intent(in) :: ident
    integer(c_int), value :: nargs
    type(c_funptr), value :: microtask
    end subroutine kmpc_fork_call

    function omp_get_max_threads() bind(c, name="omp_get_max_threads")
    import :: c_int
    integer(c_int) :: omp_get_max_threads
    end function omp_get_max_threads

    function omp_get_thread_num() bind(c, name="omp_get_thread_num")
    import :: c_int
    integer(c_int) :: omp_get_thread_num
    end function omp_get_thread_num
end interface

type ident_t
    integer(c_int) flags
    integer(c_int) reserved_1
    integer(c_int) reserved_2
    integer(c_int) reserved_3
    character(len=23) psource
end type ident_t
end module openmpinterop
```

> translated code

```fortran
subroutine main_omp() bind(c)
use :: iso_c_binding, only: c_int
use openmpinterop
integer(c_int) :: thread_id
thread_id = omp_get_thread_num()
print *, "hello from thread:", thread_id
end subroutine main_omp

program main
use openmpinterop

implicit none

interface
    subroutine main_omp() bind(c)
    end subroutine main_omp
end interface

type(c_ptr) :: loc_ptr
type(ident_t), target :: loc_
integer(c_int) :: flags, reserved_1, reserved_2, reserved_3
character(len=23) :: psource

print *, "omp_get_max_threads(): ", omp_get_max_threads()

flags = 2
reserved_1 = 0
reserved_2 = 0
reserved_3 = 22
psource = ";unknown;unknown;0;0;;"

loc_%flags = flags
loc_%reserved_1 = reserved_1
loc_%reserved_2 = reserved_2
loc_%reserved_3 = reserved_3
loc_%psource = psource

loc_ptr = c_loc(loc_)

print *, "loc_ptr: ", loc_ptr
print *, "c_funloc(main_omp): ", c_funloc(main_omp)

! Call the __kmpc_fork_call function
call kmpc_fork_call(loc_ptr, 0, c_funloc(main_omp))

end program main
```

```console
% gfortran -c openmpinterop.f90 && gfortran a-transformed.f90 /Users/pranavchiku/repos/llvm-project/openmp/build/runtime/src/libomp.dylib && ./a.out
 omp_get_max_threads():            8
 loc_ptr:            6123746520
 c_funloc(main_omp):            4343151376
 hello from thread:           3
 hello from thread:           7
 hello from thread:           6
 hello from thread:           4
 hello from thread:           2
 hello from thread:           1
 hello from thread:           5
 hello from thread:           0
```

Now, increasing complexity of example:

## Array Assignment

### openmp pragma

```fortran
subroutine initialize_array(n, a, val)
use omp_lib
integer, intent(in) :: n
real, intent(in) :: val
real, dimension(n), intent(out) :: a

integer :: i
!$omp parallel do
do i = 1, n
  a(i) = val
end do
end subroutine

program openmp
implicit none
integer :: n, i
real :: val
real, dimension(1000000) :: a

n = 1000000
val = 3.14
call initialize_array(n, a, val)

do i = 1, 10
  print *, a(i)
end do

end program
```

```console
% gfortran b.f90 -fopenmp && ./a.out
   3.14000010    
   3.14000010    
   3.14000010    
   3.14000010    
   3.14000010    
   3.14000010    
   3.14000010    
   3.14000010    
   3.14000010    
   3.14000010    
```

### translated code

We will need to add more interface into `openmpinterop` and thus it looks like:

> openmpinterop

```fortran
module openmpinterop
use, intrinsic :: iso_c_binding
interface
    subroutine kmpc_fork_call_0(ident, nargs, microtask) bind(c, name="__kmpc_fork_call")
    import :: c_ptr, c_funptr, c_int
    type(c_ptr), intent(in) :: ident
    integer(c_int), value :: nargs
    type(c_funptr), value :: microtask
    end subroutine kmpc_fork_call_0

    subroutine kmpc_fork_call_3(ident, nargs, microtask, n, a, val) bind(c, name="__kmpc_fork_call")
        import :: c_ptr, c_funptr, c_int, c_float
        type(c_ptr), intent(in) :: ident
        integer(c_int), value :: nargs
        type(c_funptr), value :: microtask

        type(c_ptr) :: n, val, a
        ! integer(c_int), value :: n
        ! real(c_float), value :: val
        ! real(c_float), dimension(n), intent(out) :: a
    end subroutine kmpc_fork_call_3

    subroutine kmpc_for_static_init_4(loc, gtid, schedtype, plastiter, plower, pupper, &
        pstride, incr, chunk) bind(c, name="__kmpc_for_static_init_4")
    import :: c_ptr, c_int
    type(c_ptr), intent(in) :: loc
    integer(c_int), value :: gtid
    integer(c_int) :: schedtype, incr, chunk
    type(c_ptr), intent(inout) :: plastiter, plower, pupper, pstride
    end subroutine kmpc_for_static_init_4

    subroutine kmpc_for_static_fini(loc, global_tid) bind(c, name="__kmpc_for_static_fini")
    import :: c_ptr, c_int
    type(c_ptr), intent(in) :: loc
    integer(c_int), value :: global_tid
    end subroutine kmpc_for_static_fini

    subroutine kmpc_barrier(loc, global_tid) bind(c, name="__kmpc_barrier")
    import :: c_ptr, c_int
    type(c_ptr), intent(in) :: loc
    integer(c_int), value :: global_tid
    end subroutine kmpc_barrier

    function omp_get_max_threads() bind(c, name="omp_get_max_threads")
    import :: c_int
    integer(c_int) :: omp_get_max_threads
    end function omp_get_max_threads

    function omp_get_thread_num() bind(c, name="omp_get_thread_num")
    import :: c_int
    integer(c_int) :: omp_get_thread_num
    end function omp_get_thread_num

    function kmp_get_global_thread_id() bind(c, name="__kmp_get_global_thread_id")
    import :: c_int
    integer(c_int) :: kmp_get_global_thread_id
    end function kmp_get_global_thread_id

end interface

type ident_t
    integer(c_int) flags
    integer(c_int) reserved_1
    integer(c_int) reserved_2
    integer(c_int) reserved_3
    character(len=23) psource
end type ident_t

contains

type(c_ptr) function create_loc(flags) result(loc_ptr)
implicit none
type(ident_t), target :: loc_
integer(c_int) :: flags, reserved_1, reserved_2, reserved_3
character(len=23) :: psource

reserved_1 = 0
reserved_2 = 0
reserved_3 = 22
psource = ";unknown;unknown;0;0;;"

loc_%flags = flags
loc_%reserved_1 = reserved_1
loc_%reserved_2 = reserved_2
loc_%reserved_3 = reserved_3
loc_%psource = psource

loc_ptr = c_loc(loc_)
end function create_loc

end module openmpinterop
```


This example is not fully working yet, but it compiles and does something which means we are heading in right direction.

> b-transformed.f90

```fortran
subroutine lcompilers_initialize_array(global_tid, bound_tid, n, a, val) bind(c)
use openmpinterop

type(c_ptr), value :: global_tid, bound_tid
type(c_ptr) :: n, val, a

integer, pointer :: f_global_tid, f_bound_tid, f_tmp

type(c_ptr) :: lastiter, lower, upper, stride
integer(c_int) :: incr, chunk
integer, pointer :: t_lastier, t_lower, t_upper, t_stride

integer :: i

type(c_ptr) :: loc_ptr
loc_ptr = create_loc(514)

incr = 1;
chunk = 1;

allocate(t_lastier)
allocate(t_lower)
allocate(t_upper)
allocate(t_stride)

! t_lastier = 1
! lastiter = c_loc(t_lastier)
! t_lower = 0
! lower = c_loc(t_lower)
! t_upper = n
! upper = c_loc(t_upper)
! t_stride = 1
! stride = c_loc(t_stride)

call c_f_pointer(global_tid, f_global_tid)
call c_f_pointer(bound_tid, f_bound_tid)

! call kmpc_for_static_init_4(loc_ptr, f_global_tid, 34, lastiter, lower, upper, stride, incr, chunk)

call c_f_pointer(lower, f_tmp)
print *, "n: ", n

! call kmpc_for_static_fini(loc_ptr, f_global_tid)
! call kmpc_barrier(loc_ptr, f_global_tid)

end subroutine

subroutine initialize_array(n, a, val)
use openmpinterop
interface
subroutine lcompilers_initialize_array(global_tid, bound_tid, n, a, val) bind(c)
import :: c_ptr, c_int, c_float
type(c_ptr), value :: global_tid, bound_tid
type(c_ptr) :: n, val, a
end subroutine lcompilers_initialize_array
end interface

integer, intent(in), target :: n
real, intent(in), target :: val
real, dimension(n), target :: a

type(c_ptr) :: n_c_ptr, val_c_ptr, a_c_ptr

type(c_ptr) :: loc_ptr
loc_ptr = create_loc(2)

n_c_ptr = c_loc(n)
val_c_ptr = c_loc(val)
a_c_ptr = c_loc(a(1))

call kmpc_fork_call_3( loc_ptr, 3, c_funloc(lcompilers_initialize_array), n_c_ptr, a_c_ptr, val_c_ptr)
end subroutine


program openmp
interface
subroutine initialize_array(n, a, val)
use iso_c_binding
integer, intent(in), target :: n
real, intent(in), target :: val
real, dimension(n), target :: a
end subroutine initialize_array
end interface

integer :: n = 1000000
real :: val = 3.14
real, dimension(1000000) :: a

integer :: i

call initialize_array(n, a, val)

do i = 1, 10
    print *, a(i)
end do

end program
```

```console
% gfortran -c openmpinterop.f90 && gfortran b-transformed.f90 /Users/pranavchiku/repos/llvm-project/openmp/build/runtime/src/libomp.dylib && ./a.out
 n:            6094206224
 n:            6094206224
 n:            6094206224
 n:            6094206224
 n:            6094206224
 n:            6094206224
 n:            6094206224
 n:            6094206224
   0.00000000    
   0.00000000    
   0.00000000    
   0.00000000    
   0.00000000    
   0.00000000    
   0.00000000    
   0.00000000    
   0.00000000    
   0.00000000    

```

The output is not yet aligning, I will continue to work on this and get it done soon. Thanks for reading this :)

---
