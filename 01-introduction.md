---
title: "Introduction"
teaching: 15
exercises: 5
---

::::::::::::::::::::::::::::::::::::::::::::: questions

- How do I access the inclusive kinematics in the simulation output?

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: objectives

- Download files for the tutorial.
- Plot basic distributions for x, y, Q2 with different reconstruction methods.

:::::::::::::::::::::::::::::::::::::::::::::

More detailed instructions on streaming and downloading simulation files can be found in the
[Analysis Tutorial](https://eic.github.io/tutorial-analysis/).

## Access Simulation from Jefferson Lab xrootd

The preferred method for browsing the simulation output is to use xrootd from within the eic-shell. To browse the directory structure and exit, one can run the commands:
```bash
./eic-shell
xrdfs root://dtn-eic.jlab.org
ls /volatile/eic/EPIC/RECO/
exit
```
Once you've located your desired file, you can copy it to your local system using the `xrdcp` command:
```bash
xrdcp root://dtn-eic.jlab.org//volatile/eic/EPIC/RECO/path-to-file ./
```

::::::::::::::::::::::::::::::::::::::::::::: callout

Note: For simulation campaigns before January 2025, the destination is `/work/eic2/EPIC` rather than
`/volatile/eic/EPIC`.

:::::::::::::::::::::::::::::::::::::::::::::

## Download files for the next step!

Let's start by downloading our files. We will look at two different files from the `26.02.0` reconstruction campaign: a low Q2 and a high Q2 Neutral Current DIS file.

From within the current `eic-shell` we can grab our files using -
```bash
xrdcp root://dtn-eic.jlab.org//volatile/eic/EPIC/RECO/26.02.0/epic_craterlake/DIS/NC/18x275/minQ2=1/pythia8NCDIS_18x275_minQ2=1_beamEffects_xAngle=-0.025_hiDiv_1.0001.eicrecon.edm4eic.root ./
xrdcp root://dtn-eic.jlab.org//volatile/eic/EPIC/RECO/26.02.0/epic_craterlake/DIS/NC/18x275/minQ2=1000/pythia8NCDIS_18x275_minQ2=1000_beamEffects_xAngle=-0.025_hiDiv_1.0001.eicrecon.edm4eic.root ./
```
Note that the ./ at the end is the target location to copy to. Change this as desired.

## Inspect the InclusiveKinematics branches

From here you you can click around the browser to inspect the basic features of the distributions - look for branches titled "InclusiveKinematics*".

Open the file in ROOT:
```bash
root -l pythia8NCDIS_18x275_minQ2=1_beamEffects_xAngle=-0.025_hiDiv_1.0001.eicrecon.edm4eic.root
TBrowser b
```

It may be inconvenient to do everything through the `TBrowser` if you want to compare the distributions, look at multiple files, or to save the histograms. Using your preferred text editor, create a file with the name `PlotDistributions.C`, and paste the following code:
```c++
void PlotDistributions(TString filename){
  
  std::vector<TString> recon_method = {"Truth", "Electron", "JB", "DA", "Sigma", "ESigma"};

  // Open the file and retrieve the chain
  auto tree = new TChain("events");
  tree->Add(filename);
  
  for (auto method : recon_method){
    
    auto canvas = new TCanvas();
    canvas->Divide(2,2);
    
    TString branch_name;
    canvas->cd(1);
    
    // Draw a histogram for each variable as reconstructed by each method
    branch_name = TString::Format("InclusiveKinematics%s.x",method.Data());
    tree->Draw(branch_name);
    canvas->cd(2);
    branch_name = TString::Format("InclusiveKinematics%s.y",method.Data());
    tree->Draw(branch_name);
    canvas->cd(3);
    branch_name = TString::Format("InclusiveKinematics%s.Q2",method.Data());
    tree->Draw(branch_name);
    canvas->cd(4);
    branch_name = TString::Format("InclusiveKinematics%s.W",method.Data());
    tree->Draw(branch_name);

    branch_name = TString::Format("InclusiveKinematics%s.pdf",method.Data());
    canvas->Print(branch_name); // Write the canvases to a pdf file
  }
}
```
You can then run this script as
```bash
root -l PlotDistributions.C\(\"pythia8NCDIS_18x275_minQ2=1_beamEffects_xAngle=-0.025_hiDiv_1.0001.eicrecon.edm4eic.root\"\)
```
replacing the file name with the name of the file that you want to plot.

This script produces histograms of the distributions of the inclusive kinematic variables when reconstructed using several different reconstruction methods. Assuming that there are no major issues in the reconstruction, these distributions should look similar to the true distribution, with some smearing due to resolution effects.

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise

Run `PlotDistributions.C` on both the low Q2 and the high Q2 file you downloaded and compare the
distributions produced by the different reconstruction methods.

::::::::::::::: solution

Each reconstruction method (`Electron`, `JB`, `DA`, `Sigma`, `ESigma`) produces its own set of
`x`, `y`, `Q2`, and `W` histograms. They should all broadly resemble the `Truth` distribution, with
method-dependent smearing. Differences between the low and high Q2 files reflect how each method
performs in different regions of phase space, which is explored in the following episodes.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: keypoints

- Use `xrdfs` from within the eic-shell to browse available files from simulations campaigns.
- Use `xrdcp` from within eic-shell to copy files to your local environment.
- Access the reconstructed kinematics using the `InclusiveKinematicsXX` branches.

:::::::::::::::::::::::::::::::::::::::::::::
