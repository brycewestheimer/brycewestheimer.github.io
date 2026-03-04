---
title: "DTL Tutorial: distributed_vector in C++, C, Fortran, and Python"
date: 2026-03-06
description: "A hands-on guide to creating and reducing a distributed_vector using DTL's four language bindings."
level: "intermediate"
tags: ["dtl", "distributed-computing", "cpp", "c", "fortran", "python", "mpi", "tutorial"]
---

DTL (Distributed Template Library) is a C++20 header-first library for distributed computing. It provides STL-inspired containers and algorithms that partition data across MPI ranks, with explicit policies for distribution, placement, execution, and consistency.

The library ships with bindings for C (stable ABI), Fortran 2008 (via the C ABI), and Python (via pybind11). All four language interfaces expose the same core workflow: create an environment, obtain a context, build distributed containers, and operate on them with local or collective operations.

DTL is at version 0.1.0-alpha.1 and licensed BSD-3-Clause. The source is available on GitHub.

## Building and installing DTL

### Prerequisites

- CMake 3.20 or later
- A C++20 compiler (GCC 11+, Clang 15+)
- An MPI implementation (OpenMPI, MPICH, or Intel MPI)
- For Fortran bindings: gfortran 10+ or Intel Fortran
- For Python bindings: Python 3.9+, pybind11, NumPy

### Build from source with all bindings

```bash
git clone https://github.com/brycewestheimer/dtl.git
cd dtl
cmake -S . -B build \
    -DCMAKE_BUILD_TYPE=Release \
    -DDTL_ENABLE_MPI=ON \
    -DDTL_BUILD_C_BINDINGS=ON \
    -DDTL_BUILD_FORTRAN=ON \
    -DDTL_BUILD_PYTHON=ON
cmake --build build -j6
cmake --install build --prefix /usr/local
```

### What gets installed

| Component | Description |
|---|---|
| `include/dtl/` | C++20 headers (header-only core) |
| `libdtl_runtime.so` | Process-global backend state: singleton lifecycle, backend discovery, plugin loading, connection pooling |
| `libdtl_c.so` | Stable C ABI wrapping all containers, algorithms, and collectives |
| `libdtl_fortran.so` | Fortran 2008 module wrappers and typed helpers |
| `*.mod` files | Fortran module interface files |
| Python package | pybind11 bindings importable as `import dtl` |

### Consumer CMake integration (C++)

```cmake
find_package(DTL REQUIRED)
target_link_libraries(my_app PRIVATE DTL::dtl)
```

The `DTL::dtl` target transitively links `libdtl_runtime`, so you do not need to add it explicitly.

### Python installation

```bash
cd dtl
pip install .
```

## Core concepts: Environment, Context, and Containers

Three types form the foundation of every DTL program.

**Environment** manages the backend lifecycle. Creating the first `environment` object initializes MPI (and any other enabled backends). Destroying the last `environment` object finalizes them. The implementation uses reference counting internally, so multiple environments can coexist safely in library code.

**Context** binds together the backend domains that a set of containers will use. It provides `rank()`, `size()`, and `barrier()`. You obtain a context from the environment via `make_world_context()`, which wraps `MPI_COMM_WORLD` in a duplicated communicator. For GPU backends, pass a `device_id` to bind the context to a specific device.

**Container** holds distributed data. `distributed_vector` is block-partitioned by default: each rank owns a contiguous chunk of the global index space. Calling `local_view()` gives communication-free access to the local partition as a span-like object (C++) or a NumPy array (Python). Calling `global_offset()` returns the global index of the first local element, so you can map between local and global indices.

The pattern is identical across all four languages: **environment, then context, then container**.

## The workflow

Every example in this tutorial follows the same six steps:

1. Create an environment (initializes MPI)
2. Create a world context from the environment
3. Create a `distributed_vector` with `global_size = 10000`
4. Fill the local partition so that each element equals its global index
5. Compute the local sum, then allreduce to get the global sum
6. Verify: the expected result is `10000 * 9999 / 2 = 49995000`

## C++

```cpp
// vector_sum.cpp
#include <dtl/dtl.hpp>
#include <iostream>
#include <numeric>

int main(int argc, char** argv) {
    // 1. Create environment (initializes MPI)
    dtl::environment env(argc, argv);

    // 2. Create world context
    auto ctx = env.make_world_context();
    auto& comm = ctx.get<dtl::mpi_domain>().communicator();

    // 3. Create distributed vector (block-partitioned across ranks)
    const dtl::size_type global_size = 10000;
    dtl::distributed_vector<long> vec(global_size, ctx);

    // 4. Fill local partition: each element = its global index
    auto local = vec.local_view();
    dtl::index_t offset = vec.global_offset();
    for (dtl::size_type i = 0; i < local.size(); ++i) {
        local[i] = static_cast<long>(offset + static_cast<dtl::index_t>(i));
    }

    // 5. Local sum, then allreduce for global sum
    long local_sum = std::accumulate(local.begin(), local.end(), 0L);
    long global_sum = comm.allreduce_sum_value<long>(local_sum);

    // 6. Verify
    long expected = static_cast<long>(global_size)
                  * static_cast<long>(global_size - 1) / 2;

    if (ctx.rank() == 0) {
        std::cout << "Global sum: " << global_sum << "\n";
        std::cout << "Expected:   " << expected << "\n";
        std::cout << (global_sum == expected ? "PASS" : "FAIL") << "\n";
    }

    return (global_sum == expected) ? 0 : 1;
}
```

### Build and run

With CMake:

```cmake
find_package(DTL REQUIRED)
add_executable(vector_sum vector_sum.cpp)
target_link_libraries(vector_sum PRIVATE DTL::dtl)
```

Or directly:

```bash
mpicxx -std=c++20 -o vector_sum vector_sum.cpp \
    -I/usr/local/include -L/usr/local/lib -ldtl_runtime
mpirun -np 4 ./vector_sum
```

### Key points

- `dtl::environment` is RAII. Construction calls `MPI_Init_thread`; destruction calls `MPI_Finalize`.
- `make_world_context()` duplicates `MPI_COMM_WORLD`, so DTL's communicator is isolated from your other MPI calls.
- `local_view()` returns a lightweight span-like object. It does not copy data and does not communicate.
- `global_offset()` returns the global index of the first element on this rank. For 4 ranks and 10000 elements, rank 0 gets offset 0, rank 1 gets 2500, rank 2 gets 5000, rank 3 gets 7500.
- `std::accumulate` works on the local view because it exposes `begin()` and `end()` iterators.
- `allreduce_sum_value<long>()` is a typed convenience wrapper on the communicator. It calls `MPI_Allreduce` with `MPI_SUM` under the hood.

## C

```c
/* vector_sum.c */
#include <dtl/bindings/c/dtl.h>
#include <stdio.h>
#include <stdint.h>

int main(int argc, char** argv) {
    dtl_environment_t env = NULL;
    dtl_context_t ctx = NULL;
    dtl_vector_t vec = NULL;

    /* 1. Create environment (initializes MPI) */
    dtl_environment_create_with_args(&env, &argc, &argv);

    /* 2. Create world context */
    dtl_environment_make_world_context(env, &ctx);

    /* 3. Create distributed vector */
    const dtl_size_t global_size = 10000;
    dtl_vector_create(ctx, DTL_DTYPE_INT64, global_size, &vec);

    /* 4. Fill local partition: each element = its global index */
    int64_t* data = (int64_t*)dtl_vector_local_data_mut(vec);
    dtl_size_t local_sz = dtl_vector_local_size(vec);
    dtl_index_t offset = dtl_vector_local_offset(vec);

    for (dtl_size_t i = 0; i < local_sz; ++i) {
        data[i] = (int64_t)(offset + (dtl_index_t)i);
    }

    /* 5. Local sum, then allreduce for global sum */
    int64_t local_sum = 0;
    for (dtl_size_t i = 0; i < local_sz; ++i) {
        local_sum += data[i];
    }

    int64_t global_sum = 0;
    dtl_allreduce(ctx, &local_sum, &global_sum,
                  1, DTL_DTYPE_INT64, DTL_OP_SUM);

    /* 6. Verify */
    int64_t expected = (int64_t)global_size
                     * ((int64_t)global_size - 1) / 2;

    if (dtl_context_rank(ctx) == 0) {
        printf("Global sum: %ld\n", (long)global_sum);
        printf("Expected:   %ld\n", (long)expected);
        printf("%s\n", (global_sum == expected) ? "PASS" : "FAIL");
    }

    /* Cleanup in reverse order */
    dtl_vector_destroy(vec);
    dtl_context_destroy(ctx);
    dtl_environment_destroy(env);

    return (global_sum == expected) ? 0 : 1;
}
```

### Build and run

```bash
mpicc -o vector_sum vector_sum.c \
    -I/usr/local/include -L/usr/local/lib \
    -ldtl_c -ldtl_runtime
mpirun -np 4 ./vector_sum
```

### Key points

- All DTL objects are opaque handles (`dtl_environment_t`, `dtl_context_t`, `dtl_vector_t`). You never dereference them directly.
- Functions return `dtl_status` (an integer). `DTL_SUCCESS` is 0. In production code, check every return value.
- `dtl_vector_local_data_mut()` returns `void*`. Cast it to the appropriate pointer type based on the `dtl_dtype` you used at creation.
- `dtl_allreduce()` takes void pointers to send and receive buffers, a count, a data type enum, and a reduction operation enum. This mirrors the MPI calling convention.
- Cleanup must happen in reverse order: destroy the vector before the context, and the context before the environment. The environment's destructor finalizes MPI.

## Fortran

```fortran
! vector_sum.f90
program vector_sum
    use dtl
    implicit none

    type(c_ptr) :: env, ctx, vec
    integer(c_int) :: status, rank
    integer(c_int64_t) :: global_size, local_size, offset, i
    type(c_ptr) :: data_ptr
    real(c_double), pointer :: data(:)
    real(c_double), target :: send_val(1), recv_val(1)
    real(c_double) :: local_sum, global_sum, expected

    ! 1. Create environment (initializes MPI)
    status = dtl_environment_create(env)

    ! 2. Create world context
    status = dtl_environment_make_world_context(env, ctx)
    rank = dtl_context_rank(ctx)

    ! 3. Create distributed vector
    global_size = 10000_c_int64_t
    status = dtl_vector_create(ctx, DTL_DTYPE_FLOAT64, global_size, vec)

    ! 4. Fill local partition: each element = its global index
    local_size = dtl_vector_local_size(vec)
    offset = dtl_vector_local_offset(vec)
    data_ptr = dtl_vector_local_data_mut(vec)
    call c_f_pointer(data_ptr, data, [local_size])

    do i = 1, local_size
        data(i) = real(offset + (i - 1), c_double)
    end do

    ! 5. Local sum, then allreduce for global sum
    local_sum = 0.0_c_double
    do i = 1, local_size
        local_sum = local_sum + data(i)
    end do

    send_val(1) = local_sum
    status = dtl_allreduce_sum_double(ctx, send_val, recv_val, 1_c_int64_t)
    global_sum = recv_val(1)

    ! 6. Verify
    expected = real(global_size, c_double) &
             * real(global_size - 1, c_double) / 2.0_c_double

    if (rank == 0) then
        print '(A,F15.1)', 'Global sum: ', global_sum
        print '(A,F15.1)', 'Expected:   ', expected
        if (global_sum == expected) then
            print *, 'PASS'
        else
            print *, 'FAIL'
        end if
    end if

    ! Cleanup in reverse order
    nullify(data)
    call dtl_vector_destroy(vec)
    call dtl_context_destroy(ctx)
    call dtl_environment_destroy(env)

end program vector_sum
```

### Build and run

```bash
mpif90 -o vector_sum vector_sum.f90 \
    -I/usr/local/include -L/usr/local/lib \
    -ldtl_fortran -ldtl_c -ldtl_runtime
mpirun -np 4 ./vector_sum
```

### Key points

- `use dtl` imports all submodules in one statement. It also re-exports `c_ptr`, `c_int`, `c_int64_t`, `c_double`, `c_f_pointer`, and other ISO_C_BINDING types, so no separate `use iso_c_binding` is needed.
- All handles are `type(c_ptr)`. This is Fortran's interoperability mechanism for opaque C pointers.
- `DTL_DTYPE_FLOAT64` is the natural choice for Fortran's `real(c_double)`.
- `dtl_vector_local_data_mut()` returns a `c_ptr`. Convert it to a Fortran array pointer with `c_f_pointer(data_ptr, data, [local_size])`.
- The local offset is 0-based (matching the C/C++ convention), but Fortran array indexing is 1-based. The fill loop uses `offset + (i - 1)` to map from the 1-based loop variable to the 0-based global index.
- `dtl_allreduce_sum_double()` from `dtl_helpers` is a typed wrapper that avoids manual `c_loc()` calls and `DTL_DTYPE`/`DTL_OP` enum arguments. It takes Fortran arrays with the `target` attribute.
- `nullify(data)` before `dtl_vector_destroy` avoids leaving a dangling Fortran pointer to freed memory.
- The Fortran bindings link three libraries: `libdtl_fortran` (module wrappers), `libdtl_c` (C ABI implementation), and `libdtl_runtime` (backend state).

## Python

```python
# vector_sum.py
import numpy as np
import dtl

# 1. Create environment (initializes MPI)
with dtl.Environment() as env:

    # 2. Create world context
    ctx = env.make_world_context()

    # 3. Create distributed vector
    global_size = 10000
    vec = dtl.DistributedVector(ctx, size=global_size, dtype=np.float64)

    # 4. Fill local partition: each element = its global index
    local = vec.local_view()
    offset = vec.local_offset
    local[:] = np.arange(offset, offset + len(local), dtype=np.float64)

    # 5. Local sum, then allreduce for global sum
    local_sum = np.sum(local)
    global_sum = dtl.allreduce(ctx, local_sum, op=dtl.SUM)

    # 6. Verify
    expected = global_size * (global_size - 1) / 2

    if ctx.rank == 0:
        print(f"Global sum: {global_sum:.1f}")
        print(f"Expected:   {expected:.1f}")
        print("PASS" if global_sum == expected else "FAIL")
```

### Run

```bash
mpirun -np 4 python vector_sum.py
```

### Key points

- `dtl.Environment()` supports the `with` statement. Entering the block initializes MPI; exiting it finalizes MPI. No explicit cleanup is needed.
- `env.make_world_context()` returns a `Context` with `.rank`, `.size`, and `.is_root` properties.
- `dtl.DistributedVector()` is a factory function, not a class constructor. It returns a typed container object whose internal type depends on the `dtype` argument.
- `vec.local_view()` returns a NumPy array that shares memory with the underlying C++ buffer (zero-copy). Modifications to the array are reflected in the container and vice versa.
- `vec.local_offset` is a property (no parentheses), not a method call.
- `dtl.allreduce()` accepts scalars or NumPy arrays. For a scalar input, it returns a scalar. The `op` parameter accepts `dtl.SUM`, `dtl.PROD`, `dtl.MIN`, or `dtl.MAX`.
- NumPy's `np.arange` fills the local partition in a single vectorized call. There is no element-wise Python loop.

## Side-by-side comparison

| Step | C++ | C | Fortran | Python |
|---|---|---|---|---|
| Create environment | `dtl::environment env(argc, argv)` | `dtl_environment_create_with_args(&env, &argc, &argv)` | `dtl_environment_create(env)` | `with dtl.Environment() as env:` |
| Create context | `env.make_world_context()` | `dtl_environment_make_world_context(env, &ctx)` | `dtl_environment_make_world_context(env, ctx)` | `env.make_world_context()` |
| Create vector | `distributed_vector<long> vec(N, ctx)` | `dtl_vector_create(ctx, DTL_DTYPE_INT64, N, &vec)` | `dtl_vector_create(ctx, DTL_DTYPE_FLOAT64, N, vec)` | `DistributedVector(ctx, size=N, dtype=np.float64)` |
| Access local data | `vec.local_view()` | `dtl_vector_local_data_mut(vec)` | `c_f_pointer(dtl_vector_local_data_mut(vec), data, [n])` | `vec.local_view()` |
| Get global offset | `vec.global_offset()` | `dtl_vector_local_offset(vec)` | `dtl_vector_local_offset(vec)` | `vec.local_offset` |
| Local sum | `std::accumulate(...)` | Manual loop | Manual loop | `np.sum(local)` |
| Allreduce | `comm.allreduce_sum_value<long>(v)` | `dtl_allreduce(ctx, &s, &r, 1, dtype, op)` | `dtl_allreduce_sum_double(ctx, s, r, 1)` | `dtl.allreduce(ctx, v, op=dtl.SUM)` |
| Cleanup | Automatic (RAII) | `destroy()` in reverse order | `destroy()` in reverse order | Automatic (`with` block) |

## Next steps

This tutorial covered the simplest DTL workflow: a single container with a block partition on CPU ranks. From here, several directions open up.

**Other containers.** `distributed_array` is a fixed-size variant (no `resize()`). `distributed_tensor` supports N-dimensional data with shape and stride queries. `distributed_map` provides a distributed key-value store with hash-based partitioning.

**Policies.** DTL containers accept policy template parameters (C++) or option structs (C/Fortran/Python) that control partition strategy (block, cyclic, block-cyclic, hash, replicated), memory placement (host, device, unified, pinned), and execution mode (sequential, parallel, async).

**GPU contexts.** Pass a `device_id` to `make_world_context()` to bind the context to a CUDA or HIP device. Containers created with a GPU context allocate device memory and support device-side kernels.

**Algorithms.** `dtl::reduce`, `dtl::transform`, `dtl::for_each`, `dtl::sort`, and other STL-parallel algorithms operate on distributed containers with automatic collective communication.

**Views.** `global_view()` returns `remote_ref<T>` proxies for transparent remote access. `segmented_view()` exposes the per-rank structure for algorithms that need partition awareness.

For the full API reference and additional examples, see the [DTL documentation](https://dtl.readthedocs.io) and the [GitHub repository](https://github.com/brycewestheimer/dtl).
