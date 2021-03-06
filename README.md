# Molecular dynamics simulation tutorial for Structural Biology 

The aim of this tutorial is to learn molecular dynamics simulation. Tryptophan cage (or Trp-cage, 1L2Y) is
an artificial miniprotein (20 residues) derived from venom of *Gila monster*. Despite its small size it folds
into a compact structure. This folding is super-fast compared to regular proteins. This make Trp-cage a
favorite model miniprotein for folding simulations. Here we will simulate its dynamics:

1. Clone this repository by:
```
git clone https://github.com/spiwokv/md4structbioinf.git
```
and got to the directory `md4structbioinf`.

2. The directory contains the file `1L2Ym1.pdb`. This contains the structure of the first model in the PDB file.
This structure was measured by NMR. Structures measured by NMR and typically stored in PDB as several models.
Each model represents the results of one independent process of determination of structure from NOE and other
restrains. For the purpose of this tutorial the first structure was retained and other structures were deleted.
PDB format is a text format, so it is possible to delete structures manually in a text editor.

This structure must be converted from PDB to Gromacs formate. At the same time topology is generated
and hydrogens are added. This can be done by command:
```
gmx pdb2gmx -f 1L2Ym1.pdb -o protein -p protein -ignh
```
You will be asked for force field and water model. Chose Amber99SB-ILDN and TIP3P. Inspect files `protein.gro`
and `protein.top`.

3. Generate the simulation box and center the protein in it. This can be done by command:
```
gmx editconf -f protein -o box -c -box 5 5 5
```
This will generate a file `box.gro` with coordinates from `protein.gro` moved so that the box size is 5x5x5 nm
and the molecule is centered in the box.

4. Fill the box with water by typing:
```
gmx solvate -cs -cp box -o solvated -p protein
```

5. Neutralize the box. Try to determine the net charge of the molecule. It is necessary to add either Na+ or Cl- 
to compensate the charge. In Gromacs there is a command `gmx genion`, but we can do without it. Instead, copy the
structure of solvated system:
```
cp solvated.gro neutral.gro
```
If the number of ions to be added is *n*, edit `neutral.gro` and i. reduce the number of atoms by 2x*n* 
(you replace H2O to Na+ or Cl-), ii. move *n* randomly selected water molecules at the end of the file
and delete hydrogens from them, and iii. change atom and residue type to those of Na+ or Cl-.

Next, edit `protein.top` and reduce the number of water molecules at the bottom of the file by *n* and add 
*n* ions to the list.

6. Minimize energy of the system. Make a new directory called `em`:
```
mkdir em
```
and copy there `protein.top`, `neutral.gro` and `em.mdp`. Inspect the file `em.mdp` to make sure all options
are OK. To run energy minimization type the pair of commands (may take few minutes):
```
gmx grompp -f em.mdp -c neutral.gro -p protein.top -o em1
gmx mdrun -s em1 -o em1 -g em1 -c after_em1 -e em1 -v
```

7. In order to settle water around protein atoms, make a new subdirectory in `md4structbioinf`, copy there
all necessary files and run restrained simulation:
```
gmx grompp -f nvt.mdp -c after_em1.gro -r after_em1.gro -p protein.top -o nvt.tpr
gmx mdrun -s nvt -o nvt -e nvt -g nvt -c after_nvt -v
```

8. In order to equilibrate pressure, make a new subdirectory in `md4structbioinf`, copy there
all necessary files and run restrained simulation:
```
gmx grompp -f npt.mdp -c after_nvt.gro -r after_nvt.gro -t state.cpt -p protein.top -o npt.tpr
gmx mdrun -s npt -o npt -e npt -g npt -c after_npt -v
```

9. Run production (unrestrained) simulation. Make a new subdirectory in `md4structbioinf`,
copy there all necessary files and run the simulation:
```
gmx grompp -f prod.mdp -c after_npt.gro -t state.cpt -p protein.top -o prod.tpr
gmx mdrun -s prod -o prod -e prod -g prod -c after_prod -v
```

