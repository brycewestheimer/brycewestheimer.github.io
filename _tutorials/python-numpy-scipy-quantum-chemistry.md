---
title: "Python for Quantum Chemistry: NumPy and SciPy Fundamentals"
date: 2024-07-09
difficulty: "intermediate"
duration: "60 min"
topics: [python, numpy, scipy, computational-chemistry]
description: "Learn how to use Python's scientific computing libraries for quantum chemistry calculations and data analysis"
prerequisites: ["Python basics", "Linear algebra concepts", "Basic quantum chemistry"]
software: ["Python 3.8+", "NumPy", "SciPy", "Matplotlib", "Jupyter Notebook"]
---

# Python for Quantum Chemistry: NumPy and SciPy Fundamentals

Python has become the lingua franca of scientific computing, and quantum chemistry is no exception. This tutorial will teach you how to leverage NumPy and SciPy for quantum chemistry calculations and data analysis.

## Learning Objectives

After completing this tutorial, you will be able to:
- Manipulate molecular data using NumPy arrays
- Perform matrix operations relevant to quantum chemistry
- Use SciPy for optimization and linear algebra tasks
- Implement basic quantum chemistry algorithms
- Visualize molecular properties and calculation results

## Prerequisites Check

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

### Distance Matrices

```python
def distance_matrix(coords):
    """Calculate distance matrix for a set of coordinates."""
    n_atoms = len(coords)
    distances = np.zeros((n_atoms, n_atoms))
    
    for i in range(n_atoms):
        for j in range(n_atoms):
            distances[i, j] = np.linalg.norm(coords[i] - coords[j])
    
    return distances

# More efficient vectorized version
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
    # STO-1G exponents for hydrogen
    alpha_h = 1.24
    
    # H2 bond length (Å)
    R = 0.74
    
    # Positions
    xa, xb = -R/2, R/2
    
    # Overlap matrix
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
    """Morse potential for H2 molecule (parameters in eV and Å)."""
    return De * (1 - np.exp(-a * (r - re)))**2

def optimize_bond_length():
    """Find equilibrium bond length using SciPy optimization."""
    
    # Initial guess
    r0 = 1.0
    
    # Minimize the potential
    result = minimize(morse_potential, r0, method='BFGS')
    
    return result

# Optimize geometry
result = optimize_bond_length()
print(f"Optimized bond length: {result.x[0]:.3f} Å")
print(f"Minimum energy: {result.fun:.3f} eV")

# Plot potential energy surface
r_values = np.linspace(0.5, 2.0, 100)
energies = morse_potential(r_values)

plt.figure(figsize=(10, 6))
plt.plot(r_values, energies, 'b-', linewidth=2, label='Morse potential')
plt.axvline(result.x[0], color='r', linestyle='--', label=f'Optimized: {result.x[0]:.3f} Å')
plt.xlabel('Bond length (Å)')
plt.ylabel('Energy (eV)')
plt.title('H₂ Potential Energy Surface')
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()
```

## Part 4: Eigenvalue Problems

### Solving the Secular Equation

```python
from scipy.linalg import eigh

def solve_huckel_system():
    """Solve Hückel theory for ethylene (C2H4)."""
    
    # Hückel matrix (α on diagonal, β on off-diagonal)
    # Using α = 0, β = -1 (in units of β)
    H = np.array([
        [0, -1],
        [-1, 0]
    ])
    
    # Solve eigenvalue problem
    eigenvalues, eigenvectors = eigh(H)
    
    print("Hückel MO energies (in units of β):")
    for i, E in enumerate(eigenvalues):
        print(f"MO {i+1}: {E:.3f}")
    
    print("\nMO coefficients:")
    for i, vec in enumerate(eigenvectors.T):
        print(f"MO {i+1}: [{vec[0]:6.3f}, {vec[1]:6.3f}]")
    
    return eigenvalues, eigenvectors

eigenvals, eigenvecs = solve_huckel_system()
```

## Part 5: Data Analysis and Visualization

### Analyzing Calculation Results

```python
def analyze_trajectory():
    """Analyze molecular dynamics trajectory data."""
    
    # Simulated trajectory data (time, energy)
    time = np.linspace(0, 10, 1000)  # ps
    
    # Add some noise to simulate real MD data
    np.random.seed(42)
    potential_energy = -150 + 5 * np.sin(2 * np.pi * time / 3) + np.random.normal(0, 2, len(time))
    kinetic_energy = 75 + 10 * np.cos(2 * np.pi * time / 2.5) + np.random.normal(0, 1.5, len(time))
    total_energy = potential_energy + kinetic_energy
    
    # Calculate running averages
    window = 50
    pe_avg = np.convolve(potential_energy, np.ones(window)/window, mode='valid')
    ke_avg = np.convolve(kinetic_energy, np.ones(window)/window, mode='valid')
    te_avg = np.convolve(total_energy, np.ones(window)/window, mode='valid')
    time_avg = time[window-1:]
    
    # Plot results
    fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(12, 8))
    
    # Energy vs time
    ax1.plot(time, potential_energy, alpha=0.3, color='red', label='PE (instantaneous)')
    ax1.plot(time, kinetic_energy, alpha=0.3, color='blue', label='KE (instantaneous)')
    ax1.plot(time, total_energy, alpha=0.3, color='green', label='Total (instantaneous)')
    
    ax1.plot(time_avg, pe_avg, color='red', linewidth=2, label='PE (running avg)')
    ax1.plot(time_avg, ke_avg, color='blue', linewidth=2, label='KE (running avg)')
    ax1.plot(time_avg, te_avg, color='green', linewidth=2, label='Total (running avg)')
    
    ax1.set_xlabel('Time (ps)')
    ax1.set_ylabel('Energy (kcal/mol)')
    ax1.set_title('Molecular Dynamics Energy Analysis')
    ax1.legend()
    ax1.grid(True, alpha=0.3)
    
    # Energy distribution
    ax2.hist(total_energy, bins=50, alpha=0.7, color='green', label='Total Energy')
    ax2.axvline(np.mean(total_energy), color='red', linestyle='--', 
                label=f'Mean: {np.mean(total_energy):.1f} kcal/mol')
    ax2.set_xlabel('Energy (kcal/mol)')
    ax2.set_ylabel('Frequency')
    ax2.set_title('Energy Distribution')
    ax2.legend()
    ax2.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.show()
    
    # Calculate statistics
    print(f"Average total energy: {np.mean(total_energy):.2f} ± {np.std(total_energy):.2f} kcal/mol")
    print(f"Energy conservation (std dev): {np.std(total_energy):.2f} kcal/mol")

analyze_trajectory()
```

## Advanced Topics

### Custom Functions for Quantum Chemistry

```python
def boys_function(n, x):
    """Boys function for electron repulsion integrals."""
    from scipy.special import hyp1f1
    
    if x < 1e-6:
        return 1.0 / (2*n + 1)
    else:
        return 0.5 * (x**(-n-0.5)) * hyp1f1(n+0.5, n+1.5, -x) * np.sqrt(np.pi)

# Example usage
x_values = np.logspace(-3, 2, 100)
for n in range(3):
    boys_vals = [boys_function(n, x) for x in x_values]
    plt.loglog(x_values, boys_vals, label=f'F_{n}(x)')

plt.xlabel('x')
plt.ylabel('F_n(x)')
plt.title('Boys Functions')
plt.legend()
plt.grid(True)
plt.show()
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
- [Computational Chemistry with Python](https://github.com/computational-chemistry-python)
- [PySCF Documentation](https://pyscf.org/)
