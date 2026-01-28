---
layout: page
title: "Projects"
description: "A showcase of my computational chemistry and software development projects"
permalink: /projects/
---

## Flagship Projects

### `libfrag` — Fragment-Based Method Sandbox
<div class="project-card featured">
  <div class="project-meta">
    <span class="project-status">Active development</span>
    <span class="project-tech">Python, C++, CMake</span>
  </div>

  A modular playground for experimenting with fragment-based *ab initio* algorithms, designed to prototype ideas before they land in production GAMESS releases. Current efforts focus on multi-layer adaptive partitioning (MAP) analysis utilities and accelerator-friendly data layouts.

  [GitHub](https://github.com/brycewestheimer/libfrag)
</div>

### `public_libaccefp` — Accelerated Effective Fragment Potential Preview
<div class="project-card">
  <div class="project-meta">
    <span class="project-status">Public preview</span>
    <span class="project-tech">C++, CUDA, Fortran</span>
  </div>

  An open slice of the Accelerated EFP library showcasing GPU-aware kernels developed through collaborations with Ames National Laboratory. The preview highlights data structures and API patterns that informed the EFP chapter in *Comprehensive Computational Chemistry* (2024).

  [GitHub](https://github.com/brycewestheimer/public_libaccefp)
</div>

### `public_libaccsapt` — Accelerated SAPT Components
<div class="project-card">
  <div class="project-meta">
    <span class="project-status">Preview</span>
    <span class="project-tech">C++, OpenMP, MPI</span>
  </div>

  A reference implementation of subsystem-local SAPT routines, capturing the exchange-repulsion ideas featured in my ACS Fall 2024 presentation. The repository documents parallelization strategies for heterogeneous CPU/GPU platforms.

  [GitHub](https://github.com/brycewestheimer/public_libaccsapt)
</div>

### `public_qccg` — Quantum Chemistry Code Generator
<div class="project-card">
  <div class="project-meta">
    <span class="project-status">In development</span>
    <span class="project-tech">C++, Python</span>
  </div>

  A template-driven code generator that produces tightly optimized kernels for GAMESS-style workflows. The public mirror tracks interface evolution while associated publications move through review.

  [GitHub](https://github.com/brycewestheimer/public_qccg)
</div>

### GAMESS Fragmentation Modernization
<div class="project-card">
  <div class="project-meta">
    <span class="project-status">Ongoing collaboration</span>
    <span class="project-tech">Fortran, C, CUDA</span>
  </div>

  Core contributor to the GAMESS fragmentation roadmap described in *JCTC* (2023) and *JCP* (2023). Current work targets MAP enablement, rigorous covalent fragment treatments, and accelerator portability in coordination with the Guidez & Lin labs.

  [JCTC 2023 article](https://doi.org/10.1021/acs.jctc.3c00379) · [JCP 2023 article](https://doi.org/10.1063/5.0156399)
</div>

## How These Projects Fit Together

- **MAP & fragment workflows** — `libfrag` prototypes feed directly into GAMESS feature branches and collaborative MAP benchmarks.
- **Accelerated EFP/SAPT** — The `public_libaccefp` and `public_libaccsapt` previews demonstrate GPU-oriented kernels that back MolSSI fellowship milestones and ACS 2024 talks.
- **Code generation** — `public_qccg` underpins automation used across collaborator labs to keep kernels consistent across CPU and GPU targets.

## Collaboration Channels

- Guidez Lab & Lin Group (CU Denver) — Multiscale method development and benchmarking.
- Ames National Laboratory — Extreme-scale deployment of GAMESS and accelerator-focused kernels.
- MolSSI community — Open-source best practices and mentoring for scientific software.

## Get Involved

Interested in contributing or benchmarking one of these projects? Check the open issues in each repository or [reach out](/contact/) with a short note about your use case. I’m especially keen on collaborations that:

- Stress-test MAP workflows on new accelerator hardware.
- Extend EFP/SAPT capabilities to novel materials or condensed-phase systems.
- Integrate the code-generation tooling with external quantum chemistry stacks.

---

*Looking for additional repositories? Everything public lives on [GitHub](https://github.com/brycewestheimer).*
