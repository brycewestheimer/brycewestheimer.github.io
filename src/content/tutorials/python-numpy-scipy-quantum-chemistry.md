---
title: "Python for Quantum Chemistry: NumPy and SciPy Fundamentals"
date: 2025-11-07
description: "Learn how to use Python's scientific computing libraries for quantum chemistry calculations and data analysis"
level: "intermediate"
tags: ["Python", "NumPy", "SciPy", "computational-chemistry"]
---

Python has become the lingua franca of scientific computing, and quantum chemistry is no exception. This tutorial will teach you how to leverage NumPy and SciPy for quantum chemistry calculations and data analysis.

## Learning Objectives

After completing this tutorial, you will be able to:
- Manipulate molecular data using NumPy arrays
- Perform matrix operations relevant to quantum chemistry
- Use SciPy for optimization and linear algebra tasks
- Implement basic quantum chemistry algorithms
- Visualize molecular properties and calculation results

## Prerequisites

Make sure you have the following installed:

```bash
pip install numpy scipy matplotlib jupyter
```

## Part 1: NumPy for Molecular Data

### Working with Coordinates

```python
import numpy as np

# Define water molecule coordinates (Å)
coords = np.array([
    [0.000,  0.000,  0.119],  # O
    [0.000,  0.757, -0.477],  # H1
    [0.000, -0.757, -0.477]   # H2
])

# Calculate center of mass
masses = np.array([15.999, 1.008, 1.008])  # amu
com = np.average(coords, weights=masses, axis=0)
print(f"Center of mass: {com}")
```

### Vectorized Distance Matrices

```python
def distance_matrix_vectorized(coords):
    """Vectorized distance matrix calculation."""
    diff = coords[:, np.newaxis, :] - coords[np.newaxis, :, :]
    return np.sqrt(np.sum(diff**2, axis=2))

dist_matrix = distance_matrix_vectorized(coords)
print("Distance matrix (Å):")
print(dist_matrix)
```

## Part 2: Matrix Operations for Quantum Chemistry

### Overlap Matrix Construction

```python
def gaussian_overlap_1d(alpha1, alpha2, xa, xb):
    """1D Gaussian overlap integral."""
    p = alpha1 + alpha2
    mu = alpha1 * alpha2 / p
    xab = xa - xb
    
    prefactor = np.sqrt(np.pi / p)
    exponential = np.exp(-mu * xab**2)
    
    return prefactor * exponential

# Build overlap matrix for H2 molecule
def build_overlap_matrix():
    alpha_h = 1.24  # STO-1G exponent
    R = 0.74  # Bond length (Å)
    xa, xb = -R/2, R/2
    
    S = np.array([
        [1.0, gaussian_overlap_1d(alpha_h, alpha_h, xa, xb)],
        [gaussian_overlap_1d(alpha_h, alpha_h, xb, xa), 1.0]
    ])
    return S

S = build_overlap_matrix()
print("Overlap matrix:")
print(S)
```

## Part 3: SciPy for Optimization

### Geometry Optimization

```python
from scipy.optimize import minimize
import matplotlib.pyplot as plt

def morse_potential(r, De=4.52, a=1.44, re=0.74):
    """Morse potential for H2 molecule (eV and Å)."""
    return De * (1 - np.exp(-a * (r - re)))**2

def optimize_bond_length():
    r0 = 1.0  # Initial guess
    result = minimize(morse_potential, r0, method='BFGS')
    return result

result = optimize_bond_length()
print(f"Optimized bond length: {result.x[0]:.3f} Å")
print(f"Minimum energy: {result.fun:.3f} eV")
```

## Part 4: Eigenvalue Problems

### Solving the Secular Equation

```python
from scipy.linalg import eigh

def solve_huckel_system():
    """Solve Hückel theory for ethylene (C2H4)."""
    
    # Hückel matrix (α=0, β=-1 in units of β)
    H = np.array([
        [0, -1],
        [-1, 0]
    ])
    
    eigenvalues, eigenvectors = eigh(H)
    
    print("Hückel MO energies (in units of β):")
    for i, E in enumerate(eigenvalues):
        print(f"MO {i+1}: {E:.3f}")
    
    return eigenvalues, eigenvectors

eigenvals, eigenvecs = solve_huckel_system()
```

## Exercises

1. **Molecular Geometry**: Calculate all bond angles in a water molecule
2. **Matrix Diagonalization**: Solve the particle in a box problem using matrix methods
3. **Curve Fitting**: Fit a Morse potential to potential energy data
4. **Statistical Analysis**: Analyze convergence of a Monte Carlo calculation

## Next Steps

- Explore advanced SciPy functionality (sparse matrices, special functions)
- Learn about specialized quantum chemistry libraries (PySCF, OpenFermion)
- Practice with real molecular data from quantum chemistry calculations
- Implement more complex algorithms (SCF, CI, perturbation theory)

## Resources

- [NumPy Documentation](https://numpy.org/doc/)
- [SciPy Tutorial](https://docs.scipy.org/doc/scipy/tutorial/)
- [PySCF Documentation](https://pyscf.org/)

---

*Questions about this tutorial? [Get in touch](/contact/) or check out my other [tutorials](/tutorials/).*
