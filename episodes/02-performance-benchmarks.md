---
title: "Performance Benchmarks"
teaching: 15
exercises: 5
questions:
- "How do I determine which reconstruction method I should be using?"
objectives:
- "Produce plots that benchmark the performance of different reconstruction methods"
keypoints:
- "Use `ROOTFrameReader` to process simulation files using the data types implemented in `edm4hep`/`edm4eic`."
---

## Using ROOTFrameReader to process simulation files

The collections contained in the simulation output often rely on data types made available by `edm4hep` and `edm4eic`. These are based on the podio EDM toolkit, which provides its own tools for reading in event data, though approaches using e.g. `TTreeReader` or `RDataFrame` are also possible. The data model contains functions that can make key information more accessible. Take the `edm4eic:ReconstructedParticle` type (see [here](https://eic.github.io/EDM4eic/classedm4eic_1_1_reconstructed_particle.html)) as an example:

Go into a ROOT prompt (`root -l`) and create an `edm4eic::ReconstructedParticle` object
```cpp
#include <edm4eic/ReconstructedParticleCollection.h>
edm4eic::ReconstructedParticle rcp
```
For such an object you can access the tracks or clusters associated with the reconstructed particle as
```cpp
rcp.getTracks()
rcp.getClusters()
```
which would return a list of the associated tracks/clusters. As our `rcp` was just initialised, the lists are empty - for the objects in the simulation output this won't be the case.

If you're not using data frames, you probably do your analysis in an event loop. An event loop with the `ROOTFrameReader` would look somthing like this
```cpp
#include "podio/Frame.h"
#include "podio/ROOTFrameReader.h"
#include "edm4eic/ReconstructedParticleCollection.h"

auto reader = podio::ROOTFrameReader();
reader.openFile("some_file.root");

for (size_t i = 0; i < reader.getEntries("events"); i++) {
    const auto event = podio::Frame(reader.readNextEntry("events"));
    auto& reco_collection = event.get<edm4eic::ReconstructedParticleCollection>("ReconstructedParticles");	
    // Your analysis here
}
```

Below is a full script to produce some resolution benchmark plots using the `InclusiveKinematicsXX` branches - copy it into a file called `BenchmarkReconstruction.C`
```cpp
// PODIO
#include "podio/Frame.h"
#include "podio/ROOTFrameReader.h"

// DATA MODEL
#include "edm4eic/InclusiveKinematicsCollection.h"

template <class T>
void BinLogX(T *h)
{
   TAxis *axis = h->GetXaxis();
   int bins = axis->GetNbins();

   Axis_t from = TMath::Log10(axis->GetXmin());
   Axis_t to = TMath::Log10(axis->GetXmax());
   Axis_t width = (to - from) / bins;
   Axis_t *new_bins = new Axis_t[bins + 1];

   for (int i = 0; i <= bins; i++) {
     new_bins[i] = TMath::Power(10, from + i * width);
   }
   axis->Set(bins, new_bins);
   delete[] new_bins;
}

void BenchmarkReconstruction(std::string filename, bool bin_log=false) {

  std::vector<std::string> inFiles = {filename};

  auto reader = podio::ROOTFrameReader();
  reader.openFiles(inFiles);

  // Declare benchmark histograms
  TH1F *hResoX_electron = new TH1F("hResoX_electron","Electron method;#Deltax/x;Counts",100,-1,1);
  TH1F *hResoX_jb = new TH1F("hResoX_jb","JB method;#Deltax/x;Counts",100,-1,1);
  TH1F *hResoX_da = new TH1F("hResoX_da","Double Angle method;#Deltax/x;Counts",100,-1,1);
  TH1F *hResoX_sigma = new TH1F("hResoX_sigma","#Sigma method;#Deltax/x;Counts",100,-1,1);
  TH1F *hResoX_esigma = new TH1F("hResoX_esigma","e-#Sigma method;#Deltax/x;Counts",100,-1,1);

  TH1F *hResoY_electron = new TH1F("hResoY_electron","Electron method;#Deltay/y;Counts",100,-1,1);
  TH1F *hResoY_jb = new TH1F("hResoY_jb","JB method;#Deltay/y;Counts",100,-1,1);
  TH1F *hResoY_da = new TH1F("hResoY_da","Double Angle method;#Deltay/y;Counts",100,-1,1);
  TH1F *hResoY_sigma = new TH1F("hResoY_sigma","#Sigma method;#Deltay/y;Counts",100,-1,1);
  TH1F *hResoY_esigma = new TH1F("hResoY_esigma","e-#Sigma method;#Deltay/y;Counts",100,-1,1);

  TH1F *hResoQ2_electron = new TH1F("hResoQ2_electron","Electron method;#DeltaQ2/Q2;Counts",100,-1,1);
  TH1F *hResoQ2_jb = new TH1F("hResoQ2_jb","JB method;#DeltaQ2/Q2;Counts",100,-1,1);
  TH1F *hResoQ2_da = new TH1F("hResoQ2_da","Double Angle method;#DeltaQ2/Q2;Counts",100,-1,1);
  TH1F *hResoQ2_sigma = new TH1F("hResoQ2_sigma","#Sigma method;#DeltaQ2/Q2;Counts",100,-1,1);
  TH1F *hResoQ2_esigma = new TH1F("hResoQ2_esigma","e-#Sigma method;#DeltaQ2/Q2;Counts",100,-1,1);

  TH2F *hResoX_2D_electron = new TH2F("hResoX_2D_electron","Electron method;y;#Deltax/x",30,0.001,1,30,-1,1);
  TH2F *hResoX_2D_jb = new TH2F("hResoX_2D_jb","JB method;y;#Deltax/x",30,0.001,1,30,-1,1);
  TH2F *hResoX_2D_da = new TH2F("hResoX_2D_da","Double Angle method;y;#Deltax/x",30,0.001,1,30,-1,1);
  TH2F *hResoX_2D_sigma = new TH2F("hResoX_2D_sigma","#Sigma method;y;#Deltax/x",30,0.001,1,30,-1,1);
  TH2F *hResoX_2D_esigma = new TH2F("hResoX_2D_esigma","e-#Sigma method;y;#Deltax/x",30,0.001,1,30,-1,1);

  TH2F *hResoY_2D_electron = new TH2F("hResoY_2D_electron","Electron method;y;#Deltay/y",30,0.001,1,30,-1,1);
  TH2F *hResoY_2D_jb = new TH2F("hResoY_2D_jb","JB method;y;#Deltay/y",30,0.001,1,30,-1,1);
  TH2F *hResoY_2D_da = new TH2F("hResoY_2D_da","Double Angle method;y;#Deltay/y",30,0.001,1,30,-1,1);
  TH2F *hResoY_2D_sigma = new TH2F("hResoY_2D_sigma","#Sigma method;y;#Deltay/y",30,0.001,1,30,-1,1);
  TH2F *hResoY_2D_esigma = new TH2F("hResoY_2D_esigma","e-#Sigma method;y;#Deltay/y",30,0.001,1,30,-1,1);

  TH2F *hResoQ2_2D_electron = new TH2F("hResoQ2_2D_electron","Electron method;y;#DeltaQ2/Q2",30,0.001,1,30,-1,1);
  TH2F *hResoQ2_2D_jb = new TH2F("hResoQ2_2D_jb","JB method;y;#DeltaQ2/Q2",30,0.001,1,30,-1,1);
  TH2F *hResoQ2_2D_da = new TH2F("hResoQ2_2D_da","Double Angle method;y;#DeltaQ2/Q2",30,0.001,1,30,-1,1);
  TH2F *hResoQ2_2D_sigma = new TH2F("hResoQ2_2D_sigma","#Sigma method;y;#DeltaQ2/Q2",30,0.001,1,30,-1,1);
  TH2F *hResoQ2_2D_esigma = new TH2F("hResoQ2_2D_esigma","e-#Sigma method;y;#DeltaQ2/Q2",30,0.001,1,30,-1,1);

  // Logarithmic binning on x axis of 2D plots
  if (bin_log){
    BinLogX(hResoX_2D_electron);
    BinLogX(hResoX_2D_jb);
    BinLogX(hResoX_2D_da);
    BinLogX(hResoX_2D_sigma);
    BinLogX(hResoX_2D_esigma);
    
    BinLogX(hResoY_2D_electron);
    BinLogX(hResoY_2D_jb);
    BinLogX(hResoY_2D_da);
    BinLogX(hResoY_2D_sigma);
    BinLogX(hResoY_2D_esigma);
    
    BinLogX(hResoQ2_2D_electron);
    BinLogX(hResoQ2_2D_jb);
    BinLogX(hResoQ2_2D_da);
    BinLogX(hResoQ2_2D_sigma);
    BinLogX(hResoQ2_2D_esigma);
  }
  
  Float_t x_truth, x_electron, x_jb, x_da, x_sigma, x_esigma;
  Float_t y_truth, y_electron, y_jb, y_da, y_sigma, y_esigma;
  Float_t Q2_truth, Q2_electron, Q2_jb, Q2_da, Q2_sigma, Q2_esigma;

  cout << reader.getEntries("events") << " events found" << endl;
  for (size_t i = 0; i < reader.getEntries("events"); i++) {// begin event loop
    const auto event = podio::Frame(reader.readNextEntry("events"));
    if (i%100==0) cout << i << " events processed" << endl;

    // Retrieve Inclusive Kinematics Collections
    auto& kin_truth = event.get<edm4eic::InclusiveKinematicsCollection>("InclusiveKinematicsTruth");
    auto& kin_electron = event.get<edm4eic::InclusiveKinematicsCollection>("InclusiveKinematicsElectron");
    auto& kin_jb = event.get<edm4eic::InclusiveKinematicsCollection>("InclusiveKinematicsJB");
    auto& kin_da = event.get<edm4eic::InclusiveKinematicsCollection>("InclusiveKinematicsDA");
    auto& kin_sigma = event.get<edm4eic::InclusiveKinematicsCollection>("InclusiveKinematicsSigma");
    auto& kin_esigma = event.get<edm4eic::InclusiveKinematicsCollection>("InclusiveKinematicsESigma");

    if (kin_truth.empty() || kin_electron.empty() || kin_jb.empty()) continue;

    x_truth = kin_truth.x()[0];
    x_electron = kin_electron.x()[0];
    x_jb = kin_jb.x()[0];
    x_da = kin_da.x()[0];
    x_sigma = kin_sigma.x()[0];
    x_esigma = kin_esigma.x()[0];

    y_truth = kin_truth.y()[0];
    y_electron = kin_electron.y()[0];
    y_jb = kin_jb.y()[0];
    y_da = kin_da.y()[0];
    y_sigma = kin_sigma.y()[0];
    y_esigma = kin_esigma.y()[0];

    Q2_truth = kin_truth.Q2()[0];
    Q2_electron = kin_electron.Q2()[0];
    Q2_jb = kin_jb.Q2()[0];
    Q2_da = kin_da.Q2()[0];
    Q2_sigma = kin_sigma.Q2()[0];
    Q2_esigma = kin_esigma.Q2()[0];

    // Some example cuts
    bool cuts = true;
    cuts = cuts && (y_truth < 0.95);
    cuts = cuts && (y_truth > 0.01);
    cuts = cuts && (Q2_truth > 1);

    if (!cuts) continue;

    hResoX_electron->Fill((x_electron-x_truth)/x_truth);
    hResoX_jb->Fill((x_jb-x_truth)/x_truth);
    hResoX_da->Fill((x_da-x_truth)/x_truth);
    hResoX_sigma->Fill((x_sigma-x_truth)/x_truth);
    hResoX_esigma->Fill((x_esigma-x_truth)/x_truth);

    hResoY_electron->Fill((y_electron-y_truth)/y_truth);
    hResoY_jb->Fill((y_jb-y_truth)/y_truth);
    hResoY_da->Fill((y_da-y_truth)/y_truth);
    hResoY_sigma->Fill((y_sigma-y_truth)/y_truth);
    hResoY_esigma->Fill((y_esigma-y_truth)/y_truth);

    hResoQ2_electron->Fill((Q2_electron-Q2_truth)/Q2_truth);
    hResoQ2_jb->Fill((Q2_jb-Q2_truth)/Q2_truth);
    hResoQ2_da->Fill((Q2_da-Q2_truth)/Q2_truth);
    hResoQ2_sigma->Fill((Q2_sigma-Q2_truth)/Q2_truth);
    hResoQ2_esigma->Fill((Q2_esigma-Q2_truth)/Q2_truth);

    hResoX_2D_electron->Fill(y_truth, (x_electron-x_truth)/x_truth);
    hResoX_2D_jb->Fill(y_truth, (x_jb-x_truth)/x_truth);
    hResoX_2D_da->Fill(y_truth, (x_da-x_truth)/x_truth);
    hResoX_2D_sigma->Fill(y_truth, (x_sigma-x_truth)/x_truth);
    hResoX_2D_esigma->Fill(y_truth, (x_esigma-x_truth)/x_truth);

    hResoY_2D_electron->Fill(y_truth, (y_electron-y_truth)/y_truth);
    hResoY_2D_jb->Fill(y_truth, (y_jb-y_truth)/y_truth);
    hResoY_2D_da->Fill(y_truth, (y_da-y_truth)/y_truth);
    hResoY_2D_sigma->Fill(y_truth, (y_sigma-y_truth)/y_truth);
    hResoY_2D_esigma->Fill(y_truth, (y_esigma-y_truth)/y_truth);

    hResoQ2_2D_electron->Fill(y_truth, (Q2_electron-Q2_truth)/Q2_truth);
    hResoQ2_2D_jb->Fill(y_truth, (Q2_jb-Q2_truth)/Q2_truth);
    hResoQ2_2D_da->Fill(y_truth, (Q2_da-Q2_truth)/Q2_truth);
    hResoQ2_2D_sigma->Fill(y_truth, (Q2_sigma-Q2_truth)/Q2_truth);
    hResoQ2_2D_esigma->Fill(y_truth, (Q2_esigma-Q2_truth)/Q2_truth);
  }// end event loop

  // Drawing the histograms
  auto canvas_x_1D = new TCanvas();
  canvas_x_1D->Divide(3,2);
  canvas_x_1D->cd(1);hResoX_electron->Draw("hist");
  canvas_x_1D->cd(2);hResoX_jb->Draw("hist");
  canvas_x_1D->cd(3);hResoX_da->Draw("hist");
  canvas_x_1D->cd(4);hResoX_sigma->Draw("hist");
  canvas_x_1D->cd(5);hResoX_esigma->Draw("hist");
  auto canvas_y_1D = new TCanvas();
  canvas_y_1D->Divide(3,2);
  canvas_y_1D->cd(1);hResoY_electron->Draw("hist");
  canvas_y_1D->cd(2);hResoY_jb->Draw("hist");
  canvas_y_1D->cd(3);hResoY_da->Draw("hist");
  canvas_y_1D->cd(4);hResoY_sigma->Draw("hist");
  canvas_y_1D->cd(5);hResoY_esigma->Draw("hist");
  auto canvas_Q2_1D = new TCanvas();
  canvas_Q2_1D->Divide(3,2);
  canvas_Q2_1D->cd(1);hResoQ2_electron->Draw("hist");
  canvas_Q2_1D->cd(2);hResoQ2_jb->Draw("hist");
  canvas_Q2_1D->cd(3);hResoQ2_da->Draw("hist");
  canvas_Q2_1D->cd(4);hResoQ2_sigma->Draw("hist");
  canvas_Q2_1D->cd(5);hResoQ2_esigma->Draw("hist");
  
  auto canvas_x_2D = new TCanvas();
  canvas_x_2D->Divide(3,2);
  canvas_x_2D->cd(1);if(bin_log) gPad->SetLogx();hResoX_2D_electron->Draw("colz");
  canvas_x_2D->cd(2);if(bin_log) gPad->SetLogx();hResoX_2D_jb->Draw("colz");
  canvas_x_2D->cd(3);if(bin_log) gPad->SetLogx();hResoX_2D_da->Draw("colz");
  canvas_x_2D->cd(4);if(bin_log) gPad->SetLogx();hResoX_2D_sigma->Draw("colz");
  canvas_x_2D->cd(5);if(bin_log) gPad->SetLogx();hResoX_2D_esigma->Draw("colz");
  auto canvas_y_2D = new TCanvas();
  canvas_y_2D->Divide(3,2);
  canvas_y_2D->cd(1);if(bin_log) gPad->SetLogx();hResoY_2D_electron->Draw("colz");
  canvas_y_2D->cd(2);if(bin_log) gPad->SetLogx();hResoY_2D_jb->Draw("colz");
  canvas_y_2D->cd(3);if(bin_log) gPad->SetLogx();hResoY_2D_da->Draw("colz");
  canvas_y_2D->cd(4);if(bin_log) gPad->SetLogx();hResoY_2D_sigma->Draw("colz");
  canvas_y_2D->cd(5);if(bin_log) gPad->SetLogx();hResoY_2D_esigma->Draw("colz");
  auto canvas_Q2_2D = new TCanvas();
  canvas_Q2_2D->Divide(3,2);
  canvas_Q2_2D->cd(1);if(bin_log) gPad->SetLogx();hResoQ2_2D_electron->Draw("colz");
  canvas_Q2_2D->cd(2);if(bin_log) gPad->SetLogx();hResoQ2_2D_jb->Draw("colz");
  canvas_Q2_2D->cd(3);if(bin_log) gPad->SetLogx();hResoQ2_2D_da->Draw("colz");
  canvas_Q2_2D->cd(4);if(bin_log) gPad->SetLogx();hResoQ2_2D_sigma->Draw("colz");
  canvas_Q2_2D->cd(5);if(bin_log) gPad->SetLogx();hResoQ2_2D_esigma->Draw("colz");

  cout << "Done!" << endl;
}
```
This script sets up the benchmark histograms, fills them in the event loop, and then draws them. Here, the resolutions on the reconstructed kinematic variables are chosen as the benchmarks, both the 1-dimensional `(reco-true)/true` distribution, and also 2-dimensional plots vs inelasticity, y. For a good reconstruction method, the `(reco-true)/true` distribution is centred on zero, with small fluctuations.

Run this script as 
```console
root -l BenchmarkReconstruction.C\(\"your_file.root\"\)
```
or as
```console
root -l BenchmarkReconstruction.C\(\"your_file.root\",true\)
```
to bin logarithmically in inelasticity. 

You may wish to investigate how the resolutions change in a scenario more relevant to your analysis. A set of example cuts are provided in the script

```console
// Some example cuts
bool cuts = true;
cuts = cuts && (y_truth < 0.95);
cuts = cuts && (y_truth > 0.01);
cuts = cuts && (Q2_truth > 1);
```
These can be replaced with whatever cuts are used in your analysis, or you could use them to select areas of the phase space that you wish to investigate.