---
title: "Getting Started with GAMESS"
date: 2025-12-05
description: "A beginner's guide to running quantum chemistry calculations with GAMESS"
level: "beginner"
tags: ["GAMESS", "quantum chemistry", "tutorial"]
---

GAMESS (General Atomic and Molecular Electronic Structure System) is a powerful program for ab initio quantum chemistry. This tutorial will help you get started with your first calculations.

## What is GAMESS?

GAMESS is a general-purpose quantum chemistry software package capable of:

- Hartree-Fock calculations (RHF, UHF, ROHF)
- Density Functional Theory (DFT)
- Post-HF methods (MP2, CI, coupled cluster)
- Geometry optimizations
- Molecular dynamics
- Fragment molecular orbital (FMO) calculations

## Installation

GAMESS is available from the [Gordon Group at Iowa State](https://www.msg.chem.iastate.edu/gamess/). You'll need to:

1. Request a license (free for academics)
2. Download the source code
3. Compile for your system

## Your First Calculation

Let's run a simple Hartree-Fock calculation on water (H₂O).

### Input File (water.inp)

```
 $CONTRL SCFTYP=RHF RUNTYP=ENERGY $END
 $BASIS GBASIS=STO NGAUSS=3 $END
 $DATA
Water STO-3G
C1
O     8.0  0.0000000  0.0000000  0.0000000
H     1.0  0.7569500  0.5858500  0.0000000
H     1.0 -0.7569500  0.5858500  0.0000000
 $END
```

### Running the Calculation

```bash
./rungms water.inp 00 1 > water.log
```

## Understanding the Output

Key sections to look for:

- **TOTAL ENERGY**: The final electronic energy
- **ORBITAL ENERGIES**: MO energy levels
- **MULLIKEN POPULATION ANALYSIS**: Atomic charges
- **GRADIENT INFORMATION**: For optimization runs

## Next Steps

- Learn about basis sets
- Explore geometry optimization
- Try DFT calculations
- Study excited states

---

*Questions? Check out the [GAMESS manual](https://www.msg.chem.iastate.edu/gamess/documentation.html) or [contact me](/contact/).*
