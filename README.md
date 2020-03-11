# SWAXS
Calculator for Small- and Wide-Angle X-ray Scattering profile.


## Notes

* This SWAXS program is written and built in Julia 1.5.0-DEV.
* SWAXS has 4 different computing modes for input files:

  1. Single `.pdb` file
  2. A pair of `.pdb` files: `solute.pdb` and `solvent.pdb` (used for buffer subtraction and excluded volume)
  3. 3D electron density map: `.mrc` or `.map`. Buffer subtraction is implemented.
  4. 3D shape with dummy voxels: `.binvox`. Buffer subtraction is implemented.

* By default, SWAXS uses 8 cores of the CPU and can be scaled for larger workstation and cluster.


## Version

Run `.\\bin\\SWAXS --version`

```
>.\\bin\\SWAXS --version
0.2.0
```

## Usage

Run `.\\bin\\SWAXS --help`

```
>.\\bin\\SWAXS --help
usage: SWAXS [-D DENSITY] [-P PDB] [-T SOLUTE] [-V SOLVENT]
             [-B BINVOX] [-s SOLVENT_DENSITY] [-v VOXEL_DENSITY]
             [-c DENSITY_CUTOFF] [-J J] [-n NPR] [-o OUTPUT]
             [--version] [-h] qmin qspacing qmax

Small and Wide-Angle X-ray Scattering calculator for single atomic
coordinates: .pdb, solute-solvent pair PDBs, electron density: .mrc or
.map and voxelized 3D shape: .binvox. SWAXS uses Debye formula,
orientational average and Park et al. to account for buffer
subtraction and solvent shell.

positional arguments:
  qmin                  Starting q value, in (1/A) (type: Float64)
  qspacing              q grid spacing, in (1/A) (type: Float64)
  qmax                  Ending q value, in (1/A) (type: Float64)

optional arguments:
  -D, --density DENSITY
                        Electron density map file: .mrc or .map
  -P, --pdb PDB         Single file containing atomic coordinates:
                        .pdb
  -T, --solute SOLUTE   The solute.pdb file
  -V, --solvent SOLVENT
                        The solvent.pdb file
  -B, --binvox BINVOX   The shape file of dummy voxels: .binvox
  -s, --solvent_density SOLVENT_DENSITY
                        The bulk solvent electron density in e/A^3
                        (type: Float64, default: 0.335)
  -v, --voxel_density VOXEL_DENSITY
                        The averaged electron density on dummy voxels
                        in e/A^3 (type: Float64, default: 0.5)
  -c, --density_cutoff DENSITY_CUTOFF
                        The electron density cutoff to exclude
                        near-zero voxels (type: Float64, default:
                        0.001)
  -J, --J J             Number of orientations to be averaged (type:
                        Int64, default: 1200)
  -n, --npr NPR         Number of parallel workers for computation
                        (type: Int64, default: 8)
  -o, --output OUTPUT   Output file prefix for saving the .dat file
                        (default: "output")
  --version             show version information and exit
  -h, --help            show this help message and exit

==============================================
Last updated: 03/10/20, Ithaca, NY.
Copyright (c) Yen-Lin Chen, 2018 - 2020
Email: yc2253@cornell.edu
==============================================
```


### 1. Calculating from single `.pdb` file containing atomic-coordinates: `rna.pdb`

   The `rna.pdb` file contains about 770 atoms.

```
>.\\bin\\SWAXS --pdb rna.pdb -o test 0.0 0.01 1.5
[ Info: --- SWAXS: Setting up parallel workers ...
[ Info: --- SWAXS: Please wait ...
[ Info: --- SWAXS: Computing SWAXS (J=1200) using single PDB file: rna.pdb.
[ Info: --- SWAXS: SWAXS program completed successfully: elapsed time = 10.169479699 seconds with 8 cores.
[ Info: --- SWAXS: Removing parallel workers ...
```

   And the swaxs profile from `q = 0.0` to `q = 1.5` with spacing `0.01` is saved as `test.dat`.

rna.pdb                    |  SWAXS profile
:-------------------------:|:-------------------------:
![](rna.png)               |  <img src="rna_swaxs.png" alt="drawing" width="800"/>


### 2. Calculating from a pair of `.pdb` files to account for buffer subtraction: `solute.pdb` and `solvent.pdb`

   Each of the `.pdb` file contains about 9000 atoms.

```
>.\\bin\\SWAXS --solute solute.pdb --solvent solvent.pdb -o test 0.0 0.01 1.5
[ Info: --- SWAXS: Setting up parallel workers ...
[ Info: --- SWAXS: Please wait ...
[ Info: --- SWAXS: Computing SWAXS (J=1200) using solute: solute.pdb and solvent: solvent.pdb.
[ Info: --- SWAXS: SWAXS program completed successfully: elapsed time = 38.712813001 seconds with 8 cores.
[ Info: --- SWAXS: Removing parallel workers ...
```

  And the swaxs profile from `q = 0.0` to `q = 1.5` with spacing `0.01` is saved as `test.dat`.

solute.pdb                 |  solvent.pdb              |  SWAXS profile
:-------------------------:|:-------------------------:|:-------------------------:
![](solute.png)            |  ![](solvent.png)         |  <img src="solute_swaxs.png" alt="drawing" width="800"/>

Note that this is just for demonstration so the solvent/ion coordinates are exactly the same (i.e. the ion/solvent shells are still shaped by the molecule). However, it's suggested to use another frame of randomized solvent for a more precise buffer subtraction.



### 3. Calculating from CCP4 3D electron density map `.mrc` or `.map` files: `den.mrc`

  The `shape.mrc` contains 96x96x96 volumetric data and is considered to be the Excessive Density
  Run

```
>.\\bin\\SWAXS --density shape.mrc -o test 0.0 0.01 1.5
[ Info: --- SWAXS: Setting up parallel workers ...
[ Info: --- SWAXS: Please wait ...
[ Info: --- SWAXS: Computing SWAXS (J=1200) using electron density file: shape.mrc, with sden=0.335 cutoff=0.001.
[ Info: --- SWAXS: SWAXS program completed successfully: elapsed time = 18.5273815 seconds with 8 cores.
[ Info: --- SWAXS: Removing parallel workers ...
```

  And the swaxs profile from `q = 0.0` to `q = 1.5` with spacing `0.01` is saved as `test.dat`.

shape.mrc                  |  SWAXS profile
:-------------------------:|:-------------------------:
![](shape.png)             |  <img src="shape_swaxs.png" alt="drawing" width="800"/>

  The `shape.mrc` is just an envelope, so it doesn't have much feature in the WAXS regime.


shape2.mrc                 |  SWAXS profile
:-------------------------:|:-------------------------:
![](shape2.png)            |  <img src="shape2_swaxs.png" alt="drawing" width="800"/>

  The `shape2.mrc` has more features, reflected in the wide-angle regime.

  Note that this is just a demonstration and in this case, it's advised to have well-defined molecular support (not yet integrated in this SWAXS version.).


### 4. Calculating from 3D shape with dummy voxels: `rabbit.binvox`

  The `rabbit.binvox` contains 51x51x51 volumetric data with a grid size of `2A` and is considered to be the 3D cat shape with Excessive Density `0.5 e/A^3`.

```
>.\\bin\\SWAXS --binvox rabbit.binvox -o test 0.0 0.01 1.5
[ Info: --- SWAXS: Setting up parallel workers ...
[ Info: --- SWAXS: Please wait ...
[ Info: --- SWAXS: Computing SWAXS (J=1200) using shape file: rabbit.binvox, with voxel_density=0.5, sden=0.335.
[ Info: --- SWAXS: SWAXS program completed successfully: elapsed time = 32.7341433 seconds with 8 cores.
[ Info: --- SWAXS: Removing parallel workers ...
```

  And the swaxs profile from `q = 0.0` to `q = 1.5` with spacing `0.01` is saved as `test.dat`.

rabbit.binvox              |  SWAXS profile
:-------------------------:|:-------------------------:
![](rabbit.png)            |  <img src="rabbit_swaxs.png" alt="drawing" width="800"/>

  This uniform 3D voxelized shape is very similar to the `shape.mrc` case since the electron density is uniform within the rabbit. So SWAXS profile at wide-angle regime actually reveals finer periodicity from electron-denser structures.


## More Notes

1. The options `--pdb`, `--density`, `--binvox` and `--solute --solvent` cannot not be specified at the same time. Otherwise, error will be thrown.
2. If high-throughput computation is required, one should bypassing the command line because it sets up parallel workers every time. To avoid that, set up your parallel workers using `SWAXS.jl`. Send request to Yen (yc2253@cornell.edu).
3. If your structures contain too many atoms or voxels, `GPUSWAXS` might be suitable.
4. The atom-name processor might be a little bit buggy, I will try to fix some of the parsing issues.
5. Not sure if I will maintain this or not in the future ...

## References

1. For the formulation and theory: Park et al. J Chem Phys. 2009 Apr 7;130(13):134114. [Link](https://doi.org/10.1063/1.3099611)
2. For the application: Chen et al. J. Phys. Chem. B 2019, 123, 46, 9773-9785. [Link](https://doi.org/10.1021/acs.jpcb.9b07502)

And many fun stuff will follow. : )

## TODO

1. Molecular support
2. Compatibility with the GROMACS output files
