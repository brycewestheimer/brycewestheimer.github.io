---
layout: page
title: "Tutorials"
description: "Step-by-step guides for computational chemistry and scientific computing"
permalink: /tutorials/
---

## Available Tutorials

<div class="tutorial-grid">
  {% for tutorial in site.tutorials %}
    <div class="tutorial-card">
      <div class="tutorial-header">
        <h3><a href="{{ tutorial.url | relative_url }}">{{ tutorial.title }}</a></h3>
        <div class="tutorial-meta">
          {% if tutorial.difficulty %}
            <span class="difficulty {{ tutorial.difficulty }}">{{ tutorial.difficulty | capitalize }}</span>
          {% endif %}
          {% if tutorial.duration %}
            <span class="duration">{{ tutorial.duration }}</span>
          {% endif %}
          {% if tutorial.topics %}
            {% for topic in tutorial.topics limit:1 %}
              <span class="topic">{{ topic | replace: '-', ' ' | capitalize }}</span>
            {% endfor %}
          {% endif %}
        </div>
      </div>
      
      {% if tutorial.description %}
        <div class="tutorial-description">
          {{ tutorial.description }}
        </div>
      {% endif %}
      
      {% if tutorial.prerequisites %}
        <div class="tutorial-prerequisites">
          <strong>Prerequisites:</strong>
          <ul>
            {% for prereq in tutorial.prerequisites %}
              <li>{{ prereq }}</li>
            {% endfor %}
          </ul>
        </div>
      {% endif %}
      
      <div class="tutorial-footer">
        <a href="{{ tutorial.url | relative_url }}" class="btn btn-primary">Start Tutorial</a>
        {% if tutorial.date %}
          <span class="tutorial-date">Updated {{ tutorial.date | date: "%B %Y" }}</span>
        {% endif %}
      </div>
    </div>
  {% endfor %}
</div>

## Tutorial Categories
    <p>Learn how to set up and run quantum chemistry calculations on GPU hardware. Covers installation, configuration, and basic optimization techniques.</p>
    <div class="tutorial-topics">
      <span class="topic-tag">CUDA</span>
      <span class="topic-tag">DFT</span>
      <span class="topic-tag">Performance</span>
    </div>
  </div>

  <div class="tutorial-card">
    <div class="tutorial-header">
      <h3><a href="/tutorials/python-quantum-chemistry/">Python for Quantum Chemistry</a></h3>
      <div class="tutorial-meta">
        <span class="difficulty beginner">Beginner</span>
        <span class="duration">60 min</span>
        <span class="topic">Python</span>
      </div>
    </div>
    <p>Introduction to using Python for quantum chemistry calculations. Covers PySCF, ASE, and other essential libraries for computational chemistry.</p>
    <div class="tutorial-topics">
      <span class="topic-tag">Python</span>
      <span class="topic-tag">PySCF</span>
      <span class="topic-tag">ASE</span>
    </div>
  </div>

  <div class="tutorial-card">
    <div class="tutorial-header">
      <h3><a href="/tutorials/hpc-workflows/">Building HPC Workflows for Chemistry</a></h3>
      <div class="tutorial-meta">
        <span class="difficulty intermediate">Intermediate</span>
        <span class="duration">90 min</span>
        <span class="topic">HPC</span>
      </div>
    </div>
    <p>Design and implement efficient workflows for running large-scale quantum chemistry calculations on high-performance computing clusters.</p>
    <div class="tutorial-topics">
      <span class="topic-tag">SLURM</span>
      <span class="topic-tag">Workflow</span>
      <span class="topic-tag">Automation</span>
    </div>
  </div>

  <div class="tutorial-card">
    <div class="tutorial-header">
      <h3><a href="/tutorials/ml-chemistry/">Machine Learning in Computational Chemistry</a></h3>
      <div class="tutorial-meta">
        <span class="difficulty intermediate">Intermediate</span>
        <span class="duration">120 min</span>
        <span class="topic">Machine Learning</span>
      </div>
    </div>
    <p>Integrate machine learning models with quantum chemistry methods. Covers property prediction, molecular representation, and model training.</p>
    <div class="tutorial-topics">
      <span class="topic-tag">Scikit-learn</span>
      <span class="topic-tag">TensorFlow</span>
      <span class="topic-tag">Molecular ML</span>
    </div>
  </div>

  <div class="tutorial-card">
    <div class="tutorial-header">
      <h3><a href="/tutorials/code-optimization/">Optimizing Scientific Python Code</a></h3>
      <div class="tutorial-meta">
        <span class="difficulty advanced">Advanced</span>
        <span class="duration">75 min</span>
        <span class="topic">Optimization</span>
      </div>
    </div>
    <p>Advanced techniques for optimizing Python code for scientific computing. Covers profiling, Cython, NumPy optimization, and parallel processing.</p>
    <div class="tutorial-topics">
      <span class="topic-tag">Cython</span>
      <span class="topic-tag">NumPy</span>
      <span class="topic-tag">Profiling</span>
    </div>
  </div>

  <div class="tutorial-card">
    <div class="tutorial-header">
      <h3><a href="/tutorials/docker-chemistry/">Containerized Chemistry with Docker</a></h3>
      <div class="tutorial-meta">
        <span class="difficulty intermediate">Intermediate</span>
        <span class="duration">60 min</span>
        <span class="topic">DevOps</span>
      </div>
    </div>
    <p>Package and deploy computational chemistry applications using Docker containers. Ensures reproducibility and easy deployment.</p>
    <div class="tutorial-topics">
      <span class="topic-tag">Docker</span>
      <span class="topic-tag">Reproducibility</span>
      <span class="topic-tag">Deployment</span>
    </div>
  </div>

</div>

## Tutorial Series

### Quantum Chemistry Programming Series
A comprehensive series covering the fundamentals of programming for quantum chemistry applications.

1. **Introduction to Quantum Chemistry Programming** (Coming Soon)
2. **Electronic Structure Methods Implementation** (Coming Soon)
3. **Basis Set Manipulation and Optimization** (Coming Soon)
4. **Property Calculation and Analysis** (Coming Soon)

### High-Performance Computing Series
Learn to leverage modern computing architectures for quantum chemistry.

1. **Parallel Programming for Chemists** (Coming Soon)
2. **GPU Programming with CUDA** (Coming Soon)
3. **Memory Optimization Techniques** (Coming Soon)
4. **Scaling Calculations to Supercomputers** (Coming Soon)

### Machine Learning for Chemistry Series
Apply modern ML techniques to chemical problems.

1. **Introduction to ML in Chemistry** (Coming Soon)
2. **Molecular Representation Learning** (Coming Soon)
3. **Property Prediction Models** (Coming Soon)
4. **Generative Models for Drug Discovery** (Coming Soon)

## Prerequisites

### For Beginners
- Basic understanding of chemistry concepts
- Some programming experience (any language)
- Familiarity with command line interfaces

### For Intermediate Tutorials
- Solid Python programming skills
- Understanding of quantum chemistry fundamentals
- Experience with Linux/Unix systems

### For Advanced Tutorials
- Strong programming background
- Experience with scientific computing
- Knowledge of computational chemistry methods

## Tutorial Format

Each tutorial includes:

- **Clear learning objectives**
- **Step-by-step instructions**
- **Downloadable code examples**
- **Practice exercises**
- **Common troubleshooting tips**
- **Further reading suggestions**

## Download Materials

All tutorial materials, including code examples, datasets, and additional resources, are available on GitHub:

<div class="download-section">
  <a href="https://github.com/brycewestheimer/tutorials" class="btn btn-primary" target="_blank">
    📚 Download Tutorial Materials
  </a>
</div>

## Interactive Environments

For selected tutorials, interactive Jupyter notebooks are available:

- **Google Colab**: Run tutorials in your browser without installation
- **Binder**: Interactive notebooks with pre-configured environments
- **Local Setup**: Download and run on your own system

## Video Companions

Some tutorials include video walkthroughs and explanations. Check the individual tutorial pages for video links.

## Community and Support

### Getting Help
- **GitHub Issues**: Report problems or ask questions about tutorial content
- **Discussion Forum**: Community discussion and peer support
- **Email Support**: Direct questions to [westheb@gmail.com](mailto:westheb@gmail.com)

### Contributing
I welcome contributions to improve these tutorials:
- **Corrections**: Fix errors or improve clarity
- **Additions**: Suggest new examples or exercises
- **New Tutorials**: Propose topics for future tutorials

## Upcoming Tutorials

Vote for the tutorials you'd most like to see:

<div class="upcoming-tutorials">
  <h4>In Development:</h4>
  <ul>
    <li>Automated Conformational Sampling</li>
    <li>Crystal Structure Prediction</li>
    <li>Reaction Mechanism Discovery</li>
    <li>Property-Driven Molecular Design</li>
    <li>Cloud Computing for Quantum Chemistry</li>
  </ul>
</div>

### Request a Tutorial

Have a specific topic you'd like covered? <a href="/contact/">Send me a request</a> and I'll consider it for future development.

## License and Usage

All tutorial content is released under Creative Commons Attribution 4.0 International License. You're free to:
- Share and adapt the content
- Use for commercial purposes
- Credit the original author

---

*These tutorials represent years of experience in computational chemistry and scientific computing. I hope they help you accelerate your research and development work!*
