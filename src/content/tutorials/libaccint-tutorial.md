---
title: "libaccint Tutorial: Building Hartree-Fock Programs in C++ and Python"
date: 2026-02-06
description: "An end-to-end guide to molecular integral computation with libaccint: from molecule definition through a complete RHF SCF calculation, with side-by-side C++ and Python examples."
level: "advanced"
tags: ["libaccint", "quantum-chemistry", "hartree-fock", "tutorial", "molecular-integrals"]
---

libaccint is a molecular integral library with transparent CPU/GPU dispatch. Its central abstraction is the `Engine`, which owns both CPU and (optionally) GPU backends behind a unified `compute(...)` interface. One-electron integrals (overlap, kinetic, nuclear attraction) return dense matrices. Two-electron integrals flow through a consumer pattern: the `Engine` computes batches of electron repulsion integrals and feeds them to a consumer object (such as `FockBuilder`) that accumulates Coulomb and exchange contributions on the fly, without materializing the full O(N^4) tensor.

This tutorial walks through a complete Restricted Hartree-Fock (RHF) calculation for water. Each section shows both C++ and Python code. By the end, you will have two self-contained programs that produce the H2O/cc-pVDZ RHF energy (~-76.02 Hartree).

**Prerequisites:** Basic familiarity with Hartree-Fock theory, C++17 or Python 3.8+, and NumPy/SciPy for the Python path. For background on the batching and dispatch architecture, see the companion post on [GPU-accelerated molecular integrals](/blog/gpu-accelerated-integrals).

## Installation and Setup

**C++ (CMake):**

```bash
cmake --preset cpu-release
cmake --build --preset cpu-release
```

For GPU support, use the `cuda-release` preset instead.

**Python (pip):**

```bash
pip install libaccint
```

**Version and capability check:**

```python
import libaccint as lai
print(f"libaccint {lai.__version__}")
print(f"CUDA backend: {lai.has_cuda_backend()}")
```

```cpp
#include <libaccint/libaccint.hpp>
std::cout << "libaccint " << libaccint::version() << "\n";
std::cout << "CUDA: " << libaccint::has_cuda_backend() << "\n";
```

## Defining the Molecule

Atomic coordinates are specified in Bohr (atomic units). Each atom is defined by its atomic number Z and a position vector [x, y, z].

**Python:**

```python
import libaccint as lai

atoms = [
    lai.Atom(8, [0.000000,  0.000000,  0.117176]),   # O
    lai.Atom(1, [0.000000,  1.430665, -0.468706]),   # H
    lai.Atom(1, [0.000000, -1.430665, -0.468706]),   # H
]
n_electrons = 10
n_occ = n_electrons // 2  # 5 doubly-occupied MOs
```

**C++:**

```cpp
using namespace libaccint;
using namespace libaccint::data;

std::vector<Atom> atoms = {
    {8, {0.000000,  0.000000,  0.117176}},   // O
    {1, {0.000000,  1.430665, -0.468706}},   // H
    {1, {0.000000, -1.430665, -0.468706}},   // H
};
const int n_electrons = 10;
const int n_occ = n_electrons / 2;
```

This geometry corresponds to the equilibrium structure of water with a bond angle of ~104.5 degrees.

## Loading a Basis Set

libaccint bundles 40+ basis sets from the Basis Set Exchange.

**Python:**

```python
basis = lai.basis_set("cc-pVDZ", atoms)
nbf = basis.n_basis_functions()
print(f"Basis functions: {nbf}, Shells: {basis.n_shells()}")
```

**C++:**

```cpp
BasisSet basis = load_basis_set("cc-pvdz", atoms);
const int nbf = static_cast<int>(basis.n_basis_functions());
std::cout << "Basis functions: " << nbf
          << ", Shells: " << basis.n_shells() << "\n";
```

Names are case-insensitive. Pople star notation works: `"6-31G*"` and `"6-31G**"` resolve automatically.

Available families include:

| Family | Examples |
|--------|----------|
| Pople | STO-3G, 3-21G, 6-31G, 6-31G*, 6-311G** |
| Dunning | cc-pVDZ, cc-pVTZ, cc-pVQZ, aug-cc-pVDZ |
| Karlsruhe def2 | def2-SVP, def2-TZVP, def2-QZVP |
| Auxiliary (RI/JK) | cc-pVTZ-JKFIT, def2-universal-jkfit |

To list all available names:

```python
names = lai.list_available_basis_sets()
```

The `BasisSet` object also provides `max_angular_momentum()` for querying the highest L present.

## Creating the Engine

The `Engine` owns both CPU and GPU backends and routes work automatically.

**Python:**

```python
engine = lai.Engine(basis)
print(f"GPU available: {engine.gpu_available()}")
```

**C++:**

```cpp
Engine engine(basis);
std::cout << "GPU available: " << engine.gpu_available() << "\n";
```

The default `DispatchConfig` is suitable for most workloads. For custom dispatch tuning:

```cpp
DispatchConfig config;
config.min_gpu_batch_size = 32;
config.enable_auto_tuning = true;
Engine engine(basis, config);
```

## One-Electron Integrals

The overlap (S), kinetic energy (T), and nuclear attraction (V) matrices are computed via convenience methods that return full nbf x nbf matrices.

**Python:**

```python
S = engine.compute_overlap_matrix()     # numpy (nbf, nbf)
T = engine.compute_kinetic_matrix()     # numpy (nbf, nbf)
V = engine.compute_nuclear_matrix(atoms)  # atoms list directly
H_core = T + V
```

In Python, `compute_nuclear_matrix` accepts the atom list directly and handles the `PointChargeParams` construction internally.

**C++:**

```cpp
// Build nuclear charge parameters (SoA layout)
PointChargeParams charges;
for (const auto& atom : atoms) {
    charges.x.push_back(atom.position.x);
    charges.y.push_back(atom.position.y);
    charges.z.push_back(atom.position.z);
    charges.charge.push_back(static_cast<Real>(atom.atomic_number));
}

std::vector<Real> S_flat, T_flat, V_flat;
engine.compute_overlap_matrix(S_flat);
engine.compute_kinetic_matrix(T_flat);
engine.compute_nuclear_matrix(charges, V_flat);
```

In C++, results are flat row-major `std::vector<Real>`. You will typically convert to Eigen matrices for linear algebra:

```cpp
Eigen::MatrixXd S = to_eigen(S_flat, nbf);
Eigen::MatrixXd T = to_eigen(T_flat, nbf);
Eigen::MatrixXd V = to_eigen(V_flat, nbf);
Eigen::MatrixXd H_core = T + V;
```

There is also a one-shot core Hamiltonian method:

```python
H_core = engine.compute_core_hamiltonian(atoms)
```

```cpp
std::vector<Real> H_flat;
engine.compute_core_hamiltonian(charges, H_flat);
```

## Two-Electron Integrals via FockBuilder

The `FockBuilder` consumer accumulates Coulomb (J) and exchange (K) matrices as two-electron integrals are computed batch by batch. This avoids storing the full ERI tensor. (See [the consumer pattern](/blog/gpu-accelerated-integrals#the-consumer-pattern) for the design rationale.)

**Python:**

```python
fock = lai.FockBuilder(nbf)
fock.set_density(D)  # D is a (nbf, nbf) numpy array
engine.compute(lai.Operator.coulomb(), fock)

J = fock.get_coulomb_matrix()
K = fock.get_exchange_matrix()
F = H_core + J - 0.5 * K
```

**C++:**

```cpp
using namespace libaccint::consumers;

FockBuilder fock_builder(static_cast<Size>(nbf));
std::vector<Real> D_flat = from_eigen(D, nbf);
fock_builder.set_density(D_flat.data(), static_cast<Size>(nbf));

engine.compute_and_consume(Operator::coulomb(), fock_builder);

auto J_span = fock_builder.get_coulomb_matrix();
auto K_span = fock_builder.get_exchange_matrix();
// Copy spans to Eigen matrices, then:
Eigen::MatrixXd F = H_core + J - 0.5 * K;
```

The `FockBuilder` lifecycle:

1. **Construct** with the basis size.
2. **Set density** for the current SCF iteration.
3. **Compute**: `engine.compute(Operator::coulomb(), fock_builder)` iterates all ShellSetQuartets internally.
4. **Extract** J and K matrices.
5. **Reset** (via `fock.reset()`) if reusing across iterations, or construct a new one.

The Fock matrix is F = H_core + J - 0.5 * K. The 0.5 factor arises because the density matrix D includes a factor of 2 from double occupancy (Szabo and Ostlund convention for closed-shell RHF).

## The SCF Loop

A complete SCF iteration:

1. Build the Fock matrix F from the current density D.
2. Transform to the orthogonal basis: F' = X^T F X, where X = S^{-1/2}.
3. Diagonalize F' to get orbital energies and coefficients.
4. Back-transform coefficients: C = X C'.
5. Build new density: D = 2 C_occ C_occ^T.
6. Check convergence (energy change and density change).

**Canonical orthogonalization** produces X = S^{-1/2} via eigendecomposition of S:

**Python:**

```python
from scipy import linalg

s_vals, U = linalg.eigh(S)
s_inv_sqrt = np.zeros_like(s_vals)
mask = s_vals > 1e-10
s_inv_sqrt[mask] = 1.0 / np.sqrt(s_vals[mask])
X = U @ np.diag(s_inv_sqrt) @ U.T
```

**C++:**

```cpp
Eigen::SelfAdjointEigenSolver<Eigen::MatrixXd> es(S);
Eigen::VectorXd s_vals = es.eigenvalues();
Eigen::MatrixXd U = es.eigenvectors();

Eigen::VectorXd s_inv_sqrt(s_vals.size());
for (int i = 0; i < s_vals.size(); ++i) {
    s_inv_sqrt(i) = (s_vals(i) > 1e-10) ? 1.0 / std::sqrt(s_vals(i)) : 0.0;
}
Eigen::MatrixXd X = U * s_inv_sqrt.asDiagonal() * U.transpose();
```

**DIIS acceleration** typically cuts the iteration count by 2-3x. The error vector e = FDS - SDF vanishes at self-consistency. DIIS maintains a history of Fock matrices and error vectors, then extrapolates an improved Fock matrix via least-squares minimization of the error norm.

**Python DIIS class:**

```python
class DIIS:
    def __init__(self, max_size=6):
        self.max_size = max_size
        self.fock_list, self.error_list = [], []

    def add(self, F, error):
        self.fock_list.append(F.copy())
        self.error_list.append(error.copy())
        if len(self.fock_list) > self.max_size:
            self.fock_list.pop(0)
            self.error_list.pop(0)

    def extrapolate(self):
        n = len(self.fock_list)
        if n < 2:
            return self.fock_list[-1]
        B = np.zeros((n + 1, n + 1))
        for i in range(n):
            for j in range(n):
                B[i, j] = np.sum(self.error_list[i] * self.error_list[j])
        B[n, :n] = -1.0
        B[:n, n] = -1.0
        rhs = np.zeros(n + 1)
        rhs[n] = -1.0
        c = np.linalg.solve(B, rhs)
        return sum(c[i] * self.fock_list[i] for i in range(n))
```

The full SCF iteration (Python, with DIIS):

```python
# Initial guess: diagonalize H_core
H_prime = X.T @ H_core @ X
eps, C_prime = linalg.eigh(H_prime)
C = X @ C_prime
D = 2.0 * C[:, :n_occ] @ C[:, :n_occ].T

E_old = 0.0
diis = DIIS(max_size=6)

for iteration in range(100):
    # Build Fock matrix
    fock = lai.FockBuilder(nbf)
    fock.set_density(np.ascontiguousarray(D))
    engine.compute(lai.Operator.coulomb(), fock)
    J = fock.get_coulomb_matrix()
    K = fock.get_exchange_matrix()
    F = H_core + J - 0.5 * K

    # Electronic energy
    E_elec = 0.5 * np.sum((H_core + F) * D)
    E_total = E_elec + E_nuc

    # DIIS extrapolation
    error = F @ D @ S - S @ D @ F
    diis.add(F, error)
    F_diis = diis.extrapolate()

    # Diagonalize
    F_prime = X.T @ F_diis @ X
    eps, C_prime = linalg.eigh(F_prime)
    C = X @ C_prime
    D_new = 2.0 * C[:, :n_occ] @ C[:, :n_occ].T

    # Convergence check
    dE = abs(E_total - E_old)
    dD = np.max(np.abs(D_new - D))
    if dE < 1e-10 and dD < 1e-8 and iteration > 0:
        print(f"Converged in {iteration + 1} iterations")
        break

    D = D_new
    E_old = E_total
```

## GPU and Parallel Options

### BackendHint

Pass a `BackendHint` to any compute call to control dispatch:

**Python:**

```python
engine.compute(lai.Operator.coulomb(), fock, lai.BackendHint.PreferGPU)
```

**C++:**

```cpp
engine.compute_and_consume(Operator::coulomb(), fock_builder, BackendHint::ForceGPU);
```

`PreferGPU` falls back to CPU if no GPU is compiled in or detected. `ForceGPU` raises an error if the GPU is unavailable.

### OpenMP Parallelism

For multi-threaded CPU execution:

**C++:**

```cpp
engine.compute_and_consume_parallel(Operator::coulomb(), fock_builder, /*n_threads=*/4);
```

The consumer must support parallel accumulation. `FockBuilder` provides three threading strategies:

```cpp
fock_builder.set_threading_strategy(FockThreadingStrategy::ThreadLocal);
fock_builder.prepare_parallel(4);
engine.compute_and_consume_parallel(Operator::coulomb(), fock_builder, 4);
fock_builder.finalize_parallel();
```

### GpuFockBuilder

For device-side accumulation (eliminates per-batch data transfers):

```cpp
#include <libaccint/consumers/fock_builder_gpu.hpp>

consumers::GpuFockBuilder gpu_fock(nbf);
gpu_fock.set_density(D_flat.data(), nbf);
engine.compute_and_consume(Operator::coulomb(), gpu_fock, BackendHint::ForceGPU);
gpu_fock.synchronize();
auto J = gpu_fock.get_coulomb_matrix();
auto K = gpu_fock.get_exchange_matrix();
```

In Python:

```python
gpu_fock = lai.GpuFockBuilder(nbf)
gpu_fock.set_density(D)
engine.compute(lai.Operator.coulomb(), gpu_fock, lai.BackendHint.ForceGPU)
gpu_fock.synchronize()
```

### DispatchConfig Tuning

```cpp
DispatchConfig config;
config.min_gpu_batch_size = 32;
config.min_gpu_primitives = 1000;
config.high_am_threshold = 4;
config.enable_auto_tuning = true;
config.auto_tune_mode = kernels::KernelCalculator::Mode::ProfileOnce;
config.n_gpu_slots = 2;  // for memory-limited GPUs
Engine engine(basis, config);
```

## Schwarz Screening

For larger molecules, Schwarz screening eliminates insignificant shell quartets before computation.

**C++:**

```cpp
screening::ScreeningOptions opts = screening::ScreeningOptions::normal();
engine.compute_and_consume_screened_parallel(
    Operator::coulomb(), fock_builder, opts, /*n_threads=*/4);
```

Three presets are available:

| Preset | Threshold | Use case |
|--------|-----------|----------|
| `loose()` | 1e-10 | Fast initial SCF iterations |
| `normal()` | 1e-12 | Production calculations (default) |
| `tight()` | 1e-14 | High-precision or benchmark work |

Density-weighted screening tightens bounds using the current density matrix, filtering more quartets as the SCF converges:

```cpp
screening::ScreeningOptions opts = screening::ScreeningOptions::normal();
opts.density_weighted = true;
```

The `SchwarzBounds` class provides diagnostic information. Call `estimate_pass_fraction(threshold)` to see what fraction of quartets survive screening at a given threshold.

## Supported Operators

### One-Electron Operators

| Operator | Factory method | Description |
|----------|---------------|-------------|
| Overlap (S) | `Operator::overlap()` | Basis function overlap |
| Kinetic (T) | `Operator::kinetic()` | Kinetic energy |
| Nuclear (V) | `Operator::nuclear(charges)` | Nuclear attraction |

One-electron operators can be composed:

```cpp
OneElectronOperator H = Operator::kinetic() + Operator::nuclear(charges);
engine.compute_1e(H, result);
```

### Two-Electron Operators

| Operator | Factory method | Description |
|----------|---------------|-------------|
| Coulomb | `Operator::coulomb()` | Standard 1/r_12 |
| erf-Coulomb | `Operator::erf_coulomb(omega)` | Long-range erf(omega * r_12) / r_12 |
| erfc-Coulomb | `Operator::erfc_coulomb(omega)` | Short-range erfc(omega * r_12) / r_12 |

The range-separated operators enable range-separated hybrid DFT functionals. The Coulomb operator decomposes as:

```
1/r_12 = erf(omega * r_12)/r_12  +  erfc(omega * r_12)/r_12
```

In Python, the operator factory methods are accessed on the `Operator` class:

```python
op_coulomb = lai.Operator.coulomb()
op_lr = lai.Operator.erf_coulomb(0.33)
op_sr = lai.Operator.erfc_coulomb(0.33)
```

## Complete Programs

### Python: H2O/cc-pVDZ RHF

```python
import numpy as np
from scipy import linalg
import libaccint as lai

# Molecule
atoms = [
    lai.Atom(8, [0.000000,  0.000000,  0.117176]),
    lai.Atom(1, [0.000000,  1.430665, -0.468706]),
    lai.Atom(1, [0.000000, -1.430665, -0.468706]),
]
n_occ = 5

# Basis and engine
basis = lai.basis_set("cc-pVDZ", atoms)
engine = lai.Engine(basis)
nbf = basis.n_basis_functions()

# Nuclear repulsion
E_nuc = 0.0
for i in range(len(atoms)):
    for j in range(i + 1, len(atoms)):
        ri, rj = atoms[i].position, atoms[j].position
        r = np.sqrt((ri.x-rj.x)**2 + (ri.y-rj.y)**2 + (ri.z-rj.z)**2)
        E_nuc += atoms[i].atomic_number * atoms[j].atomic_number / r

# One-electron integrals
S = engine.compute_overlap_matrix()
H_core = engine.compute_core_hamiltonian(atoms)

# Orthogonalization matrix X = S^{-1/2}
s_vals, U = linalg.eigh(S)
s_inv_sqrt = np.zeros_like(s_vals)
s_inv_sqrt[s_vals > 1e-10] = 1.0 / np.sqrt(s_vals[s_vals > 1e-10])
X = U @ np.diag(s_inv_sqrt) @ U.T

# Initial guess from H_core
eps, C_prime = linalg.eigh(X.T @ H_core @ X)
C = X @ C_prime
D = 2.0 * C[:, :n_occ] @ C[:, :n_occ].T

# DIIS
fock_hist, err_hist, max_diis = [], [], 6

def diis_extrapolate():
    n = len(fock_hist)
    if n < 2:
        return fock_hist[-1]
    B = np.zeros((n+1, n+1))
    for i in range(n):
        for j in range(n):
            B[i,j] = np.sum(err_hist[i] * err_hist[j])
    B[n,:n] = B[:n,n] = -1.0
    rhs = np.zeros(n+1); rhs[n] = -1.0
    c = np.linalg.solve(B, rhs)
    return sum(c[i]*fock_hist[i] for i in range(n))

# SCF loop
E_old = 0.0
for it in range(100):
    fock = lai.FockBuilder(nbf)
    fock.set_density(np.ascontiguousarray(D))
    engine.compute(lai.Operator.coulomb(), fock)
    F = H_core + fock.get_coulomb_matrix() - 0.5 * fock.get_exchange_matrix()

    E_elec = 0.5 * np.sum((H_core + F) * D)
    E_total = E_elec + E_nuc

    error = F @ D @ S - S @ D @ F
    fock_hist.append(F.copy()); err_hist.append(error.copy())
    if len(fock_hist) > max_diis:
        fock_hist.pop(0); err_hist.pop(0)
    F_diis = diis_extrapolate()

    eps, C_prime = linalg.eigh(X.T @ F_diis @ X)
    C = X @ C_prime
    D_new = 2.0 * C[:, :n_occ] @ C[:, :n_occ].T

    dE = abs(E_total - E_old)
    dD = np.max(np.abs(D_new - D))
    print(f"{it:3d}  E={E_total:.12f}  dE={dE:.2e}  dD={dD:.2e}")

    if dE < 1e-10 and dD < 1e-8 and it > 0:
        D = D_new
        fock_f = lai.FockBuilder(nbf)
        fock_f.set_density(np.ascontiguousarray(D))
        engine.compute(lai.Operator.coulomb(), fock_f)
        F_f = H_core + fock_f.get_coulomb_matrix() - 0.5 * fock_f.get_exchange_matrix()
        E_total = 0.5 * np.sum((H_core + F_f) * D) + E_nuc
        print(f"\nRHF energy: {E_total:.12f} Hartree")
        break

    D = D_new
    E_old = E_total
```

### C++: H2O/cc-pVDZ RHF

```cpp
#include <libaccint/libaccint.hpp>
#include <libaccint/consumers/fock_builder.hpp>
#include <libaccint/data/basis_parser.hpp>
#include <libaccint/data/builtin_basis.hpp>
#include <Eigen/Dense>
#include <cmath>
#include <iomanip>
#include <iostream>
#include <vector>

using namespace libaccint;
using namespace libaccint::data;
using namespace libaccint::consumers;

Eigen::MatrixXd to_eigen(const std::vector<Real>& flat, int n) {
    Eigen::MatrixXd mat(n, n);
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < n; ++j)
            mat(i, j) = flat[i * n + j];
    return mat;
}

std::vector<Real> from_eigen(const Eigen::MatrixXd& mat, int n) {
    std::vector<Real> flat(n * n);
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < n; ++j)
            flat[i * n + j] = mat(i, j);
    return flat;
}

int main() {
    // Molecule: H2O in Bohr
    std::vector<Atom> atoms = {
        {8, {0.000000,  0.000000,  0.117176}},
        {1, {0.000000,  1.430665, -0.468706}},
        {1, {0.000000, -1.430665, -0.468706}},
    };
    const int n_occ = 5;

    // Basis set and engine
    BasisSet basis = load_basis_set("cc-pvdz", atoms);
    const int nbf = static_cast<int>(basis.n_basis_functions());
    Engine engine(basis);

    // Nuclear repulsion
    Real E_nuc = 0.0;
    for (size_t i = 0; i < atoms.size(); ++i)
        for (size_t j = i + 1; j < atoms.size(); ++j) {
            Real dx = atoms[i].position.x - atoms[j].position.x;
            Real dy = atoms[i].position.y - atoms[j].position.y;
            Real dz = atoms[i].position.z - atoms[j].position.z;
            E_nuc += static_cast<Real>(atoms[i].atomic_number *
                     atoms[j].atomic_number) / std::sqrt(dx*dx + dy*dy + dz*dz);
        }

    // One-electron integrals
    PointChargeParams charges;
    for (const auto& atom : atoms) {
        charges.x.push_back(atom.position.x);
        charges.y.push_back(atom.position.y);
        charges.z.push_back(atom.position.z);
        charges.charge.push_back(static_cast<Real>(atom.atomic_number));
    }

    std::vector<Real> S_flat, T_flat, V_flat;
    engine.compute_overlap_matrix(S_flat);
    engine.compute_kinetic_matrix(T_flat);
    engine.compute_nuclear_matrix(charges, V_flat);

    Eigen::MatrixXd S = to_eigen(S_flat, nbf);
    Eigen::MatrixXd H_core = to_eigen(T_flat, nbf) + to_eigen(V_flat, nbf);

    // Orthogonalization: X = S^{-1/2}
    Eigen::SelfAdjointEigenSolver<Eigen::MatrixXd> es(S);
    Eigen::VectorXd s_inv_sqrt(nbf);
    for (int i = 0; i < nbf; ++i)
        s_inv_sqrt(i) = (es.eigenvalues()(i) > 1e-10)
            ? 1.0 / std::sqrt(es.eigenvalues()(i)) : 0.0;
    Eigen::MatrixXd X = es.eigenvectors() * s_inv_sqrt.asDiagonal()
                       * es.eigenvectors().transpose();

    // Initial guess
    Eigen::SelfAdjointEigenSolver<Eigen::MatrixXd> es2(X.transpose() * H_core * X);
    Eigen::MatrixXd C = X * es2.eigenvectors();
    Eigen::MatrixXd D = 2.0 * C.leftCols(n_occ) * C.leftCols(n_occ).transpose();

    // DIIS storage
    std::vector<Eigen::MatrixXd> fock_hist, err_hist;
    const int max_diis = 6;

    auto diis_extrapolate = [&]() -> Eigen::MatrixXd {
        int n = static_cast<int>(fock_hist.size());
        if (n < 2) return fock_hist.back();
        Eigen::MatrixXd B = Eigen::MatrixXd::Zero(n+1, n+1);
        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                B(i,j) = (err_hist[i].array() * err_hist[j].array()).sum();
        for (int i = 0; i < n; ++i) { B(n,i) = -1.0; B(i,n) = -1.0; }
        Eigen::VectorXd rhs = Eigen::VectorXd::Zero(n+1); rhs(n) = -1.0;
        Eigen::VectorXd c = B.colPivHouseholderQr().solve(rhs);
        Eigen::MatrixXd F_new = Eigen::MatrixXd::Zero(nbf, nbf);
        for (int i = 0; i < n; ++i) F_new += c(i) * fock_hist[i];
        return F_new;
    };

    // SCF loop
    Real E_old = 0.0;
    for (int it = 0; it < 100; ++it) {
        FockBuilder fock(static_cast<Size>(nbf));
        auto D_flat = from_eigen(D, nbf);
        fock.set_density(D_flat.data(), static_cast<Size>(nbf));
        engine.compute_and_consume(Operator::coulomb(), fock);

        auto J_span = fock.get_coulomb_matrix();
        auto K_span = fock.get_exchange_matrix();
        Eigen::MatrixXd J(nbf, nbf), K(nbf, nbf);
        for (int i = 0; i < nbf; ++i)
            for (int j = 0; j < nbf; ++j) {
                J(i,j) = J_span[i*nbf+j];
                K(i,j) = K_span[i*nbf+j];
            }
        Eigen::MatrixXd F = H_core + J - 0.5 * K;

        Real E_elec = 0.5 * ((H_core + F).array() * D.array()).sum();
        Real E_total = E_elec + E_nuc;

        Eigen::MatrixXd error = F * D * S - S * D * F;
        fock_hist.push_back(F); err_hist.push_back(error);
        if (static_cast<int>(fock_hist.size()) > max_diis) {
            fock_hist.erase(fock_hist.begin());
            err_hist.erase(err_hist.begin());
        }
        Eigen::MatrixXd F_diis = diis_extrapolate();

        Eigen::SelfAdjointEigenSolver<Eigen::MatrixXd> solver(
            X.transpose() * F_diis * X);
        C = X * solver.eigenvectors();
        Eigen::MatrixXd D_new = 2.0 * C.leftCols(n_occ)
                                * C.leftCols(n_occ).transpose();

        Real dE = std::abs(E_total - E_old);
        Real dD = (D_new - D).array().abs().maxCoeff();

        std::cout << std::setw(3) << it
                  << std::fixed << std::setprecision(12)
                  << "  E=" << E_total
                  << std::scientific << std::setprecision(2)
                  << "  dE=" << dE << "  dD=" << dD << "\n";

        if (dE < 1e-10 && dD < 1e-8 && it > 0) {
            std::cout << "\nRHF energy: " << std::fixed
                      << std::setprecision(12) << E_total << " Hartree\n";
            return 0;
        }

        D = D_new;
        E_old = E_total;
    }
    std::cerr << "SCF did not converge.\n";
    return 1;
}
```

Both programs produce an RHF energy of approximately -76.02 Hartree for H2O/cc-pVDZ.

## Where to Go Next

- **Custom consumers.** Implement the `accumulate` interface to build custom reductions (gradient contractions, density fitting coefficients, MP2 amplitudes).
- **Density fitting.** Use auxiliary basis sets (cc-pVTZ-JKFIT, def2-universal-jkfit) with three-center integrals for O(N^3) Coulomb and exchange.
- **Range-separated DFT.** Combine `Operator::erf_coulomb(omega)` and `Operator::erfc_coulomb(omega)` for long-range/short-range decomposition.
- **Multi-GPU.** Use `MultiGPUEngine` with `MultiGPUConfig` for multi-device parallelism with work-stealing load balancing.
- **Documentation.** Full API reference at [libaccint.readthedocs.io](https://libaccint.readthedocs.io). Source and examples at [github.com/brycewestheimer/libaccint](https://github.com/brycewestheimer/libaccint).
