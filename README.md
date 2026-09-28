# RASPA2 Run on macOS

** Widom Insertion of Ethane in ZIF-8**

 This is a tutorial for RASPA beginners, covering installation through result interpretation and demonstrating a short 1,000-cycle Widom test for ethane in ZIF-8.

This tutorial documents **how to get the workflow running**. It is not a full reproduction of the paper or a validated prediction of material properties.

## 1. What does this calculation do?

- Material: ZIF-8, treated as a rigid framework.
- Adsorbate: ethane.
- Temperature: 303 K, approximately 30 °C.
- Supercell: 2 × 2 × 2.
- Method: Widom test-particle insertion.
- Sampling in this run: 1,000 Monte Carlo cycles.
- Main result: the Henry coefficient in the low-pressure limit.

This run does not produce a full pressure–uptake isotherm or a trajectory describing molecular motion in physical time.

## 2. Installing RASPA2

### 2.1 Check Conda

打开 Mac 的“终端”，输入：

Open Terminal on your Mac and enter:

```bash
conda --version
```

This installation reported `conda 24.9.2`, so I continued directly. If Conda is not installed, install [Miniforge](https://github.com/conda-forge/miniforge#download) for your Mac’s processor architecture, then reopen Terminal.

### 2.2 Create an environment and install

```bash
conda create -n raspa2 --override-channels -c conda-forge raspa2
```

At `Proceed ([y]/n)?`, enter `y` and press Return. Wait for installation to finish, then run the following commands one at a time:

```bash
conda activate raspa2
export RASPA_DIR="$CONDA_PREFIX"
command -v simulate
```

The final command should print a path ending in `/envs/raspa2/bin/simulate`. The full path depends on where Conda is installed.

`RASPA_DIR` points to the active environment so RASPA can locate its bundled data. In each new terminal session, activate the environment and set `RASPA_DIR` again.

> GitHub’s `Code → Download ZIP` provides source code, not a Mac `.dmg` installer. This tutorial uses Conda, so compiling that source is unnecessary. Installation succeeded in this environment; if another platform reports `PackagesNotFoundError`, check package availability for that platform.

### 2.3 Locate the executable in Finder (optional)

Copy the folder path printed by `command -v simulate`, omitting the final `simulate`. In Finder, press `⌘ + Shift + G`, paste the path and press Return to locate the executable.

Leave the executable in place. Run it from the folder containing your simulation inputs rather than double-clicking it.

## 3. Prepare a simulation folder

Create a folder on your Desktop with the following name.

```text
My first RASPA
```

Download the following five files above.

| File | Role |
| --- | --- |
| `ZIF_08.cif` | Unit cell and atomic positions |
| `ethane.def` | Ethane model with two CH3 sites |
| `pseudo_atoms.def` | Atom types, masses, charges, etc. |
| `force_field_mixing_rules.def` | Lennard–Jones parameters and mixing rules |
| `force_field.def` |

Download `RASPA_simulation.input`, rename it **`simulation.input`**, and place it in the same folder.

Your folder should now contain:

```text
My first RASPA/
├── ZIF_08.cif
├── ethane.def
├── pseudo_atoms.def
├── force_field.def
├── force_field_mixing_rules.def
└── simulation.input
```

## 4. Edit simulation.input

Open the file in VS Code or another plain-text editor. Make these four changes and leave the other settings unchanged:

| Original | Replacement | Meaning |
| --- | --- | --- |
| `NumberOfCycles 200000` | `NumberOfCycles 1000` | Start with a short test |
| `Forcefield --force field--` | `Forcefield Local` | Read the local force field |
| `MoleculeName --guest--` | `MoleculeName ethane` | Select ethane |
| `MoleculeDefinition --guest force field--` | `MoleculeDefinition Local` | Read the local molecule definition |

Keep `Component 0` before `MoleculeName`. The complete edited input below is adapted from the original author’s template linked above:

```text
SimulationType                MonteCarlo
NumberOfCycles                1000
NumberOfInitializationCycles  0
PrintEvery                    1000

Forcefield                    Local
RemoveAtomNumberCodeFromLabel no
CutOffVDW                     12.0
ChargeMethod                  ewald
EwaldPrecision                1e-6
OmitAdsorbateAdsorbateCoulombInteractions no

Framework 0
FrameworkName                 ZIF_08
UnitCells                     2 2 2

ExternalTemperature           303.0

Component 0 MoleculeName       ethane
            MoleculeDefinition       Local
            WidomProbability         1.0
            CreateNumberOfMolecules  0
```

Press `⌘ + S` to save. If using TextEdit, save as plain text, not `.rtf`.

## 5. Run the first calculation

Run these commands one at a time in Terminal:

```bash
conda activate raspa2
export RASPA_DIR="$CONDA_PREFIX"
cd "$HOME/Desktop/My first RASPA"
```

If `cd` reports that the folder does not exist, correct the path before proceeding. This path assumes the folder is on the Desktop and its name matches exactly.

Start the calculation:

```bash
simulate > run.log 2>&1
```

RASPA reads `simulation.input` in the current directory. It is normal to see no terminal output while it runs, because output is redirected to `run.log`.

When the command prompt returns, the process has exited. Check the output file to determine whether it completed successfully.

## 6. Check completion

In Finder, folders such as `Output`, `Restart`, and `Movies` should appear in the simulation directory. Open:

```text
Output/System_0/
```

Open the `.data` file in VS Code. In this example, its name is:

```text
output_ZIF_08_2.2.2_303.000000_0.data
```

Press `⌘ + F` and search for `Simulation finished`. This run reported:

```text
Simulation finished, 0 warnings
```

This indicates normal completion with no reported warnings. If the file or completion marker is missing, inspect `run.log` and the end of any existing output.

## 7. Read the Henry coefficient

In the same `.data` file, search for:

```text
Average Henry coefficient:
```

The following values are from my manual run, transcribed from the output screenshot:

```text
Block[ 0] 2.21659e-05 [mol/kg/Pa]
Block[ 1] 2.29866e-05 [mol/kg/Pa]
Block[ 2] 2.22233e-05 [mol/kg/Pa]
Block[ 3] 2.36357e-05 [mol/kg/Pa]
Block[ 4] 2.28669e-05 [mol/kg/Pa]

[ethane] Average Henry coefficient: 2.27757e-05 +/- 7.52442e-07 [mol/kg/Pa]
```

This gives:

**K_H = (2.27757 × 10⁻⁵ ± 7.52442 × 10⁻⁷) mol/(kg·Pa)**

Since 1 bar = 100,000 Pa, multiply both the coefficient and its reported error by 100,000:

**K_H = 2.27757 ± 0.0752442 mol/(kg·bar)**

The reported error is approximately **3.3%** of the mean. These values are also available in [results/henry_1000_cycles.csv](results/henry_1000_cycles.csv).

### What does the result mean?

The Henry coefficient is the initial slope of the adsorption isotherm in the low-pressure limit. In the Henry regime, uptake approximately follows:

```text
q ≈ K_H × p
```

If K_H is in mol/(kg·bar) and p is in bar, q is in mol/kg. This approximation must not be extrapolated arbitrarily to high pressure; the result does not establish an uptake of 2.27757 mol/kg at 1 bar.

`Block[0]` through `Block[4]` are estimates from five sampling blocks. They are relatively close with no obvious monotonic drift, but these values alone do not demonstrate convergence.

Monte Carlo sampling is stochastic, so reruns need not produce identical values. The random seed was not recorded for this run; exact numerical reproduction is not expected.

## 8. Next: increase sampling

**Not yet completed in this record**

1. Duplicate the original folder in Finder and name the copy `RASPA_10000循环`. Keep the original for comparison.
2. In the copy’s `simulation.input`, change `NumberOfCycles` to `10000` and keep all other settings unchanged.
3. Run the following commands with the RASPA2 environment activated:

```bash
cd "$HOME/Desktop/RASPA_10000循环"
simulate > run.log 2>&1
```
