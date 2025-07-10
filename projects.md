---
layout: page
title: "Projects"
description: "A showcase of my computational chemistry and software development projects"
permalink: /projects/
---

## Featured Projects

### QCOptimizer: GPU-Accelerated Quantum Chemistry Toolkit
<div class="project-card featured">
  <div class="project-meta">
    <span class="project-status">Active Development</span>
    <span class="project-tech">CUDA, C++, Python</span>
  </div>
  
  **Description**: A comprehensive toolkit for accelerating quantum chemistry calculations on GPU architectures. Features optimized implementations of density functional theory and coupled cluster methods.

  **Key Features**:
  - CUDA kernels for computationally intensive operations
  - Python interface for ease of use
  - Support for multiple GPU architectures
  - Automatic load balancing for multi-GPU systems

  **Impact**: Achieves 10-50x speedup over CPU-only implementations for typical molecular systems.

  [View on GitHub](https://github.com/brycewestheimer/qcoptimizer) | [Documentation](https://qcoptimizer.readthedocs.io)
</div>

### MLChem: Machine Learning for Quantum Chemistry
<div class="project-card featured">
  <div class="project-meta">
    <span class="project-status">Published</span>
    <span class="project-tech">Python, TensorFlow, PyTorch</span>
  </div>
  
  **Description**: A library that integrates machine learning models with traditional quantum chemistry methods to accelerate calculations and improve predictions.

  **Key Features**:
  - Pre-trained models for property prediction
  - Integration with popular quantum chemistry packages
  - Active learning algorithms for efficient data collection
  - Uncertainty quantification for model predictions

  **Applications**: Drug discovery, materials screening, reaction prediction.

  [View on GitHub](https://github.com/brycewestheimer/mlchem) | [Paper](https://doi.org/example) | [Tutorial](/tutorials/mlchem-intro/)
</div>

### HighThroughputQC: Automated Quantum Chemistry Workflows
<div class="project-card">
  <div class="project-meta">
    <span class="project-status">Stable</span>
    <span class="project-tech">Python, Bash, SLURM</span>
  </div>
  
  **Description**: A workflow management system for running large-scale quantum chemistry calculations on high-performance computing clusters.

  **Key Features**:
  - Automatic job submission and monitoring
  - Error handling and restart capabilities
  - Resource optimization
  - Results aggregation and analysis

  **Use Cases**: Virtual screening, conformational sampling, parameter optimization.

  [View on GitHub](https://github.com/brycewestheimer/highthroughputqc)
</div>

## GitHub Repositories

<div id="github-projects" class="github-projects">
  <div class="loading">Loading GitHub repositories...</div>
</div>

<script>
// Fetch and display GitHub repositories
async function loadGitHubRepos() {
  try {
    const response = await fetch('https://api.github.com/users/brycewestheimer/repos?sort=updated&per_page=20');
    const repos = await response.json();
    
    const container = document.getElementById('github-projects');
    container.innerHTML = '';
    
    repos.forEach(repo => {
      if (!repo.fork && repo.name !== 'brycewestheimer.github.io') {
        const repoCard = createRepoCard(repo);
        container.appendChild(repoCard);
      }
    });
  } catch (error) {
    document.getElementById('github-projects').innerHTML = 
      '<p>Unable to load GitHub repositories. <a href="https://github.com/brycewestheimer">View on GitHub</a></p>';
  }
}

function createRepoCard(repo) {
  const card = document.createElement('div');
  card.className = 'repo-card';
  
  card.innerHTML = `
    <div class="repo-header">
      <h3><a href="${repo.html_url}" target="_blank">${repo.name}</a></h3>
      <div class="repo-stats">
        <span class="repo-language">${repo.language || 'Unknown'}</span>
        <span class="repo-stars">⭐ ${repo.stargazers_count}</span>
        <span class="repo-forks">🍴 ${repo.forks_count}</span>
      </div>
    </div>
    <p class="repo-description">${repo.description || 'No description available'}</p>
    <div class="repo-topics">
      ${repo.topics ? repo.topics.map(topic => `<span class="topic-tag">${topic}</span>`).join('') : ''}
    </div>
    <div class="repo-meta">
      <span class="repo-updated">Updated ${formatDate(repo.updated_at)}</span>
    </div>
  `;
  
  return card;
}

function formatDate(dateString) {
  const date = new Date(dateString);
  return date.toLocaleDateString('en-US', { 
    year: 'numeric', 
    month: 'short', 
    day: 'numeric' 
  });
}

// Load repositories when page loads
document.addEventListener('DOMContentLoaded', loadGitHubRepos);
</script>

## Project Categories

### Quantum Chemistry Methods
Projects focused on developing new theoretical approaches and implementing efficient algorithms for electronic structure calculations.

### High-Performance Computing
Software optimized for modern computing architectures, including parallel algorithms and GPU acceleration.

### Machine Learning Applications
Integration of machine learning techniques with traditional computational chemistry methods.

### Industry Tools
Practical software solutions designed for real-world applications in pharmaceutical and materials industries.

### Educational Resources
Tutorials, examples, and teaching materials for computational chemistry and scientific programming.

## Collaboration and Contributions

### Open Source Contributions
I actively contribute to several open-source quantum chemistry projects:
- **PySCF**: Python-based quantum chemistry package
- **OpenMM**: Molecular dynamics simulation toolkit
- **DIRAC**: Relativistic quantum chemistry program

### Academic Collaborations
Current collaborative projects with:
- Pacific Northwest National Laboratory
- University research groups
- Industry partners in pharmaceutical and materials sectors

### Community Involvement
- Reviewer for Journal of Chemical Theory and Computation
- Member of OpenEye Scientific Software advisory board
- Organizer for Pacific Northwest Computational Chemistry meetups

## Future Projects

### Planned Developments
- **QuantumCloud**: Cloud-based quantum chemistry calculations
- **ChemML-Bench**: Standardized benchmarks for machine learning in chemistry
- **GPUChem**: Comprehensive GPU acceleration library

### Seeking Collaborators
I'm always interested in new collaborations. Current areas of interest:
- Industry applications of quantum chemistry
- Educational technology for scientific computing
- Open-source software development
- Machine learning method development

---

*Interested in collaborating on any of these projects? [Get in touch](/contact/) to discuss opportunities!*
