---
title: "*mrh-hf*"
date: 2023-03-24
repo: https://github.com/matthew-hennefarth/mrh-hf
languages: ["Python"]
math: true
authors: ["Matthew R. Hennefarth"]
---
Simple Hartree-Fock code.
<!--more-->
## Requirements
- [PySCF]
- Python3 (version >3.8)

## Usage
Since I use the [PySCF] `gto.M` to build my integrals, we do the exact same
here! Then, we pass the `mol` to the `rhf()` function along with the energy
threshold and maximum number of scf iterations. An example is 

```python
def main():
    mol = gto.M(atom="O 0 -0.14322 0; H 1.63803 1.13654 0; H -1.63803 1.13654 0",
        basis="sto3g", unit="Bohr")
        
    my_energy = rhf(mol, e_thr=1e-9, max_scf=200)
```
It will then print out a whole lot of stuff, with the final energy at the end!


## Theory
This script utilizes the [PySCF] library to generate the 1- and 2- electron
integrals. Specifically, I read in the kinetic energy, electron-nuclear, AO
overlap, and electron-electron repulsion matrices. From these integrals, we can
construct the core matrix as
\begin{equation}
    H^\text{core} = T + V^\text{nuc}
\end{equation}
with $T$ and $V^\text{nuc}$ being the kinetic energy and electron-nuclear
attraction matrix respectively. This matrix does not change throughout the
course of the SCF procedure. Next, we construct the symmetric orthogonal matrix
\begin{equation}
    S^{-1/2} = Us^{-1/2}U^{\dag}
\end{equation}
This is done by diagonalizing the overlap matrix, $S$, and then taking each
eigenvalue (the diagonal elements of $s$) and raising to the $-1/2$ power. $U$
is simply the unitary matrix that represents the eigenvectors of $S$.

We form our initial guess for the SCF produre by diagonalizing the core
Hamiltonian. That is we take $F=H^\text{core}$. Transforming to the orthogonal
MO basis, $F^\prime$:
\begin{equation}
    F^\prime = (S^{-1/2})^{\dag} F S^{-1/2}=C^\prime \varepsilon C
\end{equation}
and diagonalizing yields eigenvectors $C^\prime$, our initial guess. We tranform
back to the AO basis by multiplying by $S^{-1/2}$.
\begin{equation}
    C = S^{-1/2}C^\prime
\end{equation}
The initial density matrix $P$ is given by
\begin{equation}
    P_{\mu\nu} = \sum_{\lambda}^{N/2}C_{\mu\lambda}C_{\nu\lambda}
\end{equation}
Here, $N$ is the number of electrons in the system. Now we begin the SCF
procedure as follows:

1. Construct the $G$ matrix as follows with $\eta$ the 2-electron matrix.
\begin{equation}
    G_{\mu\nu}^{(n)} = \sum_{\lambda\sigma}P_{\lambda\sigma}P^{(n)}(2\eta_{\mu\nu\sigma\lambda} - \eta_{\mu\lambda\sigma\nu})
\end{equation}

2. Construct the Fock matrix, F
\begin{equation}
    F^{(n)} = H^\text{core} + G^{(n)}
\end{equation}

3. Transform to the orthogonal representation $F^{(n)\prime}$ and diagonalize,
   as above.
4. Compute $C^{(n+1)}$ as above. This represents the MO coefficients in the AO
   basis.
5. Compute the new density $P^{(n+1)}$.
6. Compute the electronic energy $E_\text{el}$ via
\begin{equation}
    E_\text{el}^{(n+1)} = \sum_{\mu\nu}P_{\mu\nu}^{(n+1)}\left(H_{\mu\nu}^\text{core} + F_{\mu\nu}^{(n)}\right)
\end{equation}

7. Check convergence by
\begin{align}
    \left|E_\text{el}^{(n)} - E_\text{el}^{(n+1)}\right| &< \text{energy threshold}\\\\
    \left|rmsd\right| &< \text{density threshold}
\end{align}
where
\begin{equation}
    rmsd = \sqrt{\sum_{\mu\nu}\left(P_{\mu\nu}^{(n)}-P_{\mu\nu}^{(n+1)}\right)}
\end{equation}

If both conditions are satisfied, then we are done and converged!

Note in the above, the superscript ($n$) represents the iteration of the SCF
procedure.

[comment]: <Reference Hyperlinks>
[PySCF]: https://github.com/pyscf/pyscf/
