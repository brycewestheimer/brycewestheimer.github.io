---
title: "Python for Quantum Chemistry: NumPy and SciPy Fundamentals"
date: 2025-11-07
description: "Learn how to use Python's scientific computing libraries for quantum chemistry calculations and data analysis"
level: "intermediate"
tags: ["Python", "NumPy", "SciPy", "computational-chemistry"]
---

Python has become the default scripting language for computational chemistry. Not because it is fast (it is not), but because NumPy and SciPy give you access to compiled BLAS, LAPACK, and special-function libraries through a syntax that reads almost like the equations in a textbook. You can prototype a Hartree-Fock program in an afternoon, run it on small molecules to verify your understanding, and then use the same code as a reference implementation to test a production C++ or Fortran code.

This tutorial works through quantum chemistry problems of increasing complexity using NumPy and SciPy. It starts with molecular geometry manipulation, moves through basis function integrals, and finishes with a working restricted Hartree-Fock SCF program for H2 in a minimal basis. Along the way it covers the NumPy idioms and SciPy functions that show up repeatedly in computational chemistry code.

## Prerequisites

You need Python 3.8 or later with NumPy, SciPy, and Matplotlib. Install them in a virtual environment:

```bash
python -m venv qchem-env
source qchem-env/bin/activate   # Linux/Mac
pip install numpy scipy matplotlib
```

On Windows, replace the `source` line with `qchem-env\Scripts\activate`.

Verify:

```python
import numpy as np
import scipy
print(f"NumPy {np.__version__}, SciPy {scipy.__version__}")
```

All code in this tutorial runs in a Jupyter notebook or as a plain Python script. No quantum chemistry software is needed.

## Part 1: Molecular Geometry with NumPy

### Coordinate arrays and distance matrices

Molecular coordinates are naturally represented as NumPy arrays of shape `(n_atoms, 3)`. One row per atom, three columns for x, y, z.

```python
import numpy as np

# Water molecule, coordinates in Angstroms
labels = ["O", "H", "H"]
coords = np.array([
    [0.000,  0.000,  0.117],   # O
    [0.000,  0.757, -0.469],   # H
    [0.000, -0.757, -0.469],   # H
])
```

A distance matrix between all pairs of atoms is a common first step in any geometry analysis. The broadcasting trick avoids an explicit double loop:

```python
def distance_matrix(coords):
    """Pairwise distance matrix from an (n_atoms, 3) coordinate array."""
    diff = coords[:, np.newaxis, :] - coords[np.newaxis, :, :]
    return np.sqrt(np.sum(diff**2, axis=2))

D = distance_matrix(coords)
print("Distance matrix (Angstroms):")
print(np.array2string(D, precision=4))
```

`coords[:, np.newaxis, :]` has shape `(3, 1, 3)` and `coords[np.newaxis, :, :]` has shape `(1, 3, 3)`. Subtraction broadcasts to `(3, 3, 3)`, giving the displacement vector between every pair. Squaring, summing over the last axis, and taking the square root gives the 3x3 distance matrix.

For water, the O-H distances should be about 0.958 A and the H-H distance about 1.514 A.

### Bond angles

The angle between atoms A-B-C (with B at the vertex) is computed from the dot product of the BA and BC vectors:

```python
def bond_angle(coords, i, j, k):
    """Angle at atom j formed by atoms i-j-k, in degrees."""
    v1 = coords[i] - coords[j]
    v2 = coords[k] - coords[j]
    cos_angle = np.dot(v1, v2) / (np.linalg.norm(v1) * np.linalg.norm(v2))
    cos_angle = np.clip(cos_angle, -1.0, 1.0)  # numerical safety
    return np.degrees(np.arccos(cos_angle))

angle_HOH = bond_angle(coords, 1, 0, 2)
print(f"H-O-H angle: {angle_HOH:.1f} degrees")
# Should be ~104.5 degrees
```

The `np.clip` call prevents floating-point values slightly outside [-1, 1] from causing `arccos` to return NaN. This happens more often than you would expect when two vectors are nearly parallel or antiparallel.

### Center of mass and moment of inertia

The center of mass is a mass-weighted average of coordinates. NumPy's `np.average` handles this directly:

```python
masses = np.array([15.999, 1.008, 1.008])  # atomic masses in amu

com = np.average(coords, weights=masses, axis=0)
print(f"Center of mass: {com}")
```

The moment of inertia tensor is a 3x3 matrix that determines rotational properties. Its eigenvalues are the principal moments, and its eigenvectors are the principal axes:

```python
def inertia_tensor(coords, masses):
    """Compute the 3x3 moment of inertia tensor."""
    coords_com = coords - np.average(coords, weights=masses, axis=0)
    I = np.zeros((3, 3))
    for atom_idx in range(len(masses)):
        m = masses[atom_idx]
        r = coords_com[atom_idx]
        I += m * (np.dot(r, r) * np.eye(3) - np.outer(r, r))
    return I

I = inertia_tensor(coords, masses)
principal_moments, principal_axes = np.linalg.eigh(I)
print("Principal moments of inertia (amu*A^2):")
for val in principal_moments:
    print(f"  {val:.4f}")
```

Water is an asymmetric top: all three principal moments are different. A linear molecule (like CO2) would have one moment near zero and two equal moments. A spherical top (like methane) would have three equal moments. Checking these relationships is a quick way to classify molecular symmetry from coordinates alone.

### Unit conversions

Quantum chemistry codes typically work in atomic units (Bohr for length, Hartree for energy). The conversions come up constantly:

```python
# Conversion factors
ANGSTROM_TO_BOHR = 1.8897259886
BOHR_TO_ANGSTROM = 1.0 / ANGSTROM_TO_BOHR
HARTREE_TO_EV = 27.211386245988
HARTREE_TO_KCALMOL = 627.5094740631

coords_bohr = coords * ANGSTROM_TO_BOHR
print(f"O-H distance in Bohr: {np.linalg.norm(coords_bohr[1] - coords_bohr[0]):.4f}")
```

## Part 2: Gaussian Basis Functions

Quantum chemistry represents molecular orbitals as linear combinations of basis functions. Almost all modern basis sets use Gaussian-type functions because their products have closed-form integral expressions.

### Primitive Gaussian functions

A primitive Gaussian centered at position A with exponent alpha and angular momentum (l, m, n) is:

```
g(r; alpha, A, l, m, n) = N * (x-Ax)^l * (y-Ay)^m * (z-Az)^n * exp(-alpha * |r-A|^2)
```

For s-type functions (l=m=n=0), this simplifies to a spherical Gaussian. The normalization constant N ensures the function integrates to 1 when squared:

```python
def gaussian_norm(alpha, l, m, n):
    """Normalization constant for a Cartesian Gaussian primitive."""
    from scipy.special import factorial2
    L = l + m + n
    prefactor = (2 * alpha / np.pi) ** 0.75
    angular = (4 * alpha) ** (L / 2.0)
    denom = np.sqrt(factorial2(2*l - 1, exact=True) *
                    factorial2(2*m - 1, exact=True) *
                    factorial2(2*n - 1, exact=True))
    return prefactor * angular / denom if denom > 0 else prefactor * angular
```

`factorial2` is the double factorial from SciPy: `factorial2(5) = 5*3*1 = 15`. By convention, `factorial2(-1) = 1`.

### Contracted Gaussians

Production basis sets use contracted Gaussian functions: fixed linear combinations of primitives. Each contraction has a set of exponents and coefficients. For example, STO-3G represents each Slater-type orbital as a contraction of 3 Gaussians.

The STO-3G basis for hydrogen is:

```python
# STO-3G basis for hydrogen 1s
# Each tuple is (exponent, coefficient)
H_1s_STO3G = [
    (3.42525091, 0.15432897),
    (0.62353014, 0.53532814),
    (0.16885540, 0.44463454),
]
```

These numbers come from a least-squares fit of three Gaussians to a Slater function with exponent zeta=1.24.

### The overlap integral

The overlap integral between two s-type Gaussian primitives centered at A and B is one of the simplest integrals in quantum chemistry. For primitives with exponents alpha and beta:

```python
def overlap_1d(alpha, beta, Ax, Bx, l1, l2):
    """
    One-dimensional overlap integral between two Gaussian primitives.

    For s-type (l1=l2=0), this reduces to the standard Gaussian product formula.
    Higher angular momentum uses the Obara-Saika recurrence.
    """
    gamma = alpha + beta
    Px = (alpha * Ax + beta * Bx) / gamma  # center of product Gaussian

    if l1 == 0 and l2 == 0:
        return np.sqrt(np.pi / gamma) * np.exp(-alpha * beta / gamma * (Ax - Bx)**2)

    # Obara-Saika recurrence for higher angular momentum
    # S(l1, l2) = (Px - Ax) * S(l1-1, l2) + 1/(2*gamma) * [(l1-1)*S(l1-2, l2) + l2*S(l1-1, l2-1)]
    max_l = l1 + l2 + 1
    S = np.zeros((max_l + 1, max_l + 1))
    S[0, 0] = np.sqrt(np.pi / gamma) * np.exp(-alpha * beta / gamma * (Ax - Bx)**2)

    # Build up in first index
    for i in range(1, l1 + l2 + 1):
        S[i, 0] = (Px - Ax) * S[i-1, 0]
        if i >= 2:
            S[i, 0] += (i - 1) / (2 * gamma) * S[i-2, 0]

    # Build up in second index using horizontal recurrence
    for j in range(1, l2 + 1):
        for i in range(l1 + 1):
            S[i, j] = S[i+1, j-1] + (Ax - Bx) * S[i, j-1]

    return S[l1, l2]
```

The Obara-Saika recurrence is the standard technique. It builds overlap integrals for arbitrary angular momentum from the base case (two s-type Gaussians), using a recurrence relation that adds one unit of angular momentum at a time. This avoids computing factorials and binomial coefficients explicitly.

### Building the overlap matrix for H2

With the overlap integral function in hand, we can build the full overlap matrix for a molecule. Here is H2 in the STO-3G basis:

```python
def overlap_3d(alpha, beta, A, B, l1, m1, n1, l2, m2, n2):
    """Three-dimensional overlap integral (product of 1D integrals)."""
    Sx = overlap_1d(alpha, beta, A[0], B[0], l1, l2)
    Sy = overlap_1d(alpha, beta, A[1], B[1], m1, m2)
    Sz = overlap_1d(alpha, beta, A[2], B[2], n1, n2)
    return Sx * Sy * Sz

def build_overlap_matrix_H2(basis_A, basis_B, center_A, center_B):
    """
    Build the 2x2 overlap matrix for H2 in a minimal basis.

    basis_A, basis_B: lists of (exponent, coefficient) tuples
    center_A, center_B: coordinate arrays of shape (3,)
    """
    S = np.zeros((2, 2))

    for i, (basis_i, center_i) in enumerate([(basis_A, center_A), (basis_B, center_B)]):
        for j, (basis_j, center_j) in enumerate([(basis_A, center_A), (basis_B, center_B)]):
            val = 0.0
            for alpha, ca in basis_i:
                Na = gaussian_norm(alpha, 0, 0, 0)
                for beta, cb in basis_j:
                    Nb = gaussian_norm(beta, 0, 0, 0)
                    val += Na * Nb * ca * cb * overlap_3d(
                        alpha, beta, center_i, center_j,
                        0, 0, 0, 0, 0, 0
                    )
            S[i, j] = val

    return S

# H2 molecule: two hydrogens along z-axis, 0.74 Angstroms apart
R = 0.74 * ANGSTROM_TO_BOHR  # convert to Bohr
center_A = np.array([0.0, 0.0, -R/2])
center_B = np.array([0.0, 0.0,  R/2])

S = build_overlap_matrix_H2(H_1s_STO3G, H_1s_STO3G, center_A, center_B)
print("Overlap matrix for H2/STO-3G:")
print(np.array2string(S, precision=6))
```

The diagonal elements should be 1.0 (each function overlaps perfectly with itself, by normalization). The off-diagonal elements (around 0.66 for H2 at equilibrium) measure how much the two basis functions overlap. A value of 0 means the functions are orthogonal; a value close to 1 means they are nearly linearly dependent.

## Part 3: One-Electron Integrals

The core Hamiltonian H = T + V contains the kinetic energy and nuclear attraction integrals. These are the one-electron integrals that define the problem before we consider electron-electron repulsion.

### Kinetic energy integrals

The kinetic energy integral measures the curvature of a basis function (the "kinetic energy" of an electron described by that function). For s-type Gaussians, the Obara-Saika recurrence for the kinetic integral uses the overlap integrals as building blocks:

```python
def kinetic_1d(alpha, beta, Ax, Bx, l1, l2):
    """One-dimensional kinetic energy integral."""
    # T = beta * (2*(l2+1)*S(l1, l2) - l2*(l2-1)*S(l1, l2-2)) - 2*beta^2 * S(l1, l2+2)
    # This is derived from the second derivative of the Gaussian
    term1 = beta * (2 * l2 + 1) * overlap_1d(alpha, beta, Ax, Bx, l1, l2)
    term2 = -2 * beta**2 * overlap_1d(alpha, beta, Ax, Bx, l1, l2 + 2)
    term3 = -0.5 * l2 * (l2 - 1) * overlap_1d(alpha, beta, Ax, Bx, l1, l2 - 2) if l2 >= 2 else 0.0
    return 0.5 * (term1 + term2 + term3)

def kinetic_3d(alpha, beta, A, B, l1, m1, n1, l2, m2, n2):
    """Three-dimensional kinetic energy integral."""
    Tx = kinetic_1d(alpha, beta, A[0], B[0], l1, l2) * \
         overlap_1d(alpha, beta, A[1], B[1], m1, m2) * \
         overlap_1d(alpha, beta, A[2], B[2], n1, n2)
    Ty = overlap_1d(alpha, beta, A[0], B[0], l1, l2) * \
         kinetic_1d(alpha, beta, A[1], B[1], m1, m2) * \
         overlap_1d(alpha, beta, A[2], B[2], n1, n2)
    Tz = overlap_1d(alpha, beta, A[0], B[0], l1, l2) * \
         overlap_1d(alpha, beta, A[1], B[1], m1, m2) * \
         kinetic_1d(alpha, beta, A[2], B[2], n1, n2)
    return Tx + Ty + Tz
```

### Nuclear attraction integrals

Nuclear attraction integrals are more involved because they contain a 1/r_C term (the Coulomb attraction to a nucleus at position C). The angular integration can be done analytically, but the radial part introduces the Boys function F_n(x):

```python
from scipy.special import hyp1f1

def boys_function(n, x):
    """
    Boys function F_n(x) using the confluent hypergeometric function.

    F_n(x) = (1/(2n+1)) * 1F1(n + 1/2; n + 3/2; -x)
    """
    if x < 1e-12:
        return 1.0 / (2 * n + 1)
    return hyp1f1(n + 0.5, n + 1.5, -x) / (2 * n + 1)
```

The Boys function appears whenever you integrate a Gaussian times a Coulomb potential. SciPy's `hyp1f1` (confluent hypergeometric function of the first kind, also called Kummer's function or M(a, b, z)) computes it reliably, including the small-x limit where the series needs care.

The full nuclear attraction integral for s-type functions is:

```python
def nuclear_attraction_ss(alpha, beta, A, B, C, Z):
    """
    Nuclear attraction integral for two s-type Gaussians
    with a point charge Z at position C.
    """
    gamma = alpha + beta
    P = (alpha * A + beta * B) / gamma
    AB2 = np.dot(A - B, A - B)
    PC2 = np.dot(P - C, P - C)

    prefactor = -Z * 2 * np.pi / gamma
    exponential = np.exp(-alpha * beta / gamma * AB2)

    return prefactor * exponential * boys_function(0, gamma * PC2)
```

## Part 4: A Minimal Hartree-Fock Program

With the integral machinery in place, we can write a complete SCF program for H2/STO-3G. This is the simplest non-trivial Hartree-Fock calculation: two basis functions, two electrons, and a 2x2 Fock matrix.

### Two-electron integrals for H2

For a minimal basis with only s-type functions, the two-electron integral (ab|cd) between four s-type Gaussians has a closed-form expression using the Boys function:

```python
def electron_repulsion_ssss(alpha, beta, gamma_c, delta, A, B, C, D):
    """
    Two-electron repulsion integral (AB|CD) for four s-type Gaussians.
    """
    p = alpha + beta
    q = gamma_c + delta
    P = (alpha * A + beta * B) / p
    Q = (gamma_c * C + delta * D) / q
    PQ2 = np.dot(P - Q, P - Q)
    AB2 = np.dot(A - B, A - B)
    CD2 = np.dot(C - D, C - D)

    prefactor = 2 * np.pi**2.5 / (p * q * np.sqrt(p + q))
    exponential = np.exp(-alpha * beta / p * AB2 - gamma_c * delta / q * CD2)

    return prefactor * exponential * boys_function(0, p * q / (p + q) * PQ2)
```

Building the full set of two-electron integrals for a 2-function basis requires evaluating (ab|cd) for all combinations of a, b, c, d in {0, 1}. With 8-fold permutation symmetry, there are only a handful of unique integrals:

```python
def build_eri_tensor(basis_functions, centers):
    """
    Build the two-electron integral tensor for a minimal basis.

    basis_functions: list of [(exponent, coefficient), ...] for each basis function
    centers: list of coordinate arrays for each basis function
    """
    nbf = len(basis_functions)
    eri = np.zeros((nbf, nbf, nbf, nbf))

    for a in range(nbf):
        for b in range(nbf):
            for c in range(nbf):
                for d in range(nbf):
                    val = 0.0
                    for alpha, ca in basis_functions[a]:
                        Na = gaussian_norm(alpha, 0, 0, 0)
                        for beta, cb in basis_functions[b]:
                            Nb = gaussian_norm(beta, 0, 0, 0)
                            for gamma_c, cc in basis_functions[c]:
                                Nc = gaussian_norm(gamma_c, 0, 0, 0)
                                for delta, cd in basis_functions[d]:
                                    Nd = gaussian_norm(delta, 0, 0, 0)
                                    val += (Na * ca * Nb * cb * Nc * cc * Nd * cd *
                                            electron_repulsion_ssss(
                                                alpha, beta, gamma_c, delta,
                                                centers[a], centers[b],
                                                centers[c], centers[d]))
                    eri[a, b, c, d] = val
    return eri
```

For a 2-function basis this loop runs 2^4 * 3^4 = 1,296 primitive integral evaluations (16 index combinations times 81 primitive quartets). For larger basis sets this brute-force approach is impractical, which is why production codes use screening, batching, and compiled kernels (see the [libaccint tutorial](/tutorials/libaccint-tutorial/) for that approach).

### The SCF procedure

The restricted Hartree-Fock SCF algorithm:

1. Build the overlap S, kinetic T, and nuclear attraction V matrices
2. Form the core Hamiltonian: H_core = T + V
3. Compute the orthogonalizer X = S^(-1/2)
4. Make an initial guess for the density matrix D (from H_core eigenvectors)
5. Loop until convergence:
   a. Build the Fock matrix: F = H_core + G(D), where G is the two-electron contribution
   b. Transform to orthogonal basis: F' = X^T F X
   c. Diagonalize F' to get orbital energies and coefficients
   d. Back-transform: C = X C'
   e. Build new density: D = 2 C_occ C_occ^T
   f. Check energy change and density change

```python
from scipy.linalg import eigh

def rhf_scf(S, T, V, eri, n_occ, nuclear_repulsion, max_iter=50, e_thresh=1e-10):
    """
    Restricted Hartree-Fock SCF calculation.

    S: overlap matrix (nbf, nbf)
    T: kinetic energy matrix (nbf, nbf)
    V: nuclear attraction matrix (nbf, nbf)
    eri: two-electron integral tensor (nbf, nbf, nbf, nbf)
    n_occ: number of occupied orbitals
    nuclear_repulsion: nuclear repulsion energy
    """
    nbf = S.shape[0]
    H_core = T + V

    # Orthogonalizer: X = S^(-1/2)
    s_vals, U = eigh(S)
    s_inv_sqrt = np.zeros_like(s_vals)
    s_inv_sqrt[s_vals > 1e-10] = 1.0 / np.sqrt(s_vals[s_vals > 1e-10])
    X = U @ np.diag(s_inv_sqrt) @ U.T

    # Initial guess: diagonalize H_core in orthogonal basis
    H_prime = X.T @ H_core @ X
    eps, C_prime = eigh(H_prime)
    C = X @ C_prime
    D = 2.0 * C[:, :n_occ] @ C[:, :n_occ].T

    E_old = 0.0
    converged = False

    print(f"{'Iter':>4s}  {'E_total':>18s}  {'dE':>12s}  {'dD':>12s}")
    print("-" * 52)

    for iteration in range(max_iter):
        # Build Fock matrix: F = H_core + G
        # G_ab = sum_cd D_cd * [(ab|cd) - 0.5 * (ac|bd)]
        J = np.einsum('abcd,cd->ab', eri, D)
        K = np.einsum('acbd,cd->ab', eri, D)
        F = H_core + J - 0.5 * K

        # Electronic energy
        E_elec = 0.5 * np.sum((H_core + F) * D)
        E_total = E_elec + nuclear_repulsion

        # Diagonalize in orthogonal basis
        F_prime = X.T @ F @ X
        eps, C_prime = eigh(F_prime)
        C = X @ C_prime
        D_new = 2.0 * C[:, :n_occ] @ C[:, :n_occ].T

        # Convergence check
        dE = abs(E_total - E_old)
        dD = np.max(np.abs(D_new - D))
        print(f"{iteration:4d}  {E_total:18.12f}  {dE:12.2e}  {dD:12.2e}")

        if dE < e_thresh and dD < 1e-8 and iteration > 0:
            converged = True
            break

        D = D_new
        E_old = E_total

    if converged:
        print(f"\nConverged in {iteration + 1} iterations")
    else:
        print(f"\nWARNING: not converged after {max_iter} iterations")

    print(f"RHF energy: {E_total:.10f} Hartree")
    print(f"Orbital energies: {eps}")
    return E_total, eps, C
```

### Running the calculation

Putting it all together for H2/STO-3G:

```python
# Build integral matrices (using the functions defined above)
basis = [H_1s_STO3G, H_1s_STO3G]
centers = [center_A, center_B]

# One-electron integrals
S = build_overlap_matrix_H2(H_1s_STO3G, H_1s_STO3G, center_A, center_B)
T = np.zeros((2, 2))
V = np.zeros((2, 2))

# Build T and V using the same contraction loop pattern as S
for i, (basis_i, ci) in enumerate(zip(basis, centers)):
    for j, (basis_j, cj) in enumerate(zip(basis, centers)):
        t_val, v_val = 0.0, 0.0
        for alpha, ca in basis_i:
            Na = gaussian_norm(alpha, 0, 0, 0)
            for beta, cb in basis_j:
                Nb = gaussian_norm(beta, 0, 0, 0)
                prim = Na * ca * Nb * cb
                t_val += prim * kinetic_3d(alpha, beta, ci, cj, 0,0,0, 0,0,0)
                for k, (center_k, Z_k) in enumerate(zip(centers, [1.0, 1.0])):
                    v_val += prim * nuclear_attraction_ss(alpha, beta, ci, cj, center_k, Z_k)
        T[i, j] = t_val
        V[i, j] = v_val

# Two-electron integrals
eri = build_eri_tensor(basis, centers)

# Nuclear repulsion
R_AB = np.linalg.norm(center_A - center_B)
E_nuc = 1.0 / R_AB  # Z_A * Z_B / R_AB, both Z=1

# Run SCF
E, orbital_energies, C = rhf_scf(S, T, V, eri, n_occ=1, nuclear_repulsion=E_nuc)
```

The expected RHF energy for H2/STO-3G at the equilibrium bond length (0.74 A) is approximately -1.1175 Hartree. This is not a great result compared to the exact value of -1.1745 Hartree (full CI in a complete basis), but it confirms that the integral code and SCF implementation are working correctly.

## Part 5: Potential Energy Surfaces with SciPy

One of the most informative calculations you can do is scan the potential energy surface (PES) as a function of a geometric coordinate. For H2, this means computing the energy at a range of bond lengths.

### Bond length scan

```python
distances_angstrom = np.linspace(0.3, 4.0, 40)
energies = []

for R_ang in distances_angstrom:
    R_bohr = R_ang * ANGSTROM_TO_BOHR
    cA = np.array([0.0, 0.0, -R_bohr/2])
    cB = np.array([0.0, 0.0,  R_bohr/2])

    S_scan = build_overlap_matrix_H2(H_1s_STO3G, H_1s_STO3G, cA, cB)

    T_scan = np.zeros((2, 2))
    V_scan = np.zeros((2, 2))
    basis_scan = [H_1s_STO3G, H_1s_STO3G]
    centers_scan = [cA, cB]

    for i, (bi, ci) in enumerate(zip(basis_scan, centers_scan)):
        for j, (bj, cj) in enumerate(zip(basis_scan, centers_scan)):
            t_val, v_val = 0.0, 0.0
            for alpha, ca in bi:
                Na = gaussian_norm(alpha, 0, 0, 0)
                for beta, cb in bj:
                    Nb = gaussian_norm(beta, 0, 0, 0)
                    prim = Na * ca * Nb * cb
                    t_val += prim * kinetic_3d(alpha, beta, ci, cj, 0,0,0, 0,0,0)
                    for k, ck in enumerate(centers_scan):
                        v_val += prim * nuclear_attraction_ss(alpha, beta, ci, cj, ck, 1.0)
            T_scan[i, j] = t_val
            V_scan[i, j] = v_val

    eri_scan = build_eri_tensor(basis_scan, centers_scan)
    E_nuc_scan = 1.0 / R_bohr

    E_scan, _, _ = rhf_scf(S_scan, T_scan, V_scan, eri_scan, 1, E_nuc_scan, max_iter=50)
    energies.append(E_scan)
```

### Fitting a Morse potential

The computed PES can be fit to a Morse potential, which is the standard analytical model for a diatomic molecule:

```python
from scipy.optimize import curve_fit

def morse(r, De, a, re, E0):
    """Morse potential: V(r) = De * (1 - exp(-a*(r-re)))^2 + E0"""
    return De * (1 - np.exp(-a * (r - re)))**2 + E0

# Fit the Morse function to the computed energies
popt, pcov = curve_fit(
    morse, distances_angstrom, energies,
    p0=[0.17, 1.5, 0.75, -1.12],   # initial guess: De, a, re, E0
    maxfev=10000,
)
De, a, re, E0 = popt
print(f"Morse fit parameters:")
print(f"  De = {De:.4f} Hartree ({De * HARTREE_TO_KCALMOL:.1f} kcal/mol)")
print(f"  a  = {a:.4f} 1/Angstrom")
print(f"  re = {re:.4f} Angstrom")
```

### Visualization

```python
import matplotlib.pyplot as plt

r_fine = np.linspace(0.3, 4.0, 200)
E_fit = morse(r_fine, *popt)

fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(distances_angstrom, energies, 'o', markersize=5, label='RHF/STO-3G', color='#2563eb')
ax.plot(r_fine, E_fit, '-', linewidth=1.5, label='Morse fit', color='#dc2626')
ax.axhline(y=E0 + De, color='gray', linestyle='--', alpha=0.5, label='Dissociation limit')
ax.set_xlabel('R (Angstrom)', fontsize=12)
ax.set_ylabel('Energy (Hartree)', fontsize=12)
ax.set_title('H$_2$ Potential Energy Surface (RHF/STO-3G)', fontsize=14)
ax.legend(fontsize=11)
ax.set_xlim(0.3, 4.0)
plt.tight_layout()
plt.savefig('h2_pes.png', dpi=150)
plt.show()
```

### Numerical Hessian and vibrational frequency

The harmonic vibrational frequency can be extracted from the second derivative of the PES at the equilibrium geometry. SciPy's `approx_fprime` can compute numerical derivatives when analytical ones are not available:

```python
from scipy.misc import derivative

def energy_at_R(R_ang):
    """Compute the RHF energy at a given bond length. (Wrapper for the scan above.)"""
    # In practice, interpolate from the scan data or recompute
    from scipy.interpolate import interp1d
    interp = interp1d(distances_angstrom, energies, kind='cubic')
    return float(interp(R_ang))

# Second derivative at equilibrium (force constant)
k = derivative(energy_at_R, re, n=2, dx=0.01)

# Convert to vibrational frequency
# k is in Hartree/Angstrom^2, convert to SI
k_SI = k * 4.3597447222071e-18 / (1e-10)**2   # J/m^2
mu_H2 = 0.5 * 1.008 * 1.66053906660e-27       # reduced mass in kg
nu = 1 / (2 * np.pi) * np.sqrt(k_SI / mu_H2)  # frequency in Hz
nu_cm = nu / (2.998e10)                         # convert to cm^-1

print(f"Force constant: {k:.4f} Hartree/Angstrom^2")
print(f"Harmonic frequency: {nu_cm:.0f} cm^-1")
print(f"Experimental value: 4401 cm^-1")
```

RHF/STO-3G will overestimate the vibrational frequency because the minimal basis produces a PES that is too steep. This is expected; improving the basis set and adding electron correlation (MP2 or CCSD) brings the frequency closer to the experimental value.

## Part 6: Convergence Analysis and Visualization

Plotting the SCF convergence is helpful for diagnosing problems and comparing convergence accelerators (simple mixing, DIIS, level shifting).

```python
def rhf_scf_with_history(S, T, V, eri, n_occ, E_nuc, max_iter=50):
    """RHF SCF that returns energy and error history for plotting."""
    nbf = S.shape[0]
    H_core = T + V

    s_vals, U = eigh(S)
    s_inv_sqrt = np.zeros_like(s_vals)
    s_inv_sqrt[s_vals > 1e-10] = 1.0 / np.sqrt(s_vals[s_vals > 1e-10])
    X = U @ np.diag(s_inv_sqrt) @ U.T

    H_prime = X.T @ H_core @ X
    eps, C_prime = eigh(H_prime)
    C = X @ C_prime
    D = 2.0 * C[:, :n_occ] @ C[:, :n_occ].T

    E_old = 0.0
    energy_history = []
    error_history = []

    for iteration in range(max_iter):
        J = np.einsum('abcd,cd->ab', eri, D)
        K = np.einsum('acbd,cd->ab', eri, D)
        F = H_core + J - 0.5 * K

        E_elec = 0.5 * np.sum((H_core + F) * D)
        E_total = E_elec + E_nuc
        energy_history.append(E_total)

        F_prime = X.T @ F @ X
        eps, C_prime = eigh(F_prime)
        C = X @ C_prime
        D_new = 2.0 * C[:, :n_occ] @ C[:, :n_occ].T

        dE = abs(E_total - E_old)
        error_history.append(dE if iteration > 0 else 1.0)

        if dE < 1e-10 and iteration > 0:
            break

        D = D_new
        E_old = E_total

    return energy_history, error_history

# Plot convergence
energy_hist, error_hist = rhf_scf_with_history(S, T, V, eri, 1, E_nuc)

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4.5))

ax1.plot(range(len(energy_hist)), energy_hist, 'o-', markersize=4, color='#2563eb')
ax1.set_xlabel('Iteration', fontsize=12)
ax1.set_ylabel('Total Energy (Hartree)', fontsize=12)
ax1.set_title('SCF Energy Convergence', fontsize=13)

ax2.semilogy(range(1, len(error_hist)), error_hist[1:], 'o-', markersize=4, color='#dc2626')
ax2.set_xlabel('Iteration', fontsize=12)
ax2.set_ylabel('|dE| (Hartree)', fontsize=12)
ax2.set_title('SCF Energy Change', fontsize=13)

plt.tight_layout()
plt.savefig('scf_convergence.png', dpi=150)
plt.show()
```

For H2/STO-3G, the SCF converges in about 7-8 iterations without any acceleration. Larger molecules in bigger basis sets can take 20-50 iterations, which is where DIIS (Direct Inversion of the Iterative Subspace) becomes important. The [libaccint tutorial](/tutorials/libaccint-tutorial/) includes a DIIS implementation if you want to see how it works.

## Exercises

1. **Extend to HeH+.** Modify the H2 code to compute the RHF energy of HeH+ (helium has Z=2, hydrogen has Z=1, the ion has 2 electrons). You will need the STO-3G basis for helium: exponents (6.36242139, 1.15892300, 0.31364979) with coefficients (0.15432897, 0.53532814, 0.44463454). Check your answer against a GAMESS calculation.

2. **Basis set dependence.** Rerun the H2 calculation with different STO-nG basis sets (n=2 through 6; the exponents and coefficients are tabulated in Szabo and Ostlund, Appendix A). Plot the equilibrium energy as a function of n and observe the convergence.

3. **Water geometry.** Using the NumPy geometry tools from Part 1, write a function that takes an O-H distance and H-O-H angle and returns the Cartesian coordinates of water. Use `scipy.optimize.minimize` to find the RHF/STO-3G equilibrium geometry by optimizing these two parameters.

4. **Numerical gradients.** Compute the energy gradient of H2 with respect to the bond length using a finite-difference formula. Compare the 2-point central difference `(E(R+h) - E(R-h)) / (2h)` against the 4-point formula for different step sizes h. At what step size does the 2-point formula break down due to floating-point cancellation?

## Reference: NumPy and SciPy Functions Used

| Function | Module | Purpose in this tutorial |
|----------|--------|------------------------|
| `np.array` | NumPy | Coordinate and matrix storage |
| `np.dot` | NumPy | Vector dot products, matrix multiplication |
| `np.linalg.norm` | NumPy | Vector and matrix norms |
| `np.linalg.eigh` | NumPy | Symmetric eigenvalue decomposition |
| `np.einsum` | NumPy | Tensor contractions (Fock matrix build) |
| `np.average` | NumPy | Weighted averages (center of mass) |
| `np.outer` | NumPy | Outer products (inertia tensor, density matrix) |
| `np.clip` | NumPy | Clamp values to a range (numerical safety) |
| `scipy.linalg.eigh` | SciPy | Generalized symmetric eigenvalue problems |
| `scipy.special.hyp1f1` | SciPy | Confluent hypergeometric function (Boys function) |
| `scipy.special.factorial2` | SciPy | Double factorial (normalization constants) |
| `scipy.optimize.curve_fit` | SciPy | Nonlinear least-squares fitting (Morse potential) |
| `scipy.optimize.minimize` | SciPy | Geometry optimization |
| `scipy.misc.derivative` | SciPy | Numerical derivatives (force constants) |
| `scipy.interpolate.interp1d` | SciPy | Cubic interpolation of PES data |

## Next Steps

This tutorial implemented everything from scratch to show what is happening inside a quantum chemistry calculation. For production work, you would not write your own integral code in Python. Libraries like [PySCF](https://pyscf.org/), [Psi4](https://psicode.org/) (via its Python API), and [libaccint](/tutorials/libaccint-tutorial/) provide compiled integral engines that are orders of magnitude faster. But understanding the machinery at this level makes it much easier to use those tools effectively, debug problems when results look wrong, and extend them when you need something they do not provide out of the box.

For the C++ and Python approach to building a full Hartree-Fock program using a compiled integral library, see the [libaccint tutorial](/tutorials/libaccint-tutorial/). For running calculations on real molecules with GAMESS, see [Getting Started with GAMESS](/tutorials/getting-started-with-gamess/).

---

*Questions about this tutorial? [Get in touch](/contact/) or check out my other [tutorials](/tutorials/).*
