---
title: "Protein Hybrid Discrete Dynamics/DFT (*phd3*)"
date: 2023-03-24
repo: "https://github.com/alexandrova-lab-ucla/phd3"
abbr: "phd3"
languages: ["Python"]
authors: ["Matthew R. Hennefarth", "David J. Reilley"]
---

{{<img image="/projects/cpet.jpeg" alt="Protein Hybrid Discrete Dynamics/DFT (phd3)">}}

Protein Hybrid Discrete Dynamics/DFT. Quantum Mechanical/Discrete Molecular Dynamics engine to provide rapid sampling of metalloenzymes.

## System Requirements
- [TURBOMOLE](https://www.turbomole.org/) Version 6.6
- [piDMD](http://www.moleculesinaction.com/pdmd.html)
- [Openbabel](http://openbabel.org/wiki/Main_Page) Version 2.4.0
- [Chimera](https://www.cgl.ucsf.edu/chimera/) Version 1.6.0 or greater (1.15.0 or greater recommended)
- Python Version 3.6 or greater (3.8 or greater recommended)

## Python Package Dependencies
When phd3 is installed via pip, the following packages will also be installed if not already present on your system. 
- jinja2
- sklearn
- propka
- numpy Version 1.20.0 or greater
- hdbscan
- scipy Version 1.1.0

## Installation
First, clone this repository
```sh
git clone https://github.com/alexandrova-lab-ucla/phd3
```
Then, edit the `phd_config.json` file so that all of the paths in the `PATHS` section point to the correct directory. That is
- `TURBODIR` points to the TURBOMOLE directory that contains `basen`, `bin`, `TURBOTEST`, `Config_turbo_env`, etc.
- `DMD_DIR` points to the directory that holds the DMD binaries `complex.linux`, `comples_m2p.linux`, `pdmd.linux`, and `repdmd.linux`. 
- `parameters` points to the directory that holds the DMD parameters file `allatom.par`, `DMD_NA_allatom.par`, etc.
- `chimera` points to the directory that holds the chimera executable. Note, this is not the path TO the chimera executable, but to the directory that contains the executable.

If you are using phd3 on a cluster with queuing, you can modify the QUEUING section as well. Next we make a directory in `~/.config` and moves some files there as

```sh
mkdir ${HOME}/.config/phd3
mv phd_config.json logger_config.json phd3/templates/submit.j2 ${HOME}/.config/phd3
``` 
In order to prevent Turbomole from failing, you may need to add the following to your `.bashrc`. This tells the CPU to use a math library that TURBOMOLE depends on. If this is not used, TURBOMOLE will segfault, even in `define`.
```sh
module load mkl
export MKL_DEBUG_CPU_TYPE=5
```
Finally, we use pip3 to install
```sh
pip3 install ./
```
If you are having issues with 'sudo', add the `--user` option. Further, if you are developing, include the `-e` option as
```sh
pip3 install --user -e ./
``` 
Make sure that `~/.local/bin` is in your PATH as this is where the executibles are installed with pip3.

## Usage
The following scripts are available after installation
- `setupphd.py`
- `runphd.py`
- `submitphd.py`
- `submitdmd.py`
- `setupturbomole.py`
- `runturbomole.py`
- `submitturbomole.py`
- `tfe.py`
- `relabelpdb.py`
- `setupdmd.py`
- `rundmd.py`
- `m2p.py`
- `cutqm.py`

See the [wiki](https://github.com/alexandrova-lab-ucla/phd3/wiki) for a detailed user guide on how to run various calculations.

## Citation
Please cite the following papers if you use QM/DMD in your research on top of the TURBOMOLE and piDMD citations.

- Sparta, M.; Shirvanyants, D.; Ding, F.; Dokholyan, N. V.; Alexandrova, A. N. Hybrid Dynamics Simulation Engine for Metalloproteins. *Biophys. J.*, **2012**, 103, 767-776. DOI: [10.1016/j.bpj.2012.06.024](https://dx.doi.org/10.1016/j.bpj.2012.06.024) 
- Gallpu, N. M.; Alexandrova, A. N. Use of QM/DMD as a Multiscale Approach to Modeling Metalloenzymes. *Mehods Enzymol.*, **2016**, 577, 319-339. DOI: [10.1016/bs.mie.2016.05.018](https://dx.doi.org/10.1016/bs.mie.2016.05.018)
  
If you use the titratable DMD in your work, please cite the following in your research as well as propka.
- Reilley, D. R.; Wang, J.; Dokholyan, N. V.; Alexandrova, A. N. Titr-DMD - A Rapid, Coarse-Grained Quasi-All-Atom Constant pH Molecular Dynamics Framework. *J. Chem. Theory Comput*, **2021**, 17, 4538-4549. DOI: [10.1021/acs.jctc.1c00338](https://dx.doi.org/10.1021/acs.jctc.1c00338)