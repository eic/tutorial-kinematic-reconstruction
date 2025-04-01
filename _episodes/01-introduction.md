---
title: "Introduction"
teaching: 15
exercises: 5
questions:
- "How do I access the inclusive kinematics in the simulation output?"
objectives:
- "Download files for the tutorial"
- "Plot basic distributions for x, y, Q2 with different reconstruction methods"
keypoints:
- "Use `xrdfs` from within the eic-shell to browse available files from simulations campaigns."
- "Use `xrdcp` from within eic-shell to copy files to your local environment."
- "Access the reconstructed kinematics using the InclusiveKinematicsXX branches"
---

More detailed instructions on streaming and downloading simulation files can be found in the [Analysis Tutorial](https://eic.github.io/tutorial-analysis/)

Let's start by downloading our files. We will look at two different files from the March 2025 campaign: a low Q2 and a high Q2 Neutral Current DIS file.

## Access Simulation from Jefferson Lab xrootd

The preferred method for browsing the simulation output is to use xrootd from within the eic-shell. To browse the directory structure and exit, one can run the commands:
```console
./eic-shell
xrdfs root://dtn-eic.jlab.org
ls /volatile/eic/EPIC/RECO/25.03.1
exit
```
Once you've located your desired file, you can copy it to your local system using the `xrdcp` command:
```console
./eic-shell
xrdcp root://dtn-eic.jlab.org//volatile/eic/EPIC/RECO/25.03.1/path-to-file .
exit
```

> Note: For simulation campaigns before 2025, the destination is /work/eic2/EPIC rather than /volatile/eic/EPIC
{: .callout}

## Download files for the next step!

We will need a file to analyse going forward, if you have not done so, download a file now!

Grab files using -
```console
xrdcp root://dtn-eic.jlab.org//volatile/eic/EPIC/RECO/25.03.1/epic_craterlake/DIS/NC/18x275/minQ2=1/pythia8NCDIS_18x275_minQ2=1_beamEffects_xAngle=-0.025_hiDiv_1.0001.eicrecon.tree.edm4eic.root ./
xrdcp root://dtn-eic.jlab.org//volatile/eic/EPIC/RECO/25.03.1/epic_craterlake/DIS/NC/18x275/minQ2=1000/pythia8NCDIS_18x275_minQ2=1000_beamEffects_xAngle=-0.025_hiDiv_1.0001.eicrecon.tree.edm4eic.root ./
```
Note that the ./ at the end is the target location to copy to. Change this as desired.

## Inspect the InclusiveKinematics branches

```console
root -l pythia8NCDIS_18x275_minQ2=1_beamEffects_xAngle=-0.025_hiDiv_1.0001.eicrecon.tree.edm4eic.root
TBrowser b
events->Draw("InclusiveKinematicsTruth.x")
```