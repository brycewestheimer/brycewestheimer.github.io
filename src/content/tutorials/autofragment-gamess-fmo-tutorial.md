---
title: "Generating GAMESS FMO Inputs with autofragment"
description: "A hands-on tutorial for partitioning molecular systems and generating GAMESS FMO input files using the autofragment Python library."
date: 2026-01-02
level: "advanced"
tags: ["autofragment", "GAMESS", "FMO", "quantum chemistry", "tutorial", "Python"]
---


Fragment molecular orbital (FMO) calculations require the input system to be pre-partitioned into fragments, with each atom assigned to exactly one fragment. Each fragment needs a charge and spin multiplicity. For a water cluster this is trivial. For a 300-residue protein with crystallographic waters, ions, and ligands, doing it by hand is tedious and error-prone. autofragment automates this entire pipeline: read a structure file, partition it into chemically sensible fragments, and write a ready-to-run GAMESS input file.

This tutorial covers two workflows: water clusters (the simple case, using XYZ files) and proteins from mmCIF files (the realistic case, requiring the `[bio]` extra).

## Installation

```bash
pip install autofragment           # core: water clusters, XYZ, clustering
pip install "autofragment[bio]"    # adds mmCIF parsing (gemmi) for proteins
pip install "autofragment[all]"    # everything: bio, graph, matsci, balanced clustering
```

Verify what is available:

```python
import autofragment as af
print(af.list_features())
# e.g. ['balanced', 'bio', 'graph']
```

The `list_features()` function checks which optional dependencies are installed and reports the corresponding feature flags. The `has_feature("bio")` function returns a boolean for a specific feature.

## Part A: Water Clusters

### Reading the Input

```python
from autofragment import io

system = io.read_xyz("water16.xyz", atoms_per_molecule=3)
```

`read_xyz` returns a `ChemicalSystem` containing all atoms and metadata. The `atoms_per_molecule=3` argument (the default) tells the reader to group every three atoms as one water molecule. If your XYZ file contains molecules with a different number of atoms (for example, methanol with 6 atoms), set this accordingly. Setting `atoms_per_molecule=None` treats all atoms as a single molecule.

```python
print(f"{system.n_atoms} atoms, {len(system.to_molecules())} molecules")
# 48 atoms, 16 molecules
```

The `ChemicalSystem` stores atoms as a flat list. Each `Atom` has a `symbol` (element) and `coords` (a NumPy array of shape `(3,)`). The `to_molecules()` method splits the atom list back into groups based on the `atoms_per_molecule` metadata stored during parsing.

### Flat Partitioning: The One-Liner

For quick work, `partition_xyz` reads and partitions in a single call:

```python
import autofragment as af

tree = af.partition_xyz("water16.xyz", n_fragments=4)
print(tree.n_primary)  # 4
```

This uses k-means clustering on molecular centroids by default. The return value is a `FragmentTree`.

### Flat Partitioning: Full Control

For more control over the clustering algorithm, create a `MolecularPartitioner` directly:

```python
partitioner = af.MolecularPartitioner(
    n_fragments=4,
    method="kmeans",       # also: agglomerative, spectral, gmm, birch, kmeans_constrained
    random_state=42,
)
tree = partitioner.partition(system, source_file="water16.xyz")
```

The `method` parameter selects the clustering algorithm from scikit-learn. All methods operate on the centroids of the individual molecules (not on raw atom coordinates). The `random_state` parameter ensures reproducible results for stochastic methods like k-means and GMM.

### Inspecting the FragmentTree

`partition()` returns a `FragmentTree`. Its `.fragments` attribute is a list of `Fragment` objects. Each `Fragment` holds the atom symbols, flat geometry array, molecular charge, and spin multiplicity for one group of molecules.

```python
for frag in tree.fragments:
    print(f"{frag.id}: {frag.n_atoms} atoms, charge={frag.molecular_charge}")
# F1: 12 atoms, charge=0
# F2: 12 atoms, charge=0
# F3: 12 atoms, charge=0
# F4: 12 atoms, charge=0
```

Fragment IDs in flat mode follow the pattern `F1`, `F2`, etc. The `n_atoms` property counts atoms recursively (relevant for hierarchical fragments; for leaf fragments it is just `len(symbols)`).

Fragment coordinates are stored as a flat list internally. The `get_coords()` method returns them as a reshaped NumPy array:

```python
coords = tree.fragments[0].get_coords()  # shape (n_atoms, 3)
print(coords.shape)
# (12, 3)
```

You can also access `frag.symbols` for the element list (e.g., `['O', 'H', 'H', 'O', 'H', 'H', ...]`).

### Writing a GAMESS FMO Input

```python
from autofragment.io import write_gamess_fmo

write_gamess_fmo(
    tree.fragments,
    "water16_fmo.inp",
    basis="6-31G*",
    method="RHF",
    fmo_level=2,
    nbody=2,
    memory=500,
    title="Water16 FMO2/RHF/6-31G*",
)
```

This writes a complete GAMESS input file. Here is an annotated excerpt of what gets generated:

```
 $CONTRL SCFTYP=RHF RUNTYP=ENERGY
         COORD=UNIQUE LOCAL=NONE
 $END
 $SYSTEM MWORDS=500 $END
 $BASIS N31 NGAUSS=6 NDFUNC=1 $END
 $FMO
    NFRAG=4 NBODY=2
    ICHARG(1)=0,0,0,0
    MULT(1)=1,1,1,1
    INDAT(1)=1,1,1,1,1,1,1,1,1,1,
             1,1,2,2,2,2,2,2,2,2,
             2,2,2,2,3,3,3,3,3,3,
             3,3,3,3,3,3,4,4,4,4,
             4,4,4,4,4,4,4,4
 $END
 $GDDI NGROUP=1 $END
 $DATA
Water16 FMO2/RHF/6-31G*
C1
 O    8.0   0.0000000000   0.0000000000   0.0000000000
 H    1.0   ...
 ...
 $END
```

`$CONTRL` sets the SCF type and run type. `$BASIS` specifies the basis set. `$FMO` is the fragment specification block: `NFRAG` is the number of fragments, `NBODY` controls the many-body expansion level (2 for FMO2, 3 for FMO3), `ICHARG` and `MULT` set charge and multiplicity per fragment, and `INDAT` maps each atom (in the order they appear in `$DATA`) to its fragment index. `$GDDI` controls the parallel group decomposition. `$DATA` contains the atom coordinates with element symbols and nuclear charges.

The `basis` parameter accepts standard basis set names. autofragment maps them to GAMESS-specific keywords internally. For example, `"6-31G*"` becomes `N31 NGAUSS=6 NDFUNC=1`, `"cc-pVDZ"` becomes `CCD`, and `"aug-cc-pVTZ"` becomes `ACCT`. The full mapping includes STO-3G, 3-21G, 6-31G variants, 6-311G variants, and the Dunning correlation-consistent family.

You can pass additional GAMESS keywords via the `extra_contrl` and `extra_fmo` dictionaries:

```python
write_gamess_fmo(
    tree.fragments,
    "water16_fmo_mp2.inp",
    basis="cc-pVDZ",
    method="RHF",
    fmo_level=2,
    nbody=2,
    extra_contrl={"MPLEVL": "2"},
    title="Water16 FMO2-MP2/cc-pVDZ",
)
```

This adds `MPLEVL=2` to the `$CONTRL` group, enabling MP2 correlation on top of the RHF reference within FMO.

### Serialization

Save and reload a `FragmentTree` as JSON for archiving or passing to downstream tools:

```python
tree.to_json("water16_fragments.json")

reloaded = af.FragmentTree.from_json("water16_fragments.json")
print(reloaded.n_primary)  # 4
```

The JSON contains the full fragment definitions (symbols, geometry, charges, multiplicities) plus metadata about the source file and partitioning algorithm used. You can inspect this file directly; it is human-readable. This is useful for reproducibility and for generating inputs for multiple quantum chemistry programs from the same partition without re-running the clustering.

### Tiered (Hierarchical) Partitioning

For larger systems, a hierarchical partition groups molecules into primary fragments, each subdivided into secondary fragments. This mirrors the physical idea behind FMO: nearby fragments interact more strongly and benefit from higher body-order corrections, while distant fragments can be treated at lower body order.

```python
tree = af.partition_xyz(
    "water64.xyz",
    tiers=2,
    n_primary=4,
    n_secondary=4,
)
print(tree._is_hierarchical)  # True
print(tree.n_primary)          # 4
print(tree.n_fragments)        # 20 (4 primary + 16 secondary)
```

Primary fragments have IDs like `PF1`, `PF2`, `PF3`, `PF4`. Their children (secondary fragments) have IDs like `PF1_SF1`, `PF1_SF2`, `PF1_SF3`, `PF1_SF4`. Primary fragments are not leaf nodes; they contain no atoms directly. The actual atom data lives in the secondary (leaf) fragments.

```python
for pf in tree.fragments:
    print(f"{pf.id}: {len(pf.fragments)} sub-fragments, {pf.n_atoms} atoms total")
    for sf in pf.fragments:
        print(f"  {sf.id}: {sf.n_atoms} atoms (leaf={sf.is_leaf})")
# PF1: 4 sub-fragments, 48 atoms total
#   PF1_SF1: 12 atoms (leaf=True)
#   PF1_SF2: 12 atoms (leaf=True)
#   ...
```

To generate a GAMESS FMO input from a hierarchical tree, extract the leaf fragments. GAMESS does not have a native concept of hierarchical fragmentation, so you flatten the tree to its leaves:

```python
leaves = [sf for pf in tree.fragments for sf in pf.fragments]
write_gamess_fmo(leaves, "water64_fmo.inp", basis="6-31G*")
```

Three-tier partitioning works the same way, with `tiers=3` and an additional `n_tertiary` parameter. Tertiary fragment IDs follow the pattern `PF1_SF2_TF3`.

```python
tree = af.partition_xyz(
    "water256.xyz",
    tiers=3,
    n_primary=4,
    n_secondary=4,
    n_tertiary=4,
)
# Total leaf fragments: 4 * 4 * 4 = 64
```

### Balanced Partitioning and Seeding

For load-balanced parallel FMO calculations, all fragments should contain the same number of molecules. The `kmeans_constrained` method enforces exactly equal cluster sizes. This requires the `[balanced]` extra (or `[all]`):

```bash
pip install "autofragment[balanced]"   # installs k-means-constrained
```

```python
partitioner = af.MolecularPartitioner(
    n_fragments=4,
    method="kmeans_constrained",
    # strict_balanced defaults to True for kmeans_constrained
)
tree = partitioner.partition(system)
# All four fragments contain exactly 4 molecules (for 16-molecule system)
```

K-means results depend on initialization. The default (k-means++) is stochastic and can produce different partitions on different runs (controlled by `random_state`). autofragment provides four deterministic seeding strategies that place initial cluster centers based on the geometry of the system:

- `"pca"`: places seeds along the first principal component axis, evenly spaced. Good for elongated systems.
- `"axis"`: places seeds along a Cartesian axis (x, y, or z). Simple and predictable.
- `"halfplane"`: recursively bisects the system with hyperplanes through the centroid. Produces spatially contiguous fragments.
- `"radial"`: places seeds at equal angular intervals around the system centroid. Good for roughly spherical clusters.

```python
tree = af.partition_xyz(
    "water64.xyz",
    n_fragments=8,
    init_strategy="pca",
)
```

For tiered partitioning, different seeding strategies can be applied at each tier. For example, use PCA at the primary level (to split along the longest axis) and radial at the secondary level (to subdivide each primary region into angular sectors):

```python
partitioner = af.MolecularPartitioner(
    tiers=2,
    n_primary=4,
    n_secondary=4,
    method="kmeans",
    init_strategy_primary="pca",
    init_strategy_secondary="radial",
)
```

### CLI Equivalents

Everything above can be done from the command line. The `single` subcommand handles individual XYZ files:

```bash
# Flat partitioning with 4 fragments
autofragment single --input water16.xyz --n-fragments 4 --method kmeans

# Tiered partitioning
autofragment single --input water64.xyz --tiers 2 --n-primary 4 --n-secondary 4

# With seeding strategy
autofragment single --input water64.xyz --n-fragments 8 --init-strategy pca

# Balanced clustering
autofragment single --input water16.xyz --n-fragments 4 --method kmeans_constrained
```

These commands write a JSON file (by default, the input filename with a `.json` extension) containing the `FragmentTree` structure. You can specify a different output path with `--output`.

## Part B: Proteins from mmCIF

### Setup

Biological partitioning requires the `[bio]` extra, which pulls in `gemmi` for mmCIF parsing:

```bash
pip install "autofragment[bio]"
```

Verify:

```python
import autofragment as af
print(af.has_feature("bio"))  # True
```

If this returns `False`, check that gemmi installed correctly. Some platforms may require building gemmi from source.

### BioPartitioner

`BioPartitioner` reads an mmCIF file and partitions the structure at residue level, with waters grouped into clusters:

```python
from autofragment import BioPartitioner

bio = BioPartitioner(
    ph=7.4,                    # pH for charge calculation
    infer_bonds=True,          # detect peptide, disulfide, ligand bonds
    water_clusters=3,          # number of water clusters per chain
    water_cluster_method="kmeans_constrained",  # default for balanced water groups
)
tree = bio.partition_file("protein.cif")
```

The returned `FragmentTree` contains one `Fragment` per amino acid residue (for polymer chains), one per ligand, and waters grouped into the specified number of clusters. The `partition_file` method handles mmCIF parsing internally via gemmi.

### Residue-Level Fragmentation

`BioPartitioner` groups polymer residues by chain and secondary structure. Fragment IDs encode this context as a prefix:

```python
for frag in tree.fragments[:6]:
    print(f"{frag.id}: {frag.n_atoms} atoms, charge={frag.molecular_charge}")
# CHAIN_A_HELIX1|A:ALA:1: 10 atoms, charge=0
# CHAIN_A_HELIX1|A:ARG:2: 24 atoms, charge=1
# CHAIN_A_HELIX1|A:ASP:3: 12 atoms, charge=-1
# CHAIN_A_COIL1|A:GLY:4: 7 atoms, charge=0
# CHAIN_A_LIG|A:ATP:501: 47 atoms, charge=-4
# CHAIN_A_WCL1|A:HOH:301: 3 atoms, charge=0
```

The prefix before the pipe encodes the chain ID and secondary structure segment (`HELIX1`, `COIL1`, `STRAND2`). The part after the pipe is `chain:residue_name:residue_number`. Water clusters get `WCL` prefixes. Ligands get `LIG` prefixes.

If `water_clusters` is set to `None`, autofragment auto-selects the number of clusters as roughly $\sqrt{n_\text{waters}}$, rounded to the nearest integer. The clustering uses molecular centroids (the mean position of each water's atoms) and defaults to `kmeans_constrained` for equal-sized clusters.

### pH-Dependent Charges

Fragment charges are computed from side-chain pKa values using the Henderson-Hasselbalch equation. The `get_sidechain_charge` function computes the fractional charge for ionizable residues at a given pH:

```python
from autofragment.chemistry.ph import get_sidechain_charge

print(get_sidechain_charge("ASP", 7.4))  # -1.0 (fully deprotonated, pKa ~3.65)
print(get_sidechain_charge("GLU", 7.4))  # -1.0 (fully deprotonated, pKa ~4.25)
print(get_sidechain_charge("LYS", 7.4))  # ~1.0 (fully protonated, pKa ~10.5)
print(get_sidechain_charge("ARG", 7.4))  # ~1.0 (fully protonated, pKa ~12.5)
print(get_sidechain_charge("HIS", 7.4))  # ~0.09 (mostly neutral, pKa ~6.0)
print(get_sidechain_charge("ALA", 7.4))  # 0.0 (no ionizable side chain)
```

When `BioPartitioner` assigns charges to fragments, it rounds these fractional values to integers, since GAMESS FMO requires integer fragment charges. At physiological pH (7.4), this means ASP and GLU get charge -1, LYS and ARG get +1, and HIS gets 0. At lower pH values (say 5.0), histidine would be protonated and assigned +1.

### Inspecting Interfragment Bonds

`BioPartitioner` infers three types of interfragment covalent bonds: peptide bonds (C-N between consecutive polymer residues), disulfide bonds (SG-SG between cysteine pairs within 2.2 angstroms), and ligand-protein bonds (any covalent bond between a ligand atom and a polymer atom, detected by covalent radii):

```python
print(f"{len(tree.interfragment_bonds)} interfragment bonds")

for bond in tree.interfragment_bonds[:3]:
    btype = bond["metadata"]["type"]
    print(f"  {bond['fragment1_id']} -- {bond['fragment2_id']} ({btype})")
# CHAIN_A_HELIX1|A:ALA:1 -- CHAIN_A_HELIX1|A:ARG:2 (peptide)
# CHAIN_A_HELIX1|A:ARG:2 -- CHAIN_A_HELIX1|A:ASP:3 (peptide)
# CHAIN_A_STRAND1|A:CYS:45 -- CHAIN_A_HELIX3|A:CYS:102 (disulfide)
```

These bonds are stored in `tree.interfragment_bonds` as dictionaries with `fragment1_id`, `atom1_index`, `fragment2_id`, `atom2_index`, `bond_order`, and `metadata` fields. They record where covalent bonds were broken by the fragmentation. This information is relevant for setting up link atoms or bond detachment atoms (BDA) in the actual FMO calculation, and for understanding the connectivity between fragments.

You can disable bond inference with `infer_bonds=False` if you only need the fragment definitions.

### Writing GAMESS FMO for Proteins

The same `write_gamess_fmo` function works for biological fragments:

```python
from autofragment.io import write_gamess_fmo

write_gamess_fmo(
    tree.fragments,
    "protein_fmo.inp",
    basis="6-31G*",
    method="RHF",
    fmo_level=2,
    nbody=2,
    memory=2000,
    title="Protein FMO2/RHF/6-31G*",
)
```

The key difference from water clusters is that fragments now carry non-zero charges. The generated `ICHARG` line reflects the pH-dependent charges computed by `BioPartitioner`:

```
ICHARG(1)=0,1,-1,0,0,-1,1,0,0,0,...
```

For a 200-residue protein, the `$FMO` block will have `NFRAG=200` (approximately, depending on water clusters and ligands), and the `INDAT` array will map each of the thousands of atoms to its fragment. autofragment handles the formatting, line-wrapping (10 values per line), and atom ordering automatically.

For large proteins, you will likely want to increase `memory` beyond the default 500 MW, and consider using a smaller basis set for initial testing.

### CLI Bio Command

The `bio` subcommand handles mmCIF files:

```bash
autofragment bio --input protein.cif --output protein.json --ph 7.4 --water-clusters 3
```

Additional options:

```bash
autofragment bio --input protein.cif \
    --water-cluster-method kmeans \
    --add-implicit-hydrogens \
    --no-infer-bonds \
    --ph 6.0
```

`--add-implicit-hydrogens` uses PDBFixer and OpenMM to add missing hydrogen atoms to the structure before partitioning. This is important for crystal structures, which typically lack hydrogen coordinates. These packages must be installed separately (PDBFixer is available via conda).

`--no-infer-bonds` disables the automatic detection of peptide, disulfide, and ligand bonds. Use this if you only need fragment coordinates and charges without connectivity information.

The CLI writes a JSON file with the same `FragmentTree` structure as the Python API. You can then load it in Python to generate GAMESS inputs or perform further analysis.

## Putting It Together: A Complete Workflow

Here is a condensed end-to-end example that reads a water cluster, partitions it with PCA seeding, writes a GAMESS FMO input, and saves the fragmentation as JSON for reproducibility:

```python
import autofragment as af
from autofragment.io import write_gamess_fmo

# 1. Read and partition
tree = af.partition_xyz(
    "water64.xyz",
    n_fragments=8,
    method="kmeans",
    init_strategy="pca",
)

# 2. Write GAMESS input
write_gamess_fmo(
    tree.fragments,
    "water64_fmo2.inp",
    basis="cc-pVDZ",
    method="RHF",
    nbody=2,
    memory=1000,
    title="Water64 FMO2/RHF/cc-pVDZ",
)

# 3. Archive the partition
tree.to_json("water64_partition.json")

print(f"Wrote GAMESS input with {tree.n_primary} fragments, {tree.n_atoms} atoms")
```

For a protein, replace step 1 with `BioPartitioner`:

```python
from autofragment import BioPartitioner

bio = BioPartitioner(ph=7.4, water_clusters=5)
tree = bio.partition_file("1ubq.cif")

write_gamess_fmo(tree.fragments, "1ubq_fmo2.inp", basis="6-31G*", memory=2000)
tree.to_json("1ubq_partition.json")
```

The JSON archive lets you regenerate GAMESS inputs with different basis sets or method settings without re-running the partitioning step, which is useful when exploring the basis set dependence of FMO interaction energies.

## Quick Reference

| Task | Python API | CLI |
|---|---|---|
| Read XYZ | `io.read_xyz("file.xyz")` | (built into `single`) |
| Flat partition | `af.partition_xyz("f.xyz", n_fragments=4)` | `autofragment single -i f.xyz --n-fragments 4` |
| Full-control partition | `MolecularPartitioner(n_fragments=4).partition(system)` | same CLI with `--method`, `--random-state` |
| Tiered partition | `af.partition_xyz("f.xyz", tiers=2, n_primary=4, n_secondary=4)` | `--tiers 2 --n-primary 4 --n-secondary 4` |
| Balanced clusters | `MolecularPartitioner(method="kmeans_constrained")` | `--method kmeans_constrained` (requires `[balanced]`) |
| Seeded init | `af.partition_xyz(..., init_strategy="pca")` | `--init-strategy pca` |
| Write GAMESS FMO | `write_gamess_fmo(frags, "out.inp", basis="6-31G*")` | (not yet in CLI) |
| Bio partition | `BioPartitioner(ph=7.4).partition_file("p.cif")` | `autofragment bio -i p.cif --ph 7.4` |
| Save JSON | `tree.to_json("f.json")` | (default output format) |
| Load JSON | `FragmentTree.from_json("f.json")` | (use Python API) |
| Check features | `af.has_feature("bio")` / `af.list_features()` | `autofragment info` |

## Next Steps

- Read the [fragment-based methods overview](/blog/fragment-based-quantum-chemistry/) for the theoretical background behind MBE, FMO, and embedded methods.
- Browse the [autofragment documentation](https://brycewestheimer.github.io/autofragment-public) for the full API reference and additional examples.
- See the [GAMESS documentation](https://www.msg.chem.iastate.edu/gamess/documentation.html) for details on FMO keywords, PIEDA output, and analytic gradients.
- For questions or contributions, visit the [autofragment GitHub repository](https://github.com/brycewestheimer/autofragment).
