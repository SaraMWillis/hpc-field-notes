# LAMMPS

## Sample Input File

```
units lj
atom_style atomic
lattice fcc 0.8442
region box block 0 5 0 5 0 5
create_box 1 box
create_atoms 1 box
mass 1 1.0

pair_style lj/cut 2.5
pair_coeff 1 1 1.0 1.0 2.5

velocity all create 1.44 87287
fix 1 all nve

thermo 10
run 100
```