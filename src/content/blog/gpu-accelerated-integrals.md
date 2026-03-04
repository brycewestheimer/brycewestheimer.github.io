---
title: "GPU-Accelerated Molecular Integrals: Turning Irregular Work into Fast Computation"
date: 2026-03-05
description: "How shell-set batching, adaptive dispatch, and consumer-driven accumulation make GPU acceleration practical for two-electron integral evaluation in quantum chemistry."
tags: ["quantum-chemistry", "gpu", "hpc", "molecular-integrals", "libaccint"]
---

Every electronic structure calculation rests on molecular integrals. Overlap, kinetic energy, nuclear attraction, and electron repulsion integrals are the numerical foundation of Hartree-Fock, DFT, and post-HF methods. Among these, two-electron repulsion integrals (ERIs) dominate the cost. For a basis set with N functions, the number of unique shell quartets scales as O(N^4). Screening techniques (Schwarz, density-weighted) prune a large fraction of these, but for molecules of practical interest the surviving workload still numbers in the millions to billions of individual integral evaluations.

GPUs offer massive arithmetic throughput. A single data-center GPU delivers 10-30 TFLOPS of double-precision performance, compared to 0.5-1 TFLOPS for a multi-core CPU. The challenge is not raw speed; it is the structure of the work. Shell quartets differ enormously in size and computational cost, and naive parallelization strategies that ignore this heterogeneity leave most GPU resources idle.

This post examines the techniques that make GPU-accelerated integral evaluation practical: grouping work by angular momentum class, dispatching adaptively between CPU and GPU, streaming results through consumer objects, and integrating screening. The focus is on design patterns rather than kernel micro-optimization, because in most programs the performance bottleneck is work organization, not arithmetic throughput.

## The Structure of Two-Electron Integrals

A two-electron integral batch is defined by four shells with angular momenta (L_a, L_b, L_c, L_d). Each shell of angular momentum L contains (L+1)(L+2)/2 Cartesian basis functions. The number of integrals per shell quartet is the product of functions across all four shells.

An (ss|ss) quartet (all L=0) produces exactly 1 integral. An (ff|ff) quartet (all L=3) produces 10^4 = 10,000 integrals. An (gg|gg) quartet (all L=4) produces 15^4 = 50,625 integrals. The computational cost per quartet varies even more steeply, because higher angular momentum requires more intermediate recurrence steps and more primitive Gaussian contractions.

This irregularity is what makes GPUs hard to use here. GPU kernels achieve peak throughput when every thread in a warp executes the same instruction on data of the same shape. A kernel that mixes (ss|ss) and (ff|ff) quartets wastes threads on padding, branches on angular-momentum-dependent code paths, and underutilizes registers allocated for the worst case.

Contrast this with dense linear algebra. A matrix multiplication operates on uniformly shaped tiles. Each thread block handles the same amount of work. Load balancing is trivial. ERI evaluation does not have this property by default, and the first step toward GPU efficiency is to create it.

## ShellSet Batching: Grouping by Angular Momentum Class

All shell quartets sharing the same angular momentum signature (L_a, L_b, L_c, L_d) are structurally identical. They require the same recurrence relations, produce the same number of integrals, and use the same register layout. If these quartets are gathered into a single batch, a GPU kernel can process them without branching.

The grouping works as follows. Shells in the basis set are sorted by angular momentum into ShellSets. Each ShellSet contains all shells of a given L value. A ShellSetQuartet is then the Cartesian product of four ShellSets:

```
for each unique (La, Lb, Lc, Ld):
    set_a = shells where L == La
    set_b = shells where L == Lb
    set_c = shells where L == Lc
    set_d = shells where L == Ld
    batch = ShellSetQuartet(set_a, set_b, set_c, set_d)
```

Within a ShellSetQuartet, every individual shell quartet has the same angular momentum class. A kernel compiled (or template-instantiated) for that class processes the entire batch with zero divergence. The batch size equals |set_a| x |set_b| x |set_c| x |set_d|, which for common basis sets ranges from tens to millions of quartets per class.

libaccint implements this pattern through its `ShellSetQuartet` type. The `BasisSet` class pre-computes the grouping, and the `Engine` iterates over the resulting batches, dispatching each to the appropriate kernel.

## Dispatch Strategies

Not all ShellSetQuartet batches benefit from GPU execution. Small batches (a handful of quartets) incur kernel launch overhead that exceeds the computation time. Low angular momentum quartets ((ss|ss), (sp|ss)) require so little arithmetic that CPU SIMD can finish them before a GPU kernel even begins executing.

An effective dispatch policy considers several factors:

- **Batch size.** Below a threshold (typically 16-64 quartets), CPU is faster.
- **Angular momentum.** High-AM quartets have more work per quartet, improving GPU occupancy.
- **Primitive count.** More Gaussian primitives per shell means more arithmetic per quartet.
- **Available hardware.** No GPU at all, or a GPU already saturated by another workload.

A cost-model approach assigns an estimated FLOP count to each batch and compares CPU and GPU throughput. A simpler heuristic uses threshold tables indexed by AM class. A hybrid approach profiles the first call for each AM class and caches the result for subsequent calls.

libaccint exposes this through the `DispatchConfig` structure and the `BackendHint` enum:

```cpp
enum class BackendHint {
    Auto,       // let the dispatch policy decide
    ForceCPU,   // always use CPU
    ForceGPU,   // always use GPU (error if unavailable)
    PreferCPU,  // prefer CPU, but GPU if beneficial
    PreferGPU,  // prefer GPU, fall back to CPU
};
```

The `DispatchConfig` controls thresholds: `min_gpu_batch_size`, `min_gpu_primitives`, `high_am_threshold`, and an auto-tuning mode that profiles the first call and reuses the decision. Users pass a `BackendHint` to any compute call to override the automatic decision. The `Auto` default is correct for most workloads.

## The Consumer Pattern

Storing the full ERI tensor is impractical. For N basis functions, the tensor has N^4 elements. A modest basis of 500 functions requires 500^4 * 8 bytes = 500 GB. Even with screening, the surviving integrals number in the billions. No machine has enough memory to store them all, and no algorithm needs them all simultaneously.

Instead, integrals are computed in batches and each batch is immediately passed to a consumer that performs an O(N^2) reduction. For Hartree-Fock, the consumer is a Fock matrix builder that accumulates Coulomb (J) and exchange (K) contributions:

```
J_μν  += Σ_λσ (μν|λσ) D_λσ
K_μλ  += Σ_νσ (μν|λσ) D_νσ
```

The consumer receives a buffer of integrals for one ShellSetQuartet at a time, reads the density matrix D, and accumulates into J and K. The integral buffer is then reused for the next batch. Peak memory usage is proportional to one ShellSetQuartet buffer plus the O(N^2) accumulator matrices, regardless of how many total integrals exist.

In libaccint, the `FockBuilder` consumer implements this pattern. A typical usage is:

```cpp
FockBuilder fock_builder(nbf);
fock_builder.set_density(D.data(), nbf);
engine.compute(Operator::coulomb(), fock_builder);
// J and K are now fully accumulated
auto J = fock_builder.get_coulomb_matrix();
auto K = fock_builder.get_exchange_matrix();
```

The `Engine::compute` method iterates over all ShellSetQuartets in the basis, computes each batch (on CPU or GPU as the dispatch policy dictates), and feeds the results to the consumer. The consumer interface is generic: any type with the correct `accumulate` signature can serve as a consumer, enabling custom reductions for density fitting, Coulomb metric assembly, or gradient contractions.

On the GPU path, a `GpuFockBuilder` keeps the J and K matrices in device memory and accumulates using atomic operations, eliminating per-batch device-to-host transfers. The density matrix is uploaded once, and the result matrices are downloaded once at the end.

## Screening Integration

Schwarz screening eliminates shell quartets whose integrals are guaranteed to be below a threshold. The Schwarz inequality provides an upper bound:

```
|(μν|λσ)| <= sqrt((μν|μν)) * sqrt((λσ|λσ))
```

For large molecules with diffuse basis sets, 90-99% of quartets are screened out. This is essential for achieving near-linear scaling in practice.

Screening must happen before batch assembly, not inside the GPU kernel. Including screened-out quartets in a ShellSetQuartet batch wastes GPU threads on zero-contribution work. The correct pipeline is:

1. Precompute Schwarz bounds: diagonal integrals (μν|μν) for all shell pairs.
2. For each candidate shell quartet, check the Schwarz bound against the threshold.
3. Collect surviving quartets into a worklist.
4. Group the worklist by angular momentum class into ShellSetQuartets.
5. Dispatch each ShellSetQuartet to the appropriate backend.

Density-weighted screening tightens the bound further during SCF iterations. Instead of the bare Schwarz bound, the effective bound becomes:

```
max_λσ |D_λσ| * sqrt((μν|μν)) * sqrt((λσ|λσ))
```

As the density matrix converges and many elements shrink toward zero, more quartets are screened. Early SCF iterations compute more quartets; later iterations are faster.

Eight-fold permutation symmetry (swapping bra/ket shells, swapping within bra, swapping within ket, swapping bra with ket) reduces the number of unique quartets by up to 8x. Combined with screening, the effective workload for a well-implemented code is a small fraction of the formal O(N^4) count.

## OpenMP and SIMD on the CPU Path

For small batches and low angular momentum, multi-threaded CPU execution with SIMD vectorization often outperforms GPU dispatch. The kernel launch overhead for a GPU (typically 5-20 microseconds) exceeds the time to compute a small (ss|sp) batch entirely on CPU with AVX2 instructions.

AVX2 provides 4-wide double-precision SIMD (256-bit registers). Four identical shell quartets from the same ShellSetQuartet are interleaved across the four SIMD lanes. Each recurrence step, each primitive contraction, and each normalization operation executes on four quartets simultaneously. This is structurally similar to the GPU approach (same work in every lane) but avoids launch overhead entirely.

The CPU path parallelizes across ShellSetQuartets using OpenMP. Each thread processes one batch at a time with dynamic scheduling to handle the varying batch sizes. Thread-safety for the consumer is handled through one of three strategies:

- **Sequential:** No thread safety. Single-threaded only.
- **Atomic:** Accumulation into J and K uses atomic additions. Simple but suffers contention for small matrices.
- **Thread-local:** Each thread accumulates into private J and K copies. A final reduction sums them. Higher memory usage but zero contention.

The thread-local strategy is the default for parallel Fock builds in libaccint. Each thread calls `prepare_parallel(n_threads)` before the parallel region and `finalize_parallel()` after it to trigger the reduction.

Hybrid CPU+GPU dispatch falls out of the per-batch dispatch policy directly. Low-AM batches run on CPU threads; high-AM batches run on GPU. Both paths feed the same consumer (with appropriate thread safety). The `Auto` dispatch hint enables this without user intervention.

## Multi-GPU and Work-Stealing

For large molecules, a single GPU saturates and additional devices provide near-linear speedup. The challenge is distributing work evenly across devices with different completion rates.

Static partitioning assigns a fixed fraction of ShellSetQuartets to each device based on estimated cost. This works well when the cost model is accurate, but mispredictions create stragglers: one device finishes early and waits while another handles a disproportionate share.

Work-stealing eliminates this problem. Each device owns a double-ended queue (deque) of work items. A device pops from the back of its own queue (LIFO, for cache locality in the partitioning order). When its queue is empty, it steals from the front of another device's queue (FIFO, to minimize contention with the owning device).

libaccint's `MultiGPUEngine` implements this pattern with `WorkStealingQueue`:

```
per-device thread:
    while own_queue.try_pop(idx):    // LIFO from own queue
        process(quartets[idx])
    while global_remaining > 0:
        for each other device:
            if other_queue.try_steal(idx):  // FIFO from victim
                process(quartets[idx])
                break
```

Resource-aware weighting improves the initial partition. Before distributing work, the engine queries each device's SM availability and memory pressure through `update_resource_aware_weights()`. A device running a concurrent workload receives fewer initial assignments. The work-stealing mechanism handles any remaining imbalance dynamically.

Load balance efficiency is measured as the ratio of average per-device time to maximum per-device time. Values above 0.9 indicate effective distribution. The `MultiGPUStats` structure reports this metric along with per-device quartet counts and timing breakdowns.

## Practical Considerations

**Mixed precision.** Consumer-grade GPUs (RTX series) offer 2x higher throughput for single precision compared to double. For Coulomb integrals where the density matrix provides natural error damping, computing integrals in FP32 and accumulating in FP64 can double throughput with sub-microhartree impact on total energies. This is useful for initial SCF iterations where the density is far from converged.

**MPI distribution.** In a distributed-memory cluster, each MPI rank computes a subset of shell quartets and accumulates a local Fock matrix. A single all-reduce (summation) over the J and K matrices produces the global Fock matrix. Since J and K are O(N^2), the communication cost is small compared to the O(N^4) computation. Each rank can independently use multi-GPU dispatch within its node.

**Density fitting.** The resolution-of-the-identity (RI) approximation replaces four-center ERIs with products of two- and three-center integrals. The work unit shape changes (three-index tensors instead of four-index), but the same batching and dispatch principles apply. ShellSetTriplets group three-center integrals by AM class. Consumers accumulate into the RI coefficient matrices. GPU dispatch benefits from the larger per-integral workload at high AM.

**Profiling over kernel speed.** A common mistake is to focus on kernel optimization (faster recurrence, better register usage) while neglecting work organization. A perfectly tuned kernel processing a poorly organized workload (mixed AM classes, no screening, no consumer pattern) will be slower than a generic kernel processing well-organized batches. Measure dispatch overhead, screening pass rates, consumer accumulation time, and load balance before optimizing kernel internals.

None of these techniques are specific to any one integral library: ShellSet batching, adaptive dispatch, consumer-driven accumulation, pre-kernel screening, work-stealing across devices. They are the architectural decisions that determine whether a GPU-accelerated integral code achieves 10% or 90% of peak hardware throughput. The kernel matters, but the framework around it matters more.
