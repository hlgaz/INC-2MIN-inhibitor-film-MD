[README(1).md](https://github.com/user-attachments/files/32369523/README.1.md)
# Competing solvation and iron-anchoring roles of a carbonyl group in indole corrosion inhibitors: evidence from molecular dynamics and SCC-DFTB

Data and inputs supporting the manuscript:



Abdelilah El-khlifi1, Mouslim Messali2, Hajar Elmrayej3, Han-seung Lee4, Noureddine El Messaoudi2, Hassane Lgaz5\*



1Team of Materials, Electrochemistry and Environment, Laboratory of Organic Chemistry, Catalysis, and Environment, Faculty of Sciences, Ibn Tofail University, BP 133, Kenitra 14000, Morocco

2Department of Chemistry, College of Science, Imam Mohammad Ibn Saud Islamic University (IMSIU), P.O. Box, 90950, Riyadh 11623, Saudi Arabia

3Laboratory of Engineering, Electrochemistry, Modeling and Environment, Faculty of Sciences, Sidi Mohamed Ben Abdellah University, Fez, Morocco.

4Department of Architectural Engineering, Hanyang University ERICA, 55 Hanyangdaehak-ro, Sangrok-gu, Ansan-si 15588, Gyeonggi-do, Republic of Korea

5Innovative Durable Building and Infrastructure Research Center, Center for Creative Convergence Education, Hanyang University ERICA, 55 Hanyangdaehak-ro, Sangrok-gu, Ansan-si 15588, Gyeonggi-do, Republic of Korea.



Corresponding author: hlgaz@hanyang.ac.kr 



Journal \[Computational Materials Science]

DOI: \[to be added on acceptance]



This repository contains everything needed to reproduce the classical molecular dynamics (MD) part of the study: force-field parameters, starting structures, GROMACS run inputs, the production run-input files (`.tpr`), analysis outputs underlying every MD figure and table, and the scripts/commands used. The full MD trajectories (\~0.5 GB per system) are not hosted here and are available from the corresponding author on reasonable request.

## Molecules and naming

Files keep the internal labels used during the simulations:

|Label in files|Molecule|Abbreviation in manuscript|
|-|-|-|
|`INH3`|indole-3-carbaldehyde|**INC**|
|`INH4`|2-methylindole|**2MIN**|

Folders are named by the manuscript abbreviation (`INC/`, `2MIN/`); file names inside retain `INH3`/`INH4`.

## Repository layout

```
01\_parameterization/{INC,2MIN}/   antechamber/parmchk2 outputs per molecule
    INHx.mol2                      GAFF2 atom types, AM1-BCC partial charges
    INHx.frcmod                    supplementary force-field parameters (parmchk2)
    INHx.pdb, INHx\_clean.pdb       optimized single-molecule geometry

02\_system\_build/{INC,2MIN}/       assembly of the 30-molecule film in 1.0 mol/L HCl
    packmol\_INHx.inp               Packmol input (60 x 60 x 110 A box)
    INHx\_packed.pdb                Packmol output (starting configuration)
    tleap\_lib\_INHx.in / .log       tleap: molecule library
    tleap\_system\_INHx.in / .log    tleap: solvated system (TIP3P, Cl-, H3O+)
    INHx\_system.prmtop/.inpcrd     AMBER topology/coordinates (converted to GROMACS by ACPYPE)

03\_md/mdp/                        GROMACS run parameters (shared by both systems)
    em.mdp                         steepest descent, emtol 500 kJ mol-1 nm-1
    nvt\_heat.mdp                   NVT annealing 10 -> 298 K, 100 ps, 1 fs, tau\_T 0.5 ps
    npt\_equil.mdp                  NPT 298 K / 1 bar, 500 ps, Parrinello-Rahman (tau\_P 2.0 ps)
    nvt\_prod.mdp                   NVT production 298 K, 5 ns, 2 fs, coordinates every 1 ps

03\_md/{INC,2MIN}/                 per-system GROMACS files
    system.top, system.gro         topology and starting coordinates (from ACPYPE)
    index.ndx, hbond.ndx           index groups (Inhibitor | Solvent; H-bond donors/acceptors)
    em.tpr / nvt\_heat.tpr / npt\_equil.tpr / nvt\_prod.tpr   run inputs for each stage
    em.gro / nvt\_heat.gro / npt\_equil.gro / nvt\_prod.gro   final coordinates of each stage
    \*.log, grompp\_\*.log, mdrun\_\*.log, mdout.mdp             GROMACS logs and expanded parameters
    analysis/                      density, MSD, RDF, temperature, total energy (.xvg + .log)
    analysis\_advanced/             H-bonds, SASA, radius of gyration, film thickness,
                                   inhibitor-inhibitor / inhibitor-solvent energy decomposition
                                   (energy\_rerun.mdp/.tpr = gmx mdrun -rerun setup)
    visualization/frame\_{0..5}ns.pdb   snapshots at 1 ns intervals

MANIFEST.txt                      full file list with sizes
```

## Software

|Step|Software|
|-|-|
|Atom typing, AM1-BCC charges|antechamber, parmchk2 (AmberTools)|
|Starting configuration|Packmol|
|AMBER topology|tleap (AmberTools); TIP3P water; Joung-Cheatham Cl-|
|Conversion to GROMACS|ACPYPE|
|MD engine|GROMACS 2024.4|
|Visualization|VMD|

## Simulation protocol (from `03\_md/mdp/`)

1. `em.mdp` — steepest-descent minimization to a maximum force of 500 kJ mol-1 nm-1.
2. `nvt\_heat.mdp` — NVT heating from 10 to 298 K over 100 ps (linear annealing; 1 fs step; V-rescale, tau\_T = 0.5 ps; velocities generated at 10 K).
3. `npt\_equil.mdp` — NPT equilibration, 500 ps at 298 K and 1 bar (V-rescale tau\_T = 0.1 ps; isotropic Parrinello-Rahman, tau\_P = 2.0 ps, compressibility 4.5e-5 bar-1).
4. `nvt\_prod.mdp` — NVT production, 5 ns at 298 K, 2 fs step, coordinates every 1 ps (fixed volume keeps a stationary frame for the MSD).

Common settings: PME electrostatics (rcoulomb 1.2 nm, Fourier spacing 0.12 nm), van der Waals cutoff 1.2 nm with long-range dispersion correction (EnerPres), Verlet neighbour list, LINCS on bonds to hydrogen, two thermostat groups (Inhibitor | Solvent), 3D periodic boundaries.

## Reproducing a run

```bash
cd 03\_md/INC          # or 03\_md/2MIN
gmx grompp -f ../mdp/em.mdp        -c system.gro    -p system.top -n index.ndx -o em.tpr
gmx mdrun  -deffnm em
gmx grompp -f ../mdp/nvt\_heat.mdp  -c em.gro        -p system.top -n index.ndx -o nvt\_heat.tpr
gmx mdrun  -deffnm nvt\_heat
gmx grompp -f ../mdp/npt\_equil.mdp -c nvt\_heat.gro  -t nvt\_heat.cpt -p system.top -n index.ndx -o npt\_equil.tpr
gmx mdrun  -deffnm npt\_equil
gmx grompp -f ../mdp/nvt\_prod.mdp  -c npt\_equil.gro -t npt\_equil.cpt -p system.top -n index.ndx -o nvt\_prod.tpr
gmx mdrun  -deffnm nvt\_prod
```

The deposited `nvt\_prod.tpr` files can be used directly with `gmx mdrun -s nvt\_prod.tpr` to regenerate the production trajectory. Analyses were run with the standard GROMACS tools (`gmx density`, `gmx msd`, `gmx rdf`, `gmx hbond`, `gmx sasa`, `gmx gyrate`, `gmx energy`); the interaction-energy decomposition used `gmx mdrun -rerun` with the energy groups defined in `analysis\_advanced/energy\_rerun.mdp`. The exact commands and their output are recorded in the `.log` files next to each result.

## Map from manuscript items to files

|Manuscript item|Files (`03\_md/<mol>/` unless stated)|
|-|-|
|Initial simulation boxes|`02\_system\_build/<mol>/INHx\_packed.pdb`|
|MD snapshots (0, 3, 5 ns)|`visualization/frame\_\*ns.pdb`|
|Density profile, film thickness (FWHM)|`analysis/density\_inhibitor\_z.xvg`, `analysis/density\_water\_z.xvg`; `analysis\_advanced/film\_thickness.txt`|
|Mean square displacement, diffusion coefficient|`analysis/msd\_inhibitor.xvg` (fit in `analysis/msd.log`)|
|Radial distribution functions|`analysis/rdf\_inhibitor\_water.xvg`, `rdf\_inhibitor\_ions.xvg`, `rdf\_inhibitor\_inhibitor.xvg`|
|Hydrogen bonds (inhibitor-water, inhibitor-inhibitor)|`analysis\_advanced/hbond\_inh\_water.xvg`, `hbond\_inh\_inh.xvg`|
|Solvent-accessible surface area|`analysis\_advanced/sasa\_inhibitor.xvg`, `sasa\_per\_residue.xvg`|
|Radius of gyration|`analysis\_advanced/gyrate\_inhibitor.xvg`|
|Interaction-energy decomposition (LJ / Coulomb; inh-inh, inh-solvent)|`analysis\_advanced/energy\_inh\_inh.xvg`, `energy\_inh\_solvent.xvg`|
|Summary table of MD descriptors|`analysis/summary\_INHx.txt`, `analysis\_advanced/summary\_advanced\_INHx.txt`|
|Equilibration checks (temperature, total energy)|`analysis/temperature.xvg`, `analysis/total\_energy.xvg`|

## Not included

* Production trajectories `nvt\_prod.xtc` (\~540 MB each), the equilibration trajectories, checkpoint (`.cpt`) and energy (`.edr`) files. Available from the corresponding author on reasonable request; all analysis outputs derived from them are included.

## Citation and license

If you use these data, please cite the manuscript above. Data and analysis outputs are released under CC BY 4.0; scripts and input files under the MIT License.

## Contact

Corresponding author: \[Hassane LGAZ, Innovative Durable Building and Infrastructure Research Center, Center for Creative Convergence Education, Hanyang University ERICA, 55 Hanyangdaehak-ro, Sangrok-gu, Ansan-si 15588, Gyeonggi-do, Republic of Korea, hlgaz@hanyang.ac.kr]

