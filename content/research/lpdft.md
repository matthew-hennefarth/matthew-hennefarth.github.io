---
title: "Linearized Pair-Density Functional Theory"
date: 2022-06-01
draft: false 
current: true
author: "Matthew R. Hennefarth"
math: true
---

Generally speaking, we are interested in understanding how matter and light
interact with each other because of its applications to UV-damage (and
sunscreens), photosynthesis, and catalysis. There is a lot of work focused on
developing electronic structure methods which can accurately compute the
potential energy surface of systems, both the ground and excited state. One such
method, which is particularly attractive, is multiconfiguration pair-density
functional theory (MC-PDFT) due to its speed and accuracy. MC-PDFT is similar to
Kohn-Sham DFT in that it computes the energy for a particular state using a
energy expression which is expressed as a functional of the density.

\begin{equation}
E^\mathrm{PDFT} =
\sum_{pq}h_{pq}D_{pq}+\sum_{pqrs}g_{pqrs}D_{pq}D_{rs}+E_\mathrm{ot}[\rho, \Pi] +
h_\mathrm{nuc}
\end{equation}

While this is good, it unfortunately does not adequtely model the electronic
structure/potential surfaces where states of the same symmetry get close
together in what are called conical intersections or near-avoided crossings.
These regions of the potential surface are very common when modeling
photochemistry, and are the key drivers for internal conversion processes. As
such, I have worked on developing a multi-state PDFT (MS-PDFT) called linearized
PDFT (L-PDFT). L-PDFT works by functionalizing an effective Hamiltonian on a
particular model-space (or set of densities). Essentially, the L-PDFT
Hamiltonian comes from Taylor expanding the MC-PDFT energy expression to first
order around the state-average densities, and extracting the effective linear
operator.

\begin{equation}
  H^\mathrm{L-PDFT} = \sum_{pq}(h_{pq}+V_{pq}^0 + \mathcal{J}\_{pq}^0)E_{pq} + \sum_{pqrs}v_{pqrs}^0e_{pqrs} + h_\mathrm{const}
\end{equation}

Diagonalizing this operator with in an appropriate model-space yields the
correct potential energy surface topology near conical intersections and
near-avoided crossings. Additionally, it is formally faster than MC-PDFT since
it only has a single quadrature calculation. Furthermore, it is as accurate as
MC-PDFT for vertical excitations whereas other MS-PDFT methods are not
necessarily.

L-PDFT also has the advantage of having a *bon afide* operator making it more
suitable for future development. You can check out the first paper describing
the paper [here][**Submitted**, Available on ChemRxiv].

{{< img image="/toc/lpdft_toc.png" alt="L-PDFT TOC" width="50%">}}

This work was performed under the guidance of Professor [Laura
Gagliardi](https://gagliardigroup.uchicago.edu/) during my time as a Ph.D.
student at UChicago.

## Code Developed for this Project
- Linearized PDFT in [*PySCF-forge*](https://github.com/pyscf/pyscf-forge)
- Born-Oppenheimer Molecular Dynamics Package in
  [*PySCF*](https://github.com/pyscf/pyscf)

---
## Selected Publications:
1. [Linearized Pair-Density Functional Theory][**Submitted**, Available on
   ChemRxiv]

[comment]: <Reference Hyperlinks>
[**Submitted**, Available on ChemRxiv]: http://dx.doi.org/10.26434/chemrxiv-2023-7d0gv
