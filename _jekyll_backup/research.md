---
layout: page
title: "Research"
description: "Exploring quantum chemistry method development and high-performance computing applications"
permalink: /research/
---

## Research Overview

My work integrates quantum chemistry method development with performance-portable software aimed at accelerator-heavy computing environments. I collaborate closely with the Guidez and Lin groups at the University of Colorado Denver as well as with Ames National Laboratory teams to push fragment-based methods toward chemically realistic scales.

## Core Themes

### Multi-layer adaptive partitioning (MAP)

- Extending MAP approaches to treat complex reactive environments while maintaining chemical accuracy.
- Designing hierarchical partitioning strategies that balance subsystem fidelity and computational cost.
- Building reusable MAP analysis workflows for collaborators.

### Fragment-based electronic structure at scale

- Advancing interfragment coupling models for covalent systems, including rigorous treatments published in *J. Phys. Chem. A* and *J. Chem. Phys.*
- Modernizing GAMESS fragmentation modules to run efficiently on GPU-enabled supercomputers.
- Maintaining [`libfrag`](https://github.com/brycewestheimer/libfrag) as an open-source sandbox for new fragment-based algorithms.

### High-performance computing & accelerator enablement

- Optimizing quantum chemistry kernels for heterogeneous CPU/GPU architectures through MPI/OpenMP hybrids and CUDA implementations.
- Benchmarking adaptive partitioning workloads on DOE-class systems in partnership with Ames National Laboratory.
- Leading efforts to ensure novel methods land in production-quality, maintainable code bases.

### Community-facing infrastructure

- Delivering documentation, example inputs, and developer guides that help external groups adopt MAP-driven tools.
- Coordinating with MolSSI and multi-institutional teams to align software roadmaps with user needs.

## Active Projects

- **Subsystem-local SAPT acceleration** — Applying resolution-of-identity techniques to reduce the cost of exchange-repulsion terms, the subject of my ACS Fall 2024 presentation.
- **Accelerated MAP workflow** — Co-developing GPU-ready MAP components within the Guidez and Lin labs, targeting realistic condensed-phase simulations.
- **GAMESS extreme-scale modernization** — Continuing collaborations with Ames National Laboratory to deploy new fragmentation kernels across DOE leadership-class machines.

## Recent Highlights

- Presented "Subsystem-local resolution of the identity" at the ACS Fall 2024 PHYS Division meeting and presided over the correlated many-body embedding symposium.
- Awarded the MolSSI Software Fellowship (2020–2021) to architect open-source libraries for fragment-based quantum chemistry.
- Recognized with the Klaus Ruedenberg Theoretical Chemistry Award at Iowa State University for dissertation-era advances in covalent fragment coupling.

## Collaborations & Partnerships

- **University of Colorado Denver** — Daily collaboration with the Guidez Lab (multiscale methods) and Lin Group (computational chemistry applications).
- **Ames National Laboratory & Iowa State University** — Ongoing GAMESS development, exascale readiness, and co-authorship on novel architecture studies.
- **MolSSI community** — Contributions to best practices for sustainable scientific software and mentorship of incoming fellows.

## Research Outputs

- Peer-reviewed publications span fragment-based theory, scalable electronic structure algorithms, and novel computer architecture studies. See the [publications page](/publications/) or [Google Scholar](https://scholar.google.com/citations?user={{ site.author.google_scholar }}) for the full list.
- Open-source releases include `libfrag`, `public_libaccefp`, and `public_libaccsapt`, with additional MAP utilities under active development.
- Internal documentation and tutorials power recurring workshops for Guidez/Lin group members and collaborators exploring MAP workflows.

---

Interested in collaborating or learning more about these projects? Reach out via the [contact page](/contact/) or open an issue on GitHub.
