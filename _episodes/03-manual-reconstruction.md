---
title: "Manual Reconstruction"
teaching: 15
exercises: 5
questions:
- "How do I reconstruct the inclusive kinematics myself?"
objectives:
- "Understand how to calculate the inclusive kinematics manually"
- "Implement the various reconstruction methods in code"
- "Verify your manual calculations through comparison to the InclusiveKinematicsXX values"
keypoints:
- "The reconstruction methods all use some combination of the same basic information: Ee, theta_e, sigma_h, pt_h"
---

## Doing the reconstruction yourself

It may be that you don't want to use the default reconstruction provided in the InclusiveKinematicsXX branches - maybe you want to test a new electron finding or particle flow algorithm? In such cases you will need to calculate the kinematics yourself, as the InclusiveKinematicsXX branches only perform the reconstruction for a single scenario e.g. perfect electron ID, electron energy from tracking etc. If you're having to write the reconstruction methods in your own code, it's good to verify that your implementation of the methods is correct.

We can do this by comparing our manual calculations to the results stored in the InclusiveKinematicsXX branches. Copy the script below into a file called `ManualReconstruction.C`

```console
// PODIO
#include "podio/Frame.h"
#include "podio/ROOTFrameReader.h"

// DATA MODEL
#include "edm4eic/InclusiveKinematicsCollection.h"
#include "edm4eic/ReconstructedParticleCollection.h"
#include "edm4eic/HadronicFinalStateCollection.h"
#include "edm4eic/ClusterCollection.h"
#include "edm4hep/Vector3f.h"

std::vector<float> calc_elec_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam);
std::vector<float> calc_jb_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam);
std::vector<float> calc_da_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam);
std::vector<float> calc_sig_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam);
std::vector<float> calc_esig_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam);

void ManualReconstruction(std::string filename) {

  // Settings
  Float_t E_ebeam = 18;
  Float_t E_pbeam = 275;
  Float_t m_e = 0.000511;
  double xAngle = 25e-3;
  TLorentzVector pni, ei;
  ei.SetPxPyPzE(0, 0, -E_ebeam, E_ebeam);
  pni.SetPxPyPzE(-1*E_pbeam*TMath::Sin(xAngle), 0, E_pbeam*TMath::Cos(xAngle), E_pbeam);

  std::vector<std::string> inFiles = {filename};

  auto reader = podio::ROOTFrameReader();
  reader.openFiles(inFiles);

  // Declare benchmark histograms
  TH1F *hResoX_electron = new TH1F("hResoX_electron","Electron method;#Deltax/x;Counts",500,-1,1);
  TH1F *hResoX_jb = new TH1F("hResoX_jb","JB method;#Deltax/x;Counts",500,-1,1);
  TH1F *hResoX_da = new TH1F("hResoX_da","Double Angle method;#Deltax/x;Counts",500,-1,1);
  TH1F *hResoX_sigma = new TH1F("hResoX_sigma","#Sigma method;#Deltax/x;Counts",500,-1,1);
  TH1F *hResoX_esigma = new TH1F("hResoX_esigma","e-#Sigma method;#Deltax/x;Counts",500,-1,1);

  TH1F *hResoY_electron = new TH1F("hResoY_electron","Electron method;#Deltay/y;Counts",500,-1,1);
  TH1F *hResoY_jb = new TH1F("hResoY_jb","JB method;#Deltay/y;Counts",500,-1,1);
  TH1F *hResoY_da = new TH1F("hResoY_da","Double Angle method;#Deltay/y;Counts",500,-1,1);
  TH1F *hResoY_sigma = new TH1F("hResoY_sigma","#Sigma method;#Deltay/y;Counts",500,-1,1);
  TH1F *hResoY_esigma = new TH1F("hResoY_esigma","e-#Sigma method;#Deltay/y;Counts",500,-1,1);

  TH1F *hResoQ2_electron = new TH1F("hResoQ2_electron","Electron method;#DeltaQ2/Q2;Counts",500,-1,1);
  TH1F *hResoQ2_jb = new TH1F("hResoQ2_jb","JB method;#DeltaQ2/Q2;Counts",500,-1,1);
  TH1F *hResoQ2_da = new TH1F("hResoQ2_da","Double Angle method;#DeltaQ2/Q2;Counts",500,-1,1);
  TH1F *hResoQ2_sigma = new TH1F("hResoQ2_sigma","#Sigma method;#DeltaQ2/Q2;Counts",500,-1,1);
  TH1F *hResoQ2_esigma = new TH1F("hResoQ2_esigma","e-#Sigma method;#DeltaQ2/Q2;Counts",500,-1,1);

  Float_t x_truth, x_electron, x_jb, x_da, x_sigma, x_esigma;
  Float_t y_truth, y_electron, y_jb, y_da, y_sigma, y_esigma;
  Float_t Q2_truth, Q2_electron, Q2_jb, Q2_da, Q2_sigma, Q2_esigma;
  Float_t E, theta, sigma_h, pt_had;

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

    // Retrieve Scattered electron and HFS
    auto& eleCollection = event.get<edm4eic::ReconstructedParticleCollection>("ScatteredElectronsTruth");
    auto& hfsCollection = event.get<edm4eic::HadronicFinalStateCollection>("HadronicFinalState");

    // Store kinematics from InclusiveKinematics branches
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

    TLorentzVector scat_ele;
    E = eleCollection[0].getEnergy();
    auto& ele_momentum = eleCollection[0].getMomentum();
    scat_ele.SetPxPyPzE(ele_momentum.x, ele_momentum.y, ele_momentum.z, E);
    theta = scat_ele.Theta();
    sigma_h = hfsCollection[0].getSigma();
    pt_had = hfsCollection[0].getPT();

    // Calculate kinematics manually
    std::vector<float> elec_reco = calc_elec_method(E, theta, pt_had, sigma_h, E_ebeam, E_pbeam);
    std::vector<float> jb_reco = calc_jb_method(E, theta, pt_had, sigma_h, E_ebeam, E_pbeam);
    std::vector<float> da_reco = calc_da_method(E, theta, pt_had, sigma_h, E_ebeam, E_pbeam);
    std::vector<float> sigma_reco = calc_sig_method(E, theta, pt_had, sigma_h, E_ebeam, E_pbeam);
    std::vector<float> esigma_reco = calc_esig_method(E, theta, pt_had, sigma_h, E_ebeam, E_pbeam);

    // Some example cuts
    bool cuts = true;
    cuts = cuts && (y_truth < 0.95);
    cuts = cuts && (y_truth > 0.01);
    cuts = cuts && (Q2_truth > 1);

    if (!cuts) continue;

    // Fill histograms with difference of calculated kinematics
    // and those retrieved from InclusiveKinematics branches
    hResoX_electron->Fill((x_electron-elec_reco[0])/elec_reco[0]);
    hResoX_jb->Fill((x_jb-jb_reco[0])/jb_reco[0]);
    hResoX_da->Fill((x_da-da_reco[0])/da_reco[0]);
    hResoX_sigma->Fill((x_sigma-sigma_reco[0])/sigma_reco[0]);
    hResoX_esigma->Fill((x_esigma-esigma_reco[0])/esigma_reco[0]);

    hResoY_electron->Fill((y_electron-elec_reco[1])/elec_reco[1]);
    hResoY_jb->Fill((y_jb-jb_reco[1])/jb_reco[1]);
    hResoY_da->Fill((y_da-da_reco[1])/da_reco[1]);
    hResoY_sigma->Fill((y_sigma-sigma_reco[1])/sigma_reco[1]);
    hResoY_esigma->Fill((y_esigma-esigma_reco[1])/esigma_reco[1]);

    hResoQ2_electron->Fill((Q2_electron-elec_reco[2])/elec_reco[2]);
    hResoQ2_jb->Fill((Q2_jb-jb_reco[2])/jb_reco[2]);
    hResoQ2_da->Fill((Q2_da-da_reco[2])/da_reco[2]);
    hResoQ2_sigma->Fill((Q2_sigma-sigma_reco[2])/sigma_reco[2]);
    hResoQ2_esigma->Fill((Q2_esigma-esigma_reco[2])/esigma_reco[2]);
  }
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

  cout << "Done!" << endl;
}

// electron method
std::vector<float> calc_elec_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam) {
  float Q2  = 2.*E_ebeam*E*(1+TMath::Cos(theta));
  float y = 1. - (E/E_ebeam)*TMath::Sin(theta/2)*TMath::Sin(theta/2);
  float x = Q2/(4*E_ebeam*E_pbeam*y);
  return {x, y, Q2};
}

// jb method
std::vector<float> calc_jb_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam) {
  float y = sigma_h/(2*E_ebeam);
  float Q2 = pt_had*pt_had / (1-y);
  float x = Q2/(4*E_ebeam*E_pbeam*y);
  return {x, y, Q2};
}

// float angle method
std::vector<float> calc_da_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam) {
  float alpha_h = sigma_h/pt_had;
  float alpha_e = TMath::Tan(theta/2);
  float y = alpha_h / (alpha_e + alpha_h);
  float Q2 = 4*E_ebeam*E_ebeam / (alpha_e * (alpha_h + alpha_e));
  float x = Q2/(4*E_ebeam*E_pbeam*y);
  return {x, y, Q2};
}

// sigma method
std::vector<float> calc_sig_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam) {
  float y = sigma_h/(sigma_h + E*(1 - TMath::Cos(theta))); 
  float Q2 = E*E*TMath::Sin(theta)*TMath::Sin(theta) / (1-y);
  float x = Q2/(4*E_ebeam*E_pbeam*y);
  return {x, y, Q2};
}

// e-sigma method
std::vector<float> calc_esig_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam) {
  float Q2  = 2.*E_ebeam*E*(1+TMath::Cos(theta));
  float x = calc_sig_method(E,theta,pt_had,sigma_h,E_ebeam,E_pbeam)[0];
  float y = Q2/(4*E_ebeam*E_pbeam*x);
  return {x, y, Q2};
}
```