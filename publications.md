---
layout: page
title: "Publications"
description: "Peer-reviewed publications and research contributions in computational chemistry"
permalink: /publications/
---

## Overview

This page captures peer-reviewed work focused on fragment-based quantum chemistry, extreme-scale electronic structure, and the software that supports those calculations. For an automatically updated list, visit my [Google Scholar profile](https://scholar.google.com/citations?user={{ site.author.google_scholar }}).

## Journal articles & book chapters

### 2024

- **Peng Xu, Bryce M. Westheimer, Megan Schlinsog, Tosaporn Sattasathuchana, George Elliott, Mark S. Gordon, Emilie Guidez.** “The Effective Fragment Potential: An *Ab Initio* Force Field.” In *Comprehensive Computational Chemistry* (Elsevier). DOI: [10.1016/B978-0-12-821978-2.00141-0](https://doi.org/10.1016/B978-0-12-821978-2.00141-0).  
  *Chapter overview*: Summarizes the modern effective fragment potential (EFP) framework, including recent GPU-portable implementations that enable chemically accurate embedding in large condensed-phase systems.

### 2023

- **Jorge L. Galvez Vallejo, Calum Snowdon, Ryan Stocks, Fazeleh Kazemian, Fiona C. Y. Yu, Christopher Seidl, Zoe Seeger, Melisa Alkan, David Poole, Bryce M. Westheimer, et al.** “Toward an extreme-scale electronic structure system.” *The Journal of Chemical Physics*, 159(4), 044112. DOI: [10.1063/5.0156399](https://doi.org/10.1063/5.0156399).  
  *Highlights*: Introduces the EXESS platform and LibCChem 2.0, demonstrating Hartree–Fock and RI-MP2 calculations on 27,600 GPUs with >94% efficiency.
- **Federico Zahariev, Peng Xu, Bryce M. Westheimer, Simon Webb, Jorge Galvez Vallejo, et al.** “The General Atomic and Molecular Electronic Structure System (GAMESS): Novel Methods on Novel Architectures.” *Journal of Chemical Theory and Computation*, 19(20), 7031–7055. DOI: [10.1021/acs.jctc.3c00379](https://doi.org/10.1021/acs.jctc.3c00379).  
  *Highlights*: Documents the modernization of GAMESS, covering new fragmentation capabilities, GPU-accelerated kernels, and interoperability with heterogeneous HPC systems.

### 2022

- **Bryce M. Westheimer, Mark S. Gordon.** “General, Rigorous Approach for the Treatment of Interfragment Covalent Bonds.” *The Journal of Physical Chemistry A*, 126(39), 6995–7006. DOI: [10.1021/acs.jpca.2c04015](https://doi.org/10.1021/acs.jpca.2c04015).  
  *Highlights*: Presents a formally exact treatment for covalent connections in fragment-based quantum chemistry, eliminating ad hoc link atom schemes.

### 2021

- **Bryce M. Westheimer, Mark S. Gordon.** “Scalable *ab initio* fragmentation methods based on a truncated expansion of the non-orthogonal molecular orbital model.” *The Journal of Chemical Physics*, 155(15), 154101. DOI: [10.1063/5.0064864](https://doi.org/10.1063/5.0064864).  
  *Highlights*: Derives the expanded non-orthogonal molecular orbital (X-NOMO) hierarchy, delivering controllable accuracy with near-linear scaling for large molecular clusters.

### 2020

- **Giuseppe M. J. Barca, Colleen Bertoni, Laura Carrington, Dipayan Datta, Nuwan de Silva, ... Bryce M. Westheimer, Peng Xu, Federico Zahariev, Mark S. Gordon.** “Recent developments in the general atomic and molecular electronic structure system.” *The Journal of Chemical Physics*, 152(15), 154102. DOI: [10.1063/5.0005188](https://doi.org/10.1063/5.0005188).  
  *Highlights*: Reviews major advances in GAMESS, including EFP/FMO enhancements, density-functional improvements, and hybrid MPI/OpenMP parallelism paths.
- **Mark S. Gordon, Giuseppe M. J. Barca, Sarom S. Leang, David Poole, Alistair P. Rendell, Jorge L. Galvez Vallejo, Bryce Westheimer.** “Novel Computer Architectures and Quantum Chemistry.” *The Journal of Physical Chemistry A*, 124(23), 4557–4582. DOI: [10.1021/acs.jpca.0c02249](https://doi.org/10.1021/acs.jpca.0c02249).  
  *Highlights*: Surveys the evolution of quantum chemistry software on emerging CPU, GPU, and accelerator hardware, outlining design principles for sustained performance portability.

## Conference proceedings

- **Vaibhav Sundriyal, Masha Sosonkina, Bryce M. Westheimer.** “Comparing Frequency Scaling Efficacy on Different Memory Technologies.” In *2019 Spring Simulation Conference (SpringSim)*, pp. 1–10. DOI: [10.23919/SpringSim.2019.8732866](https://doi.org/10.23919/SpringSim.2019.8732866).  
  *Focus*: Benchmarks core versus uncore frequency scaling strategies for representative HPC workloads on hybrid memory platforms.
- **Vaibhav Sundriyal, Masha Sosonkina, Bryce M. Westheimer, Mark S. Gordon.** “Comparisons of Core and Uncore Frequency Scaling Modes in Quantum Chemistry Application GAMESS.” In *2018 Spring Simulation Conference (HPC Track)*, Article 13:1–13:11. DOI: [10.1145/3200947.3200949](https://doi.org/10.1145/3200947.3200949).  
  *Focus*: Evaluates how frequency scaling choices impact the GAMESS workload mix, guiding energy-efficient deployments on large-scale clusters.

## Staying current

Publication metadata last refreshed from Crossref and dblp on 17 Oct 2025. If a PDF or preprint is needed, please reach out via the [contact page](/contact/).

---

*For collaboration opportunities or questions about any of these publications, please [contact me](/contact/).*
