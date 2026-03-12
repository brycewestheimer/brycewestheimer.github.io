---
title: "HPC Best Practices in the Age of AI Coding Assistants"
date: 2026-02-02
description: "Hard-won lessons for scientific software development, and how to direct AI coding tools so they help rather than hurt performance."
tags: ["hpc", "scientific-computing", "ai-tools", "performance", "mpi", "cpp", "fortran", "python"]
---

A simulation that compiles and produces plausible output is not the same as a simulation that uses the hardware well. I have watched code that ran for 72 hours on a cluster finish in 4 hours after someone restructured its memory layout. The physics did not change. The algorithm did not change. The data just moved through cache instead of around it.

Scientific software has a particular failure mode: it can be wrong in ways that look right. A 2% error in a turbulence model might not show up in a spot check but will ruin a parameter sweep. A race condition in a distributed reduction might produce correct results 99 times out of 100, then silently corrupt the 100th.

AI coding assistants have entered this world, and they are genuinely useful. They can scaffold build systems, write test harnesses, and refactor boilerplate faster than any human. But they carry assumptions from the domains they were trained on most heavily, and those assumptions often conflict with what HPC code needs. The rest of this post is about the foundations that still matter, and how to steer AI tools when those foundations are at stake.

## Foundations that still matter

### Memory layout as the first design decision

Before you write a single function, decide how your data will sit in memory. This decision propagates through everything else: vectorization, cache behavior, communication buffers, GPU kernel design.

Row-major versus column-major is the obvious choice, but it is only the beginning. Structure-of-Arrays (SoA) versus Array-of-Structures (AoS) determines whether a loop that touches one field of a particle can stream through cache or must skip over three unused fields per element.

```cpp
// AoS: bad for loops that only touch position
struct Particle { double x, y, z, mass, vx, vy, vz, charge; };
std::vector<Particle> particles(N);

// SoA: each field is contiguous, vectorizes cleanly
struct Particles {
    std::vector<double> x, y, z, mass, vx, vy, vz, charge;
};
```

A modern CPU cache line is 64 bytes. Eight doubles. If your inner loop reads one double from each 64-byte struct, you are using 12.5% of every cache line fetch. The hardware prefetcher cannot help you when your access pattern looks random at the byte level.

### Communication topology before code

In distributed computing, data movement dominates. A clever local algorithm that requires an all-to-all exchange will lose to a simpler algorithm that only needs nearest-neighbor communication.

Map your communication pattern before writing the compute kernel. How many bytes cross each link? How many ranks participate? Is the pattern sparse (point-to-point) or dense (collective)? Can you overlap communication with computation?

```cpp
// Expensive: global allreduce every iteration
for (int step = 0; step < nsteps; ++step) {
    compute_local(data);
    MPI_Allreduce(local, global, N, MPI_DOUBLE, MPI_SUM, MPI_COMM_WORLD);
}

// Cheaper: halo exchange with nearest neighbors
for (int step = 0; step < nsteps; ++step) {
    exchange_halos(data, left_rank, right_rank);
    compute_local(data);
}
```

Think about NUMA topology on the node too. Two sockets sharing memory does not mean they share bandwidth equally. A thread writing to memory owned by the other socket pays a 2-3x latency penalty on most Intel and AMD systems.

### Measure before you optimize, but design for measurement

Premature optimization is wasteful, but premature measurement is not. Instrument from the start. Use timers around major phases. Track bytes moved, not just FLOPS.

```cpp
auto t0 = std::chrono::high_resolution_clock::now();
compute_stencil(grid);
auto t1 = std::chrono::high_resolution_clock::now();
double seconds = std::chrono::duration<double>(t1 - t0).count();
double gb_moved = grid_bytes * 2.0 / 1e9; // read + write
double bandwidth = gb_moved / seconds;     // compare to STREAM triad
```

Compare your measured bandwidth against the STREAM benchmark for your hardware. If you are getting 30% of STREAM, your kernel is memory-bound and there is room to improve. If you are at 80%, you are doing well and should look elsewhere for gains.

Profile with the right tool for the question. `perf stat` for top-level cycle and cache-miss counts. `perf record` / `perf report` for hotspot identification. Intel VTune or AMD uProf for microarchitectural detail. NVIDIA Nsight Systems for GPU timeline. Arm MAP for MPI communication overhead.

## What the language choice actually determines

### C++

C++ gives you control over memory layout, allocation, and dispatch. Templates enable zero-cost abstractions when the compiler can see through them, but "zero-cost" requires that the optimizer actually inlines. Check with `-Rpass=inline` (Clang) or `-fopt-info-inline` (GCC).

The traps are well-known. Virtual dispatch in a hot loop defeats devirtualization unless the compiler can prove the concrete type. `std::map` and `std::unordered_map` scatter allocations across the heap; use flat containers or sorted vectors for lookup tables in performance-critical paths. `std::shared_ptr` is thread-safe (atomic reference count), which means it is slower than `std::unique_ptr` even in single-threaded code.

C++20 concepts help constrain templates without SFINAE noise. They make overload sets clearer and error messages shorter. They also help AI tools generate correct generic code because the constraints are explicit in the signature rather than buried in `enable_if` chains.

Link-time optimization (`-flto`) is almost always worth enabling for production builds. It lets the compiler inline across translation units, which matters when your hot path calls through a header-only interface into a separately compiled backend.

SIMD-friendly code means contiguous data, no aliasing, and loop trip counts that the compiler can reason about. Use `__restrict__` when pointers do not alias. Prefer `std::span` or raw pointers with known bounds over iterators in the innermost loop. Check vectorization reports: `-fopt-info-vec-missed` (GCC) or `-Rpass-missed=loop-vectorize` (Clang).

### Fortran

Fortran's advantage is that the language semantics help the optimizer. Arrays do not alias by default (no pointer arithmetic). Column-major layout is the standard, which means the first index should vary fastest in nested loops.

```fortran
! Good: column-major access (i varies fastest)
do j = 1, ny
    do i = 1, nx
        a(i, j) = b(i, j) + c(i, j)
    end do
end do
```

Assumed-shape arrays (`real, intent(in) :: x(:,:)`) let the compiler see the rank and generate efficient code without explicit bounds arguments. Explicit-shape arrays (`real :: x(n,m)`) require passing `n` and `m`, and the compiler may not optimize as aggressively.

Coarray Fortran provides native PGAS syntax for distributed computing. `x(:)[image]` accesses data on another image (rank) with syntax that the compiler can optimize for the interconnect. It is worth considering for stencil codes where the communication pattern is static.

### Python

Python is an orchestration language for scientific computing, not a compute kernel language. If your inner loop is written in Python and operates on scalars, you have already lost.

NumPy, SciPy, and domain-specific libraries (PETSc via petsc4py, FFTW via pyFFTW) provide the bridge to compiled code. The pattern is: set up the problem in Python, call a compiled kernel for the heavy work, collect results in Python.

```python
import numpy as np

# Good: vectorized NumPy operation (calls compiled BLAS/LAPACK)
result = np.dot(A, x) + b

# Bad: scalar loop in Python (1000x slower)
result = np.zeros(n)
for i in range(n):
    for j in range(n):
        result[i] += A[i, j] * x[j]
    result[i] += b[i]
```

mpi4py provides MPI bindings that work with NumPy arrays directly. The buffer interface avoids serialization for contiguous arrays of simple types. Use uppercase methods (`Allreduce`, `Bcast`) for buffer-based communication and lowercase methods (`allreduce`, `bcast`) for pickled Python objects.

## Directing AI coding assistants for scientific software

This is where the current generation of AI tools needs the most guidance. They are trained heavily on web applications, command-line tools, and general-purpose libraries. When you ask them to write scientific code, they reach for patterns from those domains unless you tell them not to.

### What AI tools get right by default

- **Build system configuration.** CMake boilerplate, dependency detection, compiler flag selection. These are well-represented in training data and follow predictable patterns. Let the tool write your `CMakeLists.txt` and your CI pipeline.

- **Test scaffolding.** GoogleTest fixtures, pytest parametrization, CTest integration. The structure of a test is the same whether you are testing a web endpoint or a linear algebra kernel. The tool can generate the harness while you focus on the assertions.

- **Boilerplate refactoring.** Renaming variables, extracting functions, converting between equivalent representations. These are mechanical transformations that the tool handles reliably.

- **Documentation and comments.** Explaining what existing code does is something AI tools do well. Generating docstrings, writing usage examples, and annotating tricky sections. This frees you to write the code that needs careful thought rather than the prose around it.

### The web-dev gravity well

Left to its defaults, an AI coding assistant will write code that looks like a web application backend. This is not a criticism of the tool — it is a reflection of the training distribution. Web-dev patterns are overwhelmingly the most common in public code.

Common patterns that work in web development but hurt HPC performance:

- **Heap allocation per operation.** A web request handler that allocates a response object, fills it, and returns it is fine when the handler runs once per network round trip. A stencil update that allocates a temporary vector per grid cell per timestep will spend more time in `malloc` than in arithmetic.

- **Virtual dispatch everywhere.** Polymorphism through base class pointers is the default OOP pattern. It is appropriate for plugin systems and event handlers. It is inappropriate for inner loops that execute billions of times, where the vtable lookup and indirect branch prediction miss add up.

- **`std::map` where flat arrays suffice.** If your keys are dense integers from 0 to N, use a vector. If your keys are sparse but you look them up in a hot loop, use a sorted vector with binary search or a flat hash map.

- **`std::shared_ptr` where ownership is clear.** In scientific code, data ownership is almost always unambiguous. The simulation owns the grid. The solver owns its workspace. Shared ownership is rare, and the atomic reference counting in `shared_ptr` is wasted cost.

### Non-performant patterns AI produces for HPC

**Allocating in inner loops.** This is the most common performance bug AI tools introduce. The tool sees that you need a temporary buffer, so it declares one inside the loop. Move allocations outside the loop and reuse buffers.

```cpp
// AI-generated: allocates every iteration
for (int step = 0; step < 1000000; ++step) {
    std::vector<double> temp(N);  // malloc + memset per step
    compute(data, temp);
}

// Fixed: allocate once, reuse
std::vector<double> temp(N);
for (int step = 0; step < 1000000; ++step) {
    compute(data, temp);
}
```

**`vector<vector<>>` for 2D data.** A vector of vectors is not contiguous in memory. Each inner vector is a separate heap allocation. For a 2D grid, use a single flat allocation with index arithmetic.

```cpp
// AI-generated: N+1 allocations, non-contiguous
std::vector<std::vector<double>> grid(ny, std::vector<double>(nx));

// Fixed: single allocation, contiguous
std::vector<double> grid(ny * nx);
auto at = [nx](int i, int j) { return i * nx + j; };
```

**Ignoring cache locality in loop ordering.** When iterating over multidimensional data, the order of loop indices determines whether you stream through memory or jump around. AI tools often generate loops in the "natural" order (outermost index first) without considering the storage layout.

**Serializing MPI buffers unnecessarily.** For contiguous arrays of primitive types, MPI can send the buffer directly. AI tools sometimes insert a serialization step (copying into a byte buffer, then sending the byte buffer) that adds latency and memory overhead for no reason.

**Explicit-shape Fortran arrays in interfaces.** AI tools tend to generate Fortran subroutines with explicit-shape dummy arguments (`real :: x(n,m)`) instead of assumed-shape (`real, intent(in) :: x(:,:)`). The assumed-shape version is safer (bounds checking) and often optimizes better.

**Python element-wise loops.** When asked to implement a numerical operation in Python, AI tools frequently generate scalar loops instead of vectorized NumPy operations. The performance difference is typically 100x to 1000x.

### How to steer the tool

**Provide architectural constraints upfront.** Before asking the tool to write a function, tell it the performance requirements. "This function is called 10 million times per timestep. It must not allocate. All data is contiguous in memory. Use restrict pointers." The tool will respect these constraints if you state them explicitly.

**Give reference implementations.** Paste a small example of the coding style you want. If you want SoA layout, show a 5-line SoA struct. If you want flat 2D arrays with index macros, show the macro. The tool will follow the pattern.

**Ask for performance justification.** After the tool generates code, ask "what is the memory access pattern of this loop?" or "how many cache lines does one iteration of this loop touch?" This forces the tool to reason about the hardware implications of its choices, and it will often catch its own mistakes.

**Use AI for the boring parts, write the hot path yourself.** Let the tool generate the I/O layer, the argument parser, the test harness, the build system, and the visualization scripts. Write the stencil kernel, the particle force loop, and the communication overlap logic yourself. The hot path is where domain expertise matters most, and it is usually a small fraction of the total code.

**Review generated code with a profiler, not just a compiler.** Code that compiles and passes tests might still be 10x slower than it needs to be. Profile the AI-generated code before accepting it. `perf stat` takes 30 seconds to run and will tell you if the tool introduced a performance cliff.

### MPI-specific pitfalls

AI tools have a particular set of failure modes with MPI because the programming model is unusual. Most code they have seen is single-process.

**`MPI_COMM_WORLD` everywhere.** The tool will use the world communicator for every operation because that is what simple examples do. In production code, you should create subcommunicators for different phases of the computation. A solver that only involves a subset of ranks should not force all ranks to participate in its collectives.

**Forgetting collective participation.** Every rank in a communicator must call a collective operation. AI tools sometimes put `MPI_Allreduce` inside an `if (rank == 0)` block, which deadlocks because the other ranks never enter the call.

```cpp
// BUG: only rank 0 calls allreduce, others hang
if (rank == 0) {
    MPI_Allreduce(&local, &global, 1, MPI_DOUBLE, MPI_SUM, comm);
}

// Correct: all ranks participate
MPI_Allreduce(&local, &global, 1, MPI_DOUBLE, MPI_SUM, comm);
if (rank == 0) {
    printf("Global sum: %f\n", global);
}
```

**Deadlock-prone send/recv ordering.** When all ranks send to rank 0, then receive from rank 0, the tool may generate blocking sends that fill the MPI buffer and deadlock. Use non-blocking operations (`MPI_Isend`/`MPI_Irecv`) or restructure as a collective (`MPI_Gather`/`MPI_Scatter`).

**Missing thread-level initialization.** If your code uses MPI from multiple threads, you need `MPI_Init_thread` with `MPI_THREAD_MULTIPLE` or at least `MPI_THREAD_FUNNELED`. AI tools almost always generate plain `MPI_Init`, which defaults to `MPI_THREAD_SINGLE`.

**Communicator leaks.** `MPI_Comm_dup` and `MPI_Comm_split` create new communicators that consume resources in the MPI runtime. AI tools rarely generate the matching `MPI_Comm_free`. In a long-running simulation that creates communicators per phase, this is a resource leak.

### Cache and SIMD awareness gaps

AI tools do not reason about cache hierarchies or SIMD register widths unless you prompt them to.

**Loop tiling.** A naive matrix multiply accesses column elements of B with stride N, thrashing the cache when N is large. Tiling the loops into blocks that fit in L1 keeps a small tile of B resident while you multiply it against multiple rows of A. AI tools will not tile loops unless you ask.

```cpp
// Untiled: B accessed column-wise, one cache miss per element
for (int i = 0; i < N; ++i)
    for (int j = 0; j < N; ++j)
        for (int k = 0; k < N; ++k)
            C[i*N+j] += A[i*N+k] * B[k*N+j];

// Tiled: blocks of B stay in L1 across iterations
constexpr int BLK = 64;
for (int ii = 0; ii < N; ii += BLK)
    for (int jj = 0; jj < N; jj += BLK)
        for (int kk = 0; kk < N; kk += BLK)
            for (int i = ii; i < std::min(ii+BLK, N); ++i)
                for (int j = jj; j < std::min(jj+BLK, N); ++j)
                    for (int k = kk; k < std::min(kk+BLK, N); ++k)
                        C[i*N+j] += A[i*N+k] * B[k*N+j];
```

**Alignment.** SIMD loads and stores are fastest when the data address is aligned to the vector width (32 bytes for AVX2, 64 bytes for AVX-512). `alignas(64)` on stack arrays and `std::aligned_alloc` for heap arrays enable aligned instructions. AI tools rarely add alignment annotations.

**Reduction ordering.** Floating-point addition is not associative. A parallel reduction that sums in a different order than a serial reduction will produce a different result. AI tools will not warn you about this. For reproducible results across different rank counts, use a fixed-order tree reduction.

**Prefetch hints.** For irregular access patterns (sparse matrix-vector multiply, indirect addressing), software prefetch hints (`__builtin_prefetch` on GCC/Clang) can hide memory latency. This is a niche optimization that AI tools will never suggest, but it can matter in memory-bound sparse kernels.

## Testing when "correct" has tolerances

Scientific software testing differs from application testing because exact equality is often meaningless. Floating-point arithmetic is approximate, and different parallelization strategies produce different rounding.

**Floating-point comparison strategies.** Never use `==` for doubles. Use a relative tolerance with an absolute floor:

```cpp
bool approx_equal(double a, double b, double rel_tol = 1e-12,
                  double abs_tol = 1e-15) {
    double diff = std::abs(a - b);
    double scale = std::max(std::abs(a), std::abs(b));
    return diff <= std::max(rel_tol * scale, abs_tol);
}
```

Choose tolerances based on the algorithm. Direct solvers accumulate O(N) rounding errors. Iterative solvers converge to a tolerance you specify. Monte Carlo methods have statistical error that dominates floating-point error.

**Regression against known-good results.** Generate a reference solution at high precision (or on a single rank where the computation is deterministic). Store it as a binary file. Compare new runs against the reference within your chosen tolerance. This catches both correctness regressions and precision regressions from algorithm changes.

**Multi-rank testing.** Run the same problem on 1, 2, 4, and 8 ranks. The answers should agree within floating-point tolerance. If they do not, you have a partitioning bug or a communication error, not a rounding issue. Use this as a first check before investigating numerical differences.

**Property-based tests.** Conservation laws are free test oracles. If your simulation conserves energy, check that total energy at step N equals total energy at step 0 within tolerance. If your linear solver should produce `Ax = b`, check the residual `||Ax - b||` rather than comparing `x` element-wise against a reference.

```cpp
// Property: total mass is conserved
double initial_mass = global_sum(density, comm);
run_simulation(density, nsteps, comm);
double final_mass = global_sum(density, comm);
EXPECT_NEAR(initial_mass, final_mass, 1e-10 * initial_mass);
```

## The tool does not replace the thinking

AI coding assistants lower the cost of writing code. They do not lower the cost of understanding the problem. The physicist still needs to know which equations to solve. The numerical analyst still needs to choose the discretization. The systems programmer still needs to understand why a 64-byte cache line matters.

Use these tools for what they are good at: the mechanical parts of programming that take time but not insight. Direct them away from their defaults when those defaults conflict with your domain. And measure the result, because the compiler and the hardware are the final arbiters of whether code is fast, not the tool that generated it.
