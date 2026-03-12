---
title: "Hello World"
date: 2025-12-01
description: "Welcome to my blog! An introduction to who I am and what I'll be writing about."
tags: ["personal", "introduction"]
---

I have been writing quantum chemistry software for about ten years now, starting as an undergraduate plugging away at Fortran subroutines in GAMESS at Iowa State and continuing through a Ph.D. in the Gordon group, a postdoc at Ames National Laboratory, and my current position at the University of Colorado Denver. Along the way, the problems I work on changed, but the common thread stayed the same: making quantum chemistry work on systems too large for conventional methods, using whatever combination of theory, algorithms, and hardware gets the job done.

This blog is where I plan to write about that work and the topics around it. Some posts will be technical deep-dives into specific methods or software design decisions. Some will be more reflective. I have no fixed schedule in mind other than writing when I have something worth saying.

## What I work on

My research sits at the intersection of quantum chemistry theory and scientific software engineering. The short version: I develop methods that break large molecular systems into fragments, solve each fragment with high-accuracy quantum chemistry, and reassemble the pieces. This makes it possible to run calculations on proteins, molecular clusters, and other systems that would be completely out of reach for conventional electronic structure methods.

In practice, this means I spend my time on a mix of:

- **Fragment-based quantum chemistry.** The fragment molecular orbital method (FMO), the many-body expansion (MBE), and the multi-layer adaptive partitioning (MAP) method I am currently developing with Professors Guidez and Lin at CU Denver. The core challenge is always the same: how do you partition a system so that the fragmentation introduces minimal error, and how do you correct for the error that remains?

- **Molecular integral evaluation.** Two-electron repulsion integrals are the computational bottleneck of nearly every electronic structure calculation. I have worked on GPU-accelerated integral engines, adaptive CPU/GPU dispatch strategies, and the consumer pattern for accumulating Fock matrix contributions without storing the full O(N^4) integral tensor. libaccint is my current integral library.

- **Distributed computing infrastructure.** Large fragment calculations generate thousands of independent sub-problems that need to be distributed across a cluster, along with the data structures that hold partial results. DTL (Distributed Template Library) is a C++20 library I am building for this, with bindings for C, Fortran, and Python.

- **Automation tooling.** Setting up fragment calculations by hand is tedious and error-prone, especially for biological systems with hundreds of residues, ions, and crystallographic waters. autofragment is a Python library that reads structure files, partitions them into chemically sensible fragments, and writes ready-to-run GAMESS input files.

## What I plan to write about

The blog will cover roughly three categories.

**Technical methods and algorithms.** How fragment-based methods work, why GPU integral evaluation is hard, how to structure distributed data for quantum chemistry, what makes SCF convergence fail. These posts will assume some familiarity with quantum chemistry or scientific computing, but I will try to keep them accessible to anyone with a solid undergraduate background in physical chemistry or a related field.

**Software engineering for science.** HPC programming has its own set of best practices that diverge from web development and general-purpose software engineering. Memory layout, cache locality, MPI communication patterns, Fortran interoperability with C++, building and testing numerical code where "correct" has tolerances. These are the lessons that took me years to learn, and most of them are not written down anywhere.

**Tools and tutorials.** Practical guides to using GAMESS, to scientific Python, to the libraries I have built (autofragment, libaccint, DTL). If you want to run an FMO calculation on a protein and you are starting from zero, I want these tutorials to get you there.

## Why a blog

I have a few reasons. The first is that I want a place to explain things in a way that papers do not allow. A methods paper has to be concise and formal. A blog post can include the false starts, the design decisions that did not work out, the practical tips that would never survive peer review but that make the difference between a productive afternoon and a frustrating week.

The second is that I want to share what I have learned about scientific software development with people who are just getting started. When I was a first-year graduate student, I would have killed for a blog post titled "here is how to actually set up an FMO calculation on a protein, step by step, with all the mistakes I made along the way." Those resources barely existed. I want to help fill that gap.

The third is selfish: writing about a topic forces me to understand it better. If I cannot explain something clearly in prose and code, I probably do not understand it as well as I think I do.

## About this site

The site is built with [Astro](https://astro.build/) and hosted on GitHub Pages. I chose Astro because it generates static HTML with zero client-side JavaScript by default, which is all a blog needs. The source is at [github.com/brycewestheimer/brycewestheimer.github.io](https://github.com/brycewestheimer/brycewestheimer.github.io) if you are curious about the setup.

The tutorials section has hands-on guides that are longer and more structured than blog posts. The projects page lists my open-source work with links to the repositories. If you want to get in touch, the [contact page](/contact/) has my email and other coordinates.

Thanks for reading. More to come.
