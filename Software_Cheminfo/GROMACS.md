# GROMACS
Offical manual : https://manual.gromacs.org/current/index.html  

# Biological problems
- protein behavior
- binding site
- free energy calculation
- statistics and quantification
# Install (Ubuntu)
Ref : https://manual.gromacs.org/current/download.html  
Install guide : https://manual.gromacs.org/current/install-guide/index.html  
video : https://www.youtube.com/watch?v=kCKYkNygc9I  
- apt-get install
  ```
  apt install gcc
  apt install cmake
  apt-get install build-essential
  apt-get install libfftw3-dev
  apt-get install gromacs
  ```
	- Ubuntu LLVM Package
		Official documentation:https://llvm.org/docs/GettingStarted.html#checkout   
		Automatic installation: https://ralpioxxcs.github.io/post/etc/llvm_install/   
- installation w/ GPU
	```
	mkdir build
	cd build
	cmake .. -DGMX_GPU=CUDA -DGMX_OPENMP=ON -DGMX_BUILD_OWN_FFTW=ON -CUDA_TOOLKIT_ROOT_DIT=/usr/local/cuda -DCMAKE_INSTALL_PREFIX=/opt/gromacs/2022.2/ -> not work
	OR
	cmake .. -DGMX_GPU=CUDA -DGMX_OPENMP=ON -DGMX_BUILD_OWN_FFTW=ON -DCMAKE_INSTALL_PREFIX=/opt/gromacs/2022.2/
	make
	make check
	sudo make install
	source /usr/local/gromacs/bin/GMXRC
	```
## Install in Centos7
Ref: https://sites.google.com/site/rangsiman1993/comp-chem/program-install/install-gromacs-4
- installing fftw-3.3.6
	```
	wget ftp://ftp.fftw.org/pub/fftw/fftw-3.3.6-pl2.tar.gz
	tar xzf fftw-3.3.6-pl2.tar.gz
	chown root:root -R fftw-3.3.6-pl2
	cd fftw-3.3.6-pl2
	## creating single- and double-precision versions ###
	./configure --enable-threads --enable-float --prefix=/usr/local/fftw-3.3.6-pl2
	make
	make install
	make distclean
	./configure --enable-threads --prefix=/usr/local/fftw-3.3.6-pl2
	make
	make install
	```
- install `Gromacs`
	```
	wget ftp://ftp.gromacs.org/pub/gromacs/gromacs-4.5.3.tar.gz
	tar xzf gromacs-4.5.3.tar.gz
	chown root:root -R gromacs-4.5.3
	cd gromacs-4.5.3
	## Compiling Gromacs
	./configure --prefix=/usr/local/bin --enable-mpi LDFLAGS=-L/usr/local/fftw-3.3.6-pl2/lib \
	CPPFLAGS=-I/usr/local/fftw-3.3.6-pl2/include
	make
	make &> log
	make mdrun
	make install
	make install-mdrun
	make links
	```
## Install in MacOS
Ref: https://bioinformaticsreview.com/20220206/how-to-install-gromacs-on-apple-m1-macos/
	```
	% cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=OFF -DCMAKE_C_COMPILER=gcc -DREGRESSIONTEST_PATH=/Users/jihun/Downloads/software/regressiontests-2022.2
	% sudo make check
	```

# Basics
```
gmx help (module)
gmx (module) -h
```
Online Reference : https://manual.gromacs.org/archive/4.6.5/online.html  
# Lysozyme Tutorial
Ref: http://www.mdtutorials.com/gmx/lysozyme/index.html
![image](https://user-images.githubusercontent.com/48517782/172881380-7c7b5c8e-adb2-446b-a686-7adc91012dcb.png)

## Generate Topology
To strip out the crystal waters (Note that such a procedure is not universally appropriate)
```
grep -v HOH 1aki.pdb > 1AKI_clean.pdb
```
Always check your .pdb file for entries listed under the comment MISSING, as these entries indicate either atoms or whole residues that are not present in the crystal structure. Terminal regions may be absent, and may not present a problem for dynamics. Incomplete internal sequences or any amino acid residues that have missing atoms will cause pdb2gmx to fail. These missing atoms/residues must be modeled in using other software packages. Also note that pdb2gmx is not magic. It cannot generate topologies for arbitrary molecules, just the residues defined by the force field (in the *.rtp files - generally proteins, nucleic acids, and a very finite amount of cofactors, like NAD(H) and ATP).


The purpose of pdb2gmx is to generate three files:
- The topology for the molecule (topol.top by default) : all the information necessary to define the molecule within a simulation. This information includes nonbonded parameters (atom types and charges) as well as bonded parameters (bonds, angles, and dihedrals).
- A position restraint file.
- A post-processed structure file.


an input pdb file should contain all atoms
```
gmx pdb2gmx -f 1AKI_clean.pdb -o 1AKI_processed.gro -water spce
```
> -f : input file
> -ignh :  Ignore H atoms in the PDB file; especially useful for NMR structures. Otherwise, if H atoms are present, they must be in the named exactly how the force fields in GROMACS expect them to be. Different conventions exist, so dealing with H atoms can occasionally be a headache! If you need to preserve the initial H coordinates, but renaming is required, then the Linux sed command is your friend.
> -ter: Interactively assign charge states for N- and C-termini.
> -inter: Interactively assign charge states for Glu, Asp, Lys, Arg, and His; choose which Cys are involved in disulfide bonds.

To select the Force Field (The force field will contain the information that will be written to the topology. This is a very important choice! You should always read thoroughly about each force field and decide which is most applicable to your situation.)
```
Select the Force Field:
From '/usr/local/gromacs/share/gromacs/top':
 1: AMBER03 protein, nucleic AMBER94 (Duan et al., J. Comp. Chem. 24, 1999-2012, 2003)
 2: AMBER94 force field (Cornell et al., JACS 117, 5179-5197, 1995)
 3: AMBER96 protein, nucleic AMBER94 (Kollman et al., Acc. Chem. Res. 29, 461-469, 1996)
 4: AMBER99 protein, nucleic AMBER94 (Wang et al., J. Comp. Chem. 21, 1049-1074, 2000)
 5: AMBER99SB protein, nucleic AMBER94 (Hornak et al., Proteins 65, 712-725, 2006)
 6: AMBER99SB-ILDN protein, nucleic AMBER94 (Lindorff-Larsen et al., Proteins 78, 1950-58, 2010)
 7: AMBERGS force field (Garcia & Sanbonmatsu, PNAS 99, 2782-2787, 2002)
 8: CHARMM27 all-atom force field (CHARM22 plus CMAP for proteins)
 9: GROMOS96 43a1 force field
10: GROMOS96 43a2 force field (improved alkane dihedrals)
11: GROMOS96 45a3 force field (Schuler JCC 2001 22 1205)
12: GROMOS96 53a5 force field (JCC 2004 vol 25 pag 1656)
13: GROMOS96 53a6 force field (JCC 2004 vol 25 pag 1656)
14: GROMOS96 54a7 force field (Eur. Biophys. J. (2011), 40,, 843-856, DOI: 10.1007/s00249-011-0700-9)
15: OPLS-AA/L all-atom force field (2001 aminoacid dihedrals)
```
Output :
- 1AKI_processed.gro is a GROMACS-formatted structure file that contains all the atoms defined within the force field (i.e., H atoms have been added to the amino acids in the protein).
- The topol.top file is the system topology
  
  The interpretation of this information is as follows:
  nr: Atom number
  type: Atom type
  resnr: Amino acid residue number
  residue: The amino acid residue name
  Note that this residue was "LYS" in the PDB file; the use of .rtp entry "LYSH" indicates that the residue is protonated (the predominant state at neutral pH).
  atom: Atom name
  cgnr: Charge group number
  Charge groups define units of integer charge; they aid in speeding up calculations
  charge: Self-explanatory
  The "qtot" descriptor is a running total of the charge on the molecule
  mass: Also self-explanatory
  typeB, chargeB, massB: Used for free energy perturbation (not discussed here)

- The posre.itp file contains information used to restrain the positions of heavy atoms (more on this later).
- Trial and Error:
	- the input structure to pdb2gmx has hydrogen atoms whose naming differs from that given in the .rtp file of the force field.
		Ref: https://www.researchgate.net/post/How-can-I-rectify-the-following-GROMACS-error-Fatal-error-Atom-HB3-in-residue-SER-3-was-not-found-in-rtp-entry-SER-with-8-atoms-while-sorting-atoms   


To generate a box for simulation (`box.gro`)
```
gmx editconf -f file.gro -o box.gro -c -d 1.0 -bt cubic 
```
> -d : distance from cubic box
> -  the box size does not significantly affect the mobility of protein atoms on a relatively short trajectory, while the effect of the precipitant concentration on this trajectory is noticeable. (Ref: Kordonskaya, Y.V., Timofeev, V.I., Dyakova, Y.A. et al. Effect of the Simulation Box Size and Precipitant Concentration on the Behavior of Tetragonal Lysozyme Dimer. Crystallogr. Rep. 66, 525–528 (2021). https://doi.org/10.1134/S106377452103010X)
```
gmx solvate -cp box.gro -cs configuration_of_solvent_from_library.gro -o water_box.gro -p file.top
```
Assemble tpr file
```
gmx grompp -f inos.mdp -c water_box.gro -p file.top -o ions.tpr
```

To add ion
`ions.mdp` file
```
; ions.mdp - used as input into grompp to generate ions.tpr
; Parameters describing what to do, when to stop and what to save
integrator  = steep         ; Algorithm (steep = steepest descent minimization)
emtol       = 1000.0        ; Stop minimization when the maximum force < 1000.0 kJ/mol/nm
emstep      = 0.01          ; Minimization step size
nsteps      = 50000         ; Maximum number of (minimization) steps to perform

; Parameters describing how to find the neighbors of each atom and how to calculate the interactions
nstlist         = 1         ; Frequency to update the neighbor list and long range forces
cutoff-scheme	= Verlet    ; Buffered neighbor searching 
ns_type         = grid      ; Method to determine neighbor list (simple, grid)
coulombtype     = cutoff    ; Treatment of long range electrostatic interactions
rcoulomb        = 1.0       ; Short-range electrostatic cut-off
rvdw            = 1.0       ; Short-range Van der Waals cut-off
pbc             = xyz       ; Periodic Boundary Conditions in all 3 dimensions
```

```
gmx genioin -s ions.tpr -o water_ions.gro -p file.top -pname NA -nname CL -neutral 
```


## Energy minimization (EM) : EM ensured that we have a reasonable starting structure, in terms of geometry and solvent orientation. The purpose of energy minimization is not to find the global or local minimum, but to escape from the force in high gradient and avoid protein collapse.
`nunu.mdp` file
```
; minim.mdp - used as input into grompp to generate em.tpr
; Parameters describing what to do, when to stop and what to save
integrator  = steep         ; Algorithm (steep = steepest descent minimization)
emtol       = 1000.0        ; Stop minimization when the maximum force < 1000.0 kJ/mol/nm
emstep      = 0.01          ; Minimization step size
nsteps      = 50000         ; Maximum number of (minimization) steps to perform

; Parameters describing how to find the neighbors of each atom and how to calculate the interactions
nstlist         = 1         ; Frequency to update the neighbor list and long range forces
cutoff-scheme   = Verlet    ; Buffered neighbor searching
ns_type         = grid      ; Method to determine neighbor list (simple, grid)
coulombtype     = PME       ; Treatment of long range electrostatic interactions
rcoulomb        = 1.0       ; Short-range electrostatic cut-off
rvdw            = 1.0       ; Short-range Van der Waals cut-off
pbc             = xyz       ; Periodic Boundary Conditions in all 3 dimensions
```
```
$ gmx grompp -f minim.mdp -c water_ions.gro -p topol.top -o em.tpr
$ gmx mdrun -v -deffnm em
$ gmx energy -f em.edr -o potential.xvg
$ xmgrace potential.xvg
```

Output:   
- em.log: ASCII-text log file of the EM process
- em.edr: Binary energy file
- em.trr: Binary full-precision trajectory
- em.gro: Energy-minimized structure

### Equilibration
- NVT equilibrium
	nvt.mdp
	```
	title                   = OPLS Lysozyme NVT equilibration 
	define                  = -DPOSRES  ; position restrain the protein
	; Run parameters
	integrator              = md        ; leap-frog integrator
	nsteps                  = 50000     ; 2 * 50000 = 100 ps
	dt                      = 0.002     ; 2 fs
	; Output control
	nstxout                 = 500       ; save coordinates every 1.0 ps
	nstvout                 = 500       ; save velocities every 1.0 ps
	nstenergy               = 500       ; save energies every 1.0 ps
	nstlog                  = 500       ; update log file every 1.0 ps
	; Bond parameters
	continuation            = no        ; first dynamics run
	constraint_algorithm    = lincs     ; holonomic constraints 
	constraints             = h-bonds   ; bonds involving H are constrained
	lincs_iter              = 1         ; accuracy of LINCS
	lincs_order             = 4         ; also related to accuracy
	; Nonbonded settings 
	cutoff-scheme           = Verlet    ; Buffered neighbor searching
	ns_type                 = grid      ; search neighboring grid cells
	nstlist                 = 10        ; 20 fs, largely irrelevant with Verlet
	rcoulomb                = 1.0       ; short-range electrostatic cutoff (in nm)
	rvdw                    = 1.0       ; short-range van der Waals cutoff (in nm)
	DispCorr                = EnerPres  ; account for cut-off vdW scheme
	; Electrostatics
	coulombtype             = PME       ; Particle Mesh Ewald for long-range electrostatics
	pme_order               = 4         ; cubic interpolation
	fourierspacing          = 0.16      ; grid spacing for FFT
	; Temperature coupling is on
	tcoupl                  = V-rescale             ; modified Berendsen thermostat
	tc-grps                 = Protein Non-Protein   ; two coupling groups - more accurate
	tau_t                   = 0.1     0.1           ; time constant, in ps
	ref_t                   = 300     300           ; reference temperature, one for each group, in K
	; Pressure coupling is off
	pcoupl                  = no        ; no pressure coupling in NVT
	; Periodic boundary conditions
	pbc                     = xyz       ; 3-D PBC
	; Velocity generation
	gen_vel                 = yes       ; assign velocities from Maxwell distribution
	gen_temp                = 300       ; temperature for Maxwell distribution
	gen_seed                = -1        ; generate a random seed
	```

	```
	$ gmx grompp -f nvt.mdp -c em_50000.gro -r em_50000.gro -p topol.top -o nvt.tpr
	$ gmx mdrun -deffnm nvt
	$ gmx energy -f nvt.edr -o temperature.xvg
	$ xmgrace potential.xvg
	```
- NPT equilibrium
	npt.mdp
	```
	title                   = OPLS Lysozyme NPT equilibration 
	define                  = -DPOSRES  ; position restrain the protein
	; Run parameters
	integrator              = md        ; leap-frog integrator
	nsteps                  = 50000     ; 2 * 50000 = 100 ps
	dt                      = 0.002     ; 2 fs
	; Output control
	nstxout                 = 500       ; save coordinates every 1.0 ps
	nstvout                 = 500       ; save velocities every 1.0 ps
	nstenergy               = 500       ; save energies every 1.0 ps
	nstlog                  = 500       ; update log file every 1.0 ps
	; Bond parameters
	continuation            = yes       ; Restarting after NVT 
	constraint_algorithm    = lincs     ; holonomic constraints 
	constraints             = h-bonds   ; bonds involving H are constrained
	lincs_iter              = 1         ; accuracy of LINCS
	lincs_order             = 4         ; also related to accuracy
	; Nonbonded settings 
	cutoff-scheme           = Verlet    ; Buffered neighbor searching
	ns_type                 = grid      ; search neighboring grid cells
	nstlist                 = 10        ; 20 fs, largely irrelevant with Verlet scheme
	rcoulomb                = 1.0       ; short-range electrostatic cutoff (in nm)
	rvdw                    = 1.0       ; short-range van der Waals cutoff (in nm)
	DispCorr                = EnerPres  ; account for cut-off vdW scheme
	; Electrostatics
	coulombtype             = PME       ; Particle Mesh Ewald for long-range electrostatics
	pme_order               = 4         ; cubic interpolation
	fourierspacing          = 0.16      ; grid spacing for FFT
	; Temperature coupling is on
	tcoupl                  = V-rescale             ; modified Berendsen thermostat
	tc-grps                 = Protein Non-Protein   ; two coupling groups - more accurate
	tau_t                   = 0.1     0.1           ; time constant, in ps
	ref_t                   = 300     300           ; reference temperature, one for each group, in K
	; Pressure coupling is on
	pcoupl                  = Parrinello-Rahman     ; Pressure coupling on in NPT
	pcoupltype              = isotropic             ; uniform scaling of box vectors
	tau_p                   = 2.0                   ; time constant, in ps
	ref_p                   = 1.0                   ; reference pressure, in bar
	compressibility         = 4.5e-5                ; isothermal compressibility of water, bar^-1
	refcoord_scaling        = com
	; Periodic boundary conditions
	pbc                     = xyz       ; 3-D PBC
	; Velocity generation
	gen_vel                 = no        ; Velocity generation is off 
	```
	
	```
	$ gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -t nvt.cpt -p topol.top -o npt.tpr
			> -t (time check point): the checkpoint file from the NVT equilibration, containing all the necessary state variables to continue our simulation.
			> -c (coordinate file): the final output of the NVT simulation 
	$ gmx mdrun -deffnm npt -nb gpu
	$ gmx energy -f npt.edr -o pressure.xvg
	$ xmgrace pressure.xvg
	```
## Production MD
md.mdp
```
title                   = NARS2
; Run parameters
integrator              = md        ; leap-frog integrator
nsteps                  = 500000    ; 2 * 500000 = 1000 ps (1 ns)
dt                      = 0.002     ; 2 fs
; Output control
nstxout                 = 0         ; suppress bulky .trr file by specifying 
nstvout                 = 0         ; 0 for output frequency of nstxout,
nstfout                 = 0         ; nstvout, and nstfout
nstenergy               = 5000      ; save energies every 10.0 ps
nstlog                  = 5000      ; update log file every 10.0 ps
nstxout-compressed      = 5000      ; save compressed coordinates every 10.0 ps
compressed-x-grps       = System    ; save the whole system
; Bond parameters
continuation            = yes       ; Restarting after NPT 
constraint_algorithm    = lincs     ; holonomic constraints 
constraints             = h-bonds   ; bonds involving H are constrained
lincs_iter              = 1         ; accuracy of LINCS
lincs_order             = 4         ; also related to accuracy
; Neighborsearching
cutoff-scheme           = Verlet    ; Buffered neighbor searching
ns_type                 = grid      ; search neighboring grid cells
nstlist                 = 10        ; 20 fs, largely irrelevant with Verlet scheme
rcoulomb                = 1.0       ; short-range electrostatic cutoff (in nm)
rvdw                    = 1.0       ; short-range van der Waals cutoff (in nm)
; Electrostatics
coulombtype             = PME       ; Particle Mesh Ewald for long-range electrostatics
pme_order               = 4         ; cubic interpolation
fourierspacing          = 0.16      ; grid spacing for FFT
; Temperature coupling is on
tcoupl                  = V-rescale             ; modified Berendsen thermostat
tc-grps                 = Protein Non-Protein   ; two coupling groups - more accurate
tau_t                   = 0.1     0.1           ; time constant, in ps
ref_t                   = 300     300           ; reference temperature, one for each group, in K
; Pressure coupling is on
pcoupl                  = Parrinello-Rahman     ; Pressure coupling on in NPT
pcoupltype              = isotropic             ; uniform scaling of box vectors
tau_p                   = 2.0                   ; time constant, in ps
ref_p                   = 1.0                   ; reference pressure, in bar
compressibility         = 4.5e-5                ; isothermal compressibility of water, bar^-1
; Periodic boundary conditions
pbc                     = xyz       ; 3-D PBC
; Dispersion correction
DispCorr                = EnerPres  ; account for cut-off vdW scheme
; Velocity generation
gen_vel                 = no        ; Velocity generation is off 
```
```
$ gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_1.tpr
$ gmx mdrun -deffnm md_0_1 -nb gpu -gpu_id 0
	> Starts gmx mdrun with 1 GPU with id 0
```
- thread control
Ref: https://manual.gromacs.org/documentation/current/user-guide/mdrun-performance.html
```
Reading file md_0_1.tpr, VERSION 2022.4 (single precision)
Changing nstlist from 10 to 80, rlist from 1 to 1.147

On host gpu1 2 GPUs selected for this run.
Mapping of GPU IDs to the 4 GPU tasks in the 4 ranks on this node:
  PP:0,PP:0,PP:1,PP:1
PP tasks will do (non-perturbed) short-ranged and most bonded interactions on the GPU
PP task will update and constrain coordinates on the CPU
Using 4 MPI threads
Using 6 OpenMP threads per tMPI thread

```
1 GPU selected for this run.
Mapping of GPU IDs to the 2 GPU tasks in the 1 rank on this node:
  PP:0,PME:0
PP tasks will do (non-perturbed) short-ranged interactions on the GPU
PP task will update and constrain coordinates on the CPU
PME tasks will do all aspects on the GPU
Using 1 MPI thread
Using 24 OpenMP threads
```

```
# Practice
```
# To remove water molecules in PDB file
$ grep -v HOH 1aki.pdb > 1AKI_clean.pdb

# To prepare gro file
$ gmx pdb2gmx -f AF-Q96I59-F1-model_v4.pdb -o AF-Q96I59-F1-model_v4.gro -water tip4p

To create box
$ gmx editconf -f AF-Q96I59-F1-model_v4.gro -o AF-Q96I59-F1-model_v4_newbox.gro -c -d 1.0 -bt cubic

To add solvate
$ gmx solvate -cp AF-Q96I59-F1-model_v4_newbox.gro -cs tip4p.gro -o AF-Q96I59-F1-model_v4_solv.gro -p topol.top

To assemble tpr file
$ gmx grompp -f ions.mdp -c AF-Q96I59-F1-model_v4_solv.gro -p topol.top -o ions.tpr

To generate ion
$ gmx genion -s ions.tpr -o AF-Q96I59-F1-model_v4_ions.gro -p topol.top -pname NA -nname CL -neutral

For gromacs preprocessor (Energy Minimization)
$ gmx grompp -f minim.mdp -c AF-Q96I59-F1-model_v4_ions.gro -p topol.top -o em.tpr
$ gmx mdrun -v -deffnm em


step 25: One or more water molecules can not be settled.
Check for bad contacts and/or reduce the timestep if appropriate.

# Equilibration of solvent and ions around the protein
$ gmx grompp -f nvt.mdp -c em_50000.gro -r em_50000.gro -p topol.top -o nvt.tpr
$ gmx mdrun -deffnm nvt
```
```
Dynamic load balancing report:
 DLB got disabled because it was unsuitable to use.
 Average load imbalance: 3.5%.
 The balanceable part of the MD step is 82%, load imbalance is computed from this.
 Part of the total run time spent waiting due to load imbalance: 2.9%.
 Average PME mesh/force load: 0.692
 Part of the total run time spent waiting due to PP/PME imbalance: 3.9 %


               Core t (s)   Wall t (s)        (%)
       Time:   129218.860     4614.959     2800.0
                         1h16:54
                 (ns/day)    (hour/ns)
Performance:        1.872       12.819
```
```
$ gmx energy -f nvt.edr -o temperature.xvg
$ gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -t nvt.cpt -p topol.top -o npt.tpr
$ gmx mdrun -deffnm npt
$ gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_1.tpr
$ gmx mdrun -deffnm md_0_1
```
```
GROMACS:      gmx mdrun, version 2021.4-Ubuntu-2021.4-2
Executable:   /usr/bin/gmx
Data prefix:  /usr
Working dir:  /home/jihun/molecular_dynamics/dars2
Command line:
  gmx mdrun -deffnm md_0_1

Compiled SIMD: SSE4.1, but for this host/run AVX_512 might be better (see
log).
Reading file md_0_1.tpr, VERSION 2021.4-Ubuntu-2021.4-2 (single precision)
Changing nstlist from 10 to 40, rlist from 1 to 1.098

Using 28 MPI threads
Using 1 OpenMP thread per tMPI thread

starting mdrun 'ASPARAGINE--TRNA LIGASE, CYTOPLASMIC in water'
500000 steps,   1000.0 ps.

Writing final coordinates.


Dynamic load balancing report:
 DLB was turned on during the run due to measured imbalance.
 Average load imbalance: 6.7%.
 The balanceable part of the MD step is 73%, load imbalance is computed from this.
 Part of the total run time spent waiting due to load imbalance: 4.9%.
 Steps where the load balancing was limited by -rdd, -rcon and/or -dds: X 0 % Y 0 % Z 0 %
 Average PME mesh/force load: 0.677
 Part of the total run time spent waiting due to PP/PME imbalance: 4.1 %


               Core t (s)   Wall t (s)        (%)
       Time:  1508233.468    53865.481     2800.0
                         14h57:45
                 (ns/day)    (hour/ns)
Performance:        1.604       14.963

GROMACS reminds you: "Your Shopping Techniques are Amazing" (Gogol Bordello)

```
```
$ gmx trjconv -s md_0_1.tpr  -f md_0_1.xtc -o md_0_1.noPBC.xtc -pbc mol -center
             :-) GROMACS - gmx trjconv, 2021.4-Ubuntu-2021.4-2 (-:

                            GROMACS is written by:
     Andrey Alekseenko              Emile Apol              Rossen Apostolov     
         Paul Bauer           Herman J.C. Berendsen           Par Bjelkmar       
       Christian Blau           Viacheslav Bolnykh             Kevin Boyd        
     Aldert van Buuren           Rudi van Drunen             Anton Feenstra      
    Gilles Gouaillardet             Alan Gray               Gerrit Groenhof      
       Anca Hamuraru            Vincent Hindriksen          M. Eric Irrgang      
      Aleksei Iupinov           Christoph Junghans             Joe Jordan        
    Dimitrios Karkoulis            Peter Kasson                Jiri Kraus        
      Carsten Kutzner              Per Larsson              Justin A. Lemkul     
       Viveca Lindahl            Magnus Lundborg             Erik Marklund       
        Pascal Merz             Pieter Meulenhoff            Teemu Murtola       
        Szilard Pall               Sander Pronk              Roland Schulz       
       Michael Shirts            Alexey Shvetsov             Alfons Sijbers      
       Peter Tieleman              Jon Vincent              Teemu Virolainen     
     Christian Wennberg            Maarten Wolf              Artem Zhmurov       
                           and the project leaders:
        Mark Abraham, Berk Hess, Erik Lindahl, and David van der Spoel

Copyright (c) 1991-2000, University of Groningen, The Netherlands.
Copyright (c) 2001-2019, The GROMACS development team at
Uppsala University, Stockholm University and
the Royal Institute of Technology, Sweden.
check out http://www.gromacs.org for more information.

GROMACS is free software; you can redistribute it and/or modify it
under the terms of the GNU Lesser General Public License
as published by the Free Software Foundation; either version 2.1
of the License, or (at your option) any later version.

GROMACS:      gmx trjconv, version 2021.4-Ubuntu-2021.4-2
Executable:   /usr/bin/gmx
Data prefix:  /usr
Working dir:  /home/jihun/molecular_dynamics/dars2
Command line:
  gmx trjconv -s md_0_1.tpr -f md_0_1.xtc -o md_0_1.noPBC.xtc -pbc mol -center

Note that major changes are planned in future for trjconv, to improve usability and utility.
Will write xtc: Compressed trajectory (portable xdr format): xtc
Reading file md_0_1.tpr, VERSION 2021.4-Ubuntu-2021.4-2 (single precision)
Reading file md_0_1.tpr, VERSION 2021.4-Ubuntu-2021.4-2 (single precision)
Select group for centering
Group     0 (         System) has 974523 elements
Group     1 (        Protein) has 27117 elements
Group     2 (      Protein-H) has 13690 elements
Group     3 (        C-alpha) has  1692 elements
Group     4 (       Backbone) has  5076 elements
Group     5 (      MainChain) has  6764 elements
Group     6 (   MainChain+Cb) has  8336 elements
Group     7 (    MainChain+H) has  8368 elements
Group     8 (      SideChain) has 18749 elements
Group     9 (    SideChain-H) has  6926 elements
Group    10 (    Prot-Masses) has 27117 elements
Group    11 (    non-Protein) has 947406 elements
Group    12 (          Water) has 947364 elements
Group    13 (            SOL) has 947364 elements
Group    14 (      non-Water) has 27159 elements
Group    15 (            Ion) has    42 elements
Group    16 (             NA) has    42 elements
Group    17 ( Water_and_ions) has 947406 elements
Select a group: 1
Selected 1: 'Protein'
Select group for output
Group     0 (         System) has 974523 elements
Group     1 (        Protein) has 27117 elements
Group     2 (      Protein-H) has 13690 elements
Group     3 (        C-alpha) has  1692 elements
Group     4 (       Backbone) has  5076 elements
Group     5 (      MainChain) has  6764 elements
Group     6 (   MainChain+Cb) has  8336 elements
Group     7 (    MainChain+H) has  8368 elements
Group     8 (      SideChain) has 18749 elements
Group     9 (    SideChain-H) has  6926 elements
Group    10 (    Prot-Masses) has 27117 elements
Group    11 (    non-Protein) has 947406 elements
Group    12 (          Water) has 947364 elements
Group    13 (            SOL) has 947364 elements
Group    14 (      non-Water) has 27159 elements
Group    15 (            Ion) has    42 elements
Group    16 (             NA) has    42 elements
Group    17 ( Water_and_ions) has 947406 elements
Select a group: 0
Selected 0: 'System'
Reading frame       0 time    0.000   
Precision of md_0_1.xtc is 0.001 (nm)
Using output precision of 0.001 (nm)
Last frame        100 time 1000.000    ->  frame    100 time 1000.000      


GROMACS reminds you: "C++ is tricky. You can do everything. You can even make every mistake." (Nicolai Josuttis, CppCon2017)
```

```
xmgrace [file_name].xvg
```

# Analysis
- Solvent Accessible Surface Area (SASA)
Ref: https://www.compchems.com/how-to-compute-the-solvent-accessible-surface-areas-sasa-with-gromacs/#solvent-accessible-surface-area-sasa
- 