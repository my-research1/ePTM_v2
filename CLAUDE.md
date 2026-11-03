# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ePTM v2 (Enhanced Particle Tracking Model) simulates juvenile salmon migrating through the Sacramento–San Joaquin Delta. It is a fork of California DWR's DSM2 Particle Tracking Model (PTM, Java package `DWR.DMS.PTM`). The fork adds fish behavior (swimming, route selection at junctions, predation mortality) and improved lateral/vertical random-walk mixing. Hydrodynamics (velocities, depths) come from DSM2 HYDRO output (a tidefile in HDF5 format). ePTM does not solve the flow itself. The full user guide is the GitHub Wiki (https://github.com/cvclcm/ePTM_v2/wiki) and `documentation/RunningEPTMv2.docx`.

## Platform constraints

- **Windows only, 32-bit Java 8 (JRE 1.8).** `Globals.java` calls `System.loadLibrary("PTM")`, and `PTMHydroInput`/`Globals` declare `native` methods (tidefile reading, internal↔external channel/node mapping, model date/time). These are implemented in DSM2's native `PTM.dll`, so the model cannot run on Linux or macOS, or under a 64-bit JVM. On this machine (WSL) you can edit and possibly `javac` the code, but you cannot run a simulation.
- There is no build system (no Maven, Gradle or Ant). The code is built as an Eclipse project (see `documentation/ePTM_Eclipse_setup_02mar20.docx` and `documentation/videos/7–9*.mp4`). It is compiled against the jars in `ptm_SWFSC/lib/` (jhdf5, opencsv, COM/edu/xml/pj support jars) and exported as the runnable `eptm.jar` at the repo root. A plain compile check looks roughly like:
  `javac -cp "ptm_SWFSC/lib/*" -d build ptm_SWFSC/DWR/DMS/PTM/*.java`
- There are no tests or linters.

## Running

The entry point is `DWR.DMS.PTM.MainPTM`. It takes 13 positional arguments (see `MainPTM.main`):

```
java -Xss5M -Xms512M -Xmx1024M -jar eptm.jar <ePTM.inp> <behavior.h5> <out.h5> <riverBendsDir> <junctionsDir> \
     ChansOrientsBends812.csv Junctions812.csv latVel.csv latEps.csv latdEp.csv latd2Ep.csv VerEps.csv VerdEp.csv
```

The two directory arguments normally both point to `configFiles/`. Template drivers in `scripts/` (all paths are `PATH_TO_*` placeholders the user fills in):
- `runEPTMBasic.py` runs one simulation. `ePTMRun.py` does the same with a release/run ID argument. `ePTMBatch.py` launches runs in new Windows `cmd` windows. Runs are independent (fish don't interact), so batches are embarrassingly parallel.
- `runAnimEPTM.py` converts the animation binary with `animateEPTM.jar` (source: `animateEPTM/plotPTM/readEPTMbinary.java`).
- `h5rpkBatch.bat` runs `h5repack -l CONTI` on the DSM2 hydro tidefile, which must be done before ePTM can read it.

## Input pipeline

`scripts/ePTMCreateInputs.R` (rhdf5, lubridate, tidyverse) generates the two per-run inputs:
1. **Behavior HDF5 file.** It contains `channelPars` (calibrated per-channel parameters from `configFiles/ePTMv2CalibValidParams812.csv`), `checkpoints`, `swimCode` (11 = current behavior modules), `nodeDecision` (routing model 0–4), `immortal`, sunrise/sunset, and so on. `BehavedParticle` reads these by name with `HDF5Factory`, so dataset names in the R script and in the Java code must stay in sync.
2. **ePTM `.inp` config.** This is DSM2-PTM text format (see `configFiles/ePTMConfigTemplate.inp`). It points at a DSM2 config (`configFiles/DSM2ConfigTemplate.inp`, which supplies `${HYDROTIDEFILE}` and `${TEMPDIR}`) and has blocks such as `SCALAR`, `IO_FILE`, `GROUP`, `PARTICLE_FLUX_OUTPUT` and `PARTICLE_INSERTION`. The R script sets the run window to 7 days before the first release through 10 days after the last.

**Node numbering:** users give release nodes and checkpoints as DSM2 **external** node numbers. The R script converts checkpoints to **internal** numbers using `configFiles/nodesIntExt_v8_1_2.csv` (`channelsIntExt_v8_1_2.csv` maps channels the same way). A mix-up here is a known past source of errors. A new DSM2 grid needs these mapping files, `ChansOrientsBends812.csv`, and `Junctions812.csv` regenerated.

## Code architecture (`ptm_SWFSC/DWR/DMS/PTM/`)

- Most files are upstream DWR PTM: `PTMEnv`/`PTMFixedInput`/`*FixedData` parse the `.inp`, `PTMHydroInput` reads the tidefile via JNI, and `Waterbody` → `Channel`/`Reservoir`/`Conveyor`/`Boundary` plus `Node` and `XSection` form the network. The `*Output`/`*Flux*`/`Group*` classes handle outputs.
- ePTM additions (comments often tagged `VKS` or "Sridharan"):
  - `BehavedParticle extends Particle`: all fish behavior. It holds static behavior parameters loaded from the behavior HDF5, swim modes switched on `swimCode`, junction routing switched on `nodeDecision`, checkpoint arrival recording (`checkCheckpoints`), and HDF5 output writing (`IHDF5SimpleWriter`).
  - `Particle`: core movement and random-walk mixing. `MainPTM` sets `Particle.mixCase = 4` (the new mixing algorithm).
  - `SmartChannel`: a channel with orientation and bend data. `RiverBendsInput` and `LatData`/`VertData`/`ProfileData` load the lateral and vertical velocity and eddy-diffusivity profiles from the `configFiles/lat*.csv` and `Ver*.csv` files.
  - `JunctionsInput`, `DecisionTree`: junction-specific routing data.
- `MainPTM` flow: init globals and env → read bends and junctions → update `SmartChannel` orientations → create `BehavedParticle[]` → set insertions → time-step loop → write the behavior output HDF5.

## Repo contents

`Data/` holds the acoustic telemetry datasets used for calibration and validation (see `Data/readme`). `documentation/` has papers and tutorial videos. `modelFiles/` has DSM2 grid KML files (internal and external numbering). `SupportingMaterial/` has README images and the calibration/validation deck. The large binaries (`eptm.jar`, `animateEPTM.jar`, the jars under `lib/`, mp4, pdf) are committed to git.
