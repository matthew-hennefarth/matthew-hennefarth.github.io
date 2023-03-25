---
title: "Computationally Designed Artificial Enzymes"
date: 2018-09-25
tags: ["Archive"]
---
Enzymes catalyze chemical reactions with incredible efficiency and stereoselectivity under physiological conditions, making enzymes desirable tools for catalyzing non-native reactions of human interest. Designing artificial enzymes requires understanding the various parameters that make enzymes good catalysts such as substrate binding affinity, active site residues, etc. Artificial enzymes are designed computationally or produced via directed evolution. Computational design protocols have been successful in developing catalytic proteins, yet the best enzymes are still several orders of magnitude less active than natural enzymes. Directed evolved enzymes are often more better catalysts and often, computationally designed enzymes are further refined via directed evolution. However, since directed evolution mimics natural evolution, it does not tell us why particular point mutations are important or how they improve an enzymes efficiency. Hence our current understanding of why enzymes are effective catalysts is missing some fundamental aspect that directed evolution can account for. 

{{< img image="/protein-electrostatic/electrostatic_preorganization.png" alt="Electrostatic Preorganization" width="100%"
    caption="Illustration of the 'electrostatic inverse design problem'. That is, how should you place charges (or charged residues) around the following reaction? There is in zero and infinite number of possible configuration of charges as well as perhaps non-unique solutions which minimize the reaction barrier. Somehow, nature knows exactly how to do this!"
>}}

While a protein can be comprised of several hundred amino acid residues, only a small subset of these residues actually interacts with the substrate during the reaction. This begs the question of the necessity of the rest of the protein scaffold for catalysis. It is hypothesized that the rest of the protein contributes to catalysis by producing a highly specific and preorganized electric field that facilitates the reaction. My research focused on quantifying and understanding how shifts in the electric field created by the protein scaffold affected the proteins' reactivity. I developed a novel method of quantifying differences between electric fields as well as studied how shifts in the electric field are manifested as signatures within the active-space electron density. Additionally, we highlighted how these metrics can be incorporated into the computation design-protocol of novel artificial enzymes to enhance their efficiency and achieve a reactivity similar to those of natural enzymes.

This work as performed under the guidance of Professor [Anastassia N. Alexandrova](https://www.chem.ucla.edu/~ana/) during my time as a staff scientist at UCLA.

## Code Developed for this Project
- Classical Protein Electric Field Topology ([*CPET*]({{<ref "/projects/cpet">}}))
- [*RAII-Thread*]({{<ref "/projects/raii-thread">}})
- [*VectorFieldAnalysis*](https://github.com/matthew-hennefarth/VectorFieldAnalysis)
- Protein Hybrid Discrete Dynamics/DFT ([*phd3*]({{<ref "/projects/phd3">}}))

---
## Selected Publications:
1. [Advances in optimizing enzyme electrostatic preorganization][*Curr. Opin. Struct. Biol.* **2022**, 72, 1]
2. [Heterogeneous Intramolecular Electric Field as a Descriptor of Diels–Alder Reactivity][*JPC A* **2021**, 125, 1289]
3. [Direct Look at the Electric Field in Ketosteroid Isomerase and Its Variants][*ACS Catal.* **2020**, 10, 9915]
4. [The Case for Enzymatic Competitive Metal Affinity Methods][*ACS Catal.* **2020**, 10, 2298]

[comment]: <Reference Hyperlinkes>
[*ACS Catal.* **2020**, 10, 2298]: https://dx.doi.org/10.1021/acscatal.9b04831
[*Curr. Opin. Struct. Biol.* **2022**, 72, 1]: https://dx.doi.org/10.1016/j.sbi.2021.06.006
[*JPC A* **2021**, 125, 1289]: https://dx.doi.org/10.1021/acs.jpca.1c00181
[*ACS Catal.* **2020**, 10, 9915]: https://dx.doi.org/10.1021/acscatal.0c02795
