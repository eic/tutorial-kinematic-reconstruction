---
title: "Optimise Reconstruction"
teaching: 15
exercises: 5
questions:
- "How can I improve the reconstruction when using standard methods?"
objectives:
- "Compare kinematic resolutions for electron reconstruction based on tracks alone and for tracks+calorimetry"
- "Use realistic scattered electron ID in the reconstruction"
keypoints:
- "Some kinematics may favour a calorimeter based electron energy over a tracking based determination - check which is better for your analysis"
---

## Optimising the reconstruction

There are four quantities that are used to reconstruct the inclusive kinematics: the scattered electron energy and polar angle, the hadronic final state (HFS) transverse momentum, and the E-pz sum of all particles in the HFS. The values of these quantities depends on the information used in the reconstruction of the scattered electron and the HFS. In some regions of the phase space, the calorimeters may do a much better job of reconstructing the scattered electron compared to the tracking system.

We can investigate this using the script below, which should be copied into a file called `OptimiseReconstruction.C`

```cpp
// PODIO
#include "podio/Frame.h"
#include "podio/ROOTFrameReader.h"

// DATA MODEL
#include "edm4eic/InclusiveKinematicsCollection.h"
#include "edm4hep/MCParticleCollection.h"
#include "edm4eic/ReconstructedParticleCollection.h"
#include "edm4eic/MCRecoClusterParticleAssociationCollection.h"
#include "edm4eic/MCRecoParticleAssociationCollection.h"
#include "edm4eic/HadronicFinalStateCollection.h"
#include "edm4eic/ClusterCollection.h"
#include "edm4hep/Vector3f.h"

std::vector<float> calc_elec_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam);
std::vector<float> calc_jb_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam);
std::vector<float> calc_da_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam);
std::vector<float> calc_sig_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam);
std::vector<float> calc_esig_method(float E, float theta, float pt_had, float sigma_h, float E_ebeam, float E_pbeam);

void OptimiseReconstruction(std::string filename) {

  // Settings
  Float_t E_ebeam = 18;
  Float_t E_pbeam = 275;
  Float_t m_e = 0.000511;

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

  TH1F *hResoX_calo_electron = new TH1F("hResoX_calo_electron","Electron method;#Deltax/x;Counts",100,-1,1);
  TH1F *hResoX_calo_jb = new TH1F("hResoX_calo_jb","JB method;#Deltax/x;Counts",100,-1,1);
  TH1F *hResoX_calo_da = new TH1F("hResoX_calo_da","Double Angle method;#Deltax/x;Counts",100,-1,1);
  TH1F *hResoX_calo_sigma = new TH1F("hResoX_calo_sigma","#Sigma method;#Deltax/x;Counts",100,-1,1);
  TH1F *hResoX_calo_esigma = new TH1F("hResoX_calo_esigma","e-#Sigma method;#Deltax/x;Counts",100,-1,1);

  TH1F *hResoY_calo_electron = new TH1F("hResoY_calo_electron","Electron method;#Deltay/y;Counts",100,-1,1);
  TH1F *hResoY_calo_jb = new TH1F("hResoY_calo_jb","JB method;#Deltay/y;Counts",100,-1,1);
  TH1F *hResoY_calo_da = new TH1F("hResoY_calo_da","Double Angle method;#Deltay/y;Counts",100,-1,1);
  TH1F *hResoY_calo_sigma = new TH1F("hResoY_calo_sigma","#Sigma method;#Deltay/y;Counts",100,-1,1);
  TH1F *hResoY_calo_esigma = new TH1F("hResoY_calo_esigma","e-#Sigma method;#Deltay/y;Counts",100,-1,1);

  TH1F *hResoQ2_calo_electron = new TH1F("hResoQ2_calo_electron","Electron method;#DeltaQ2/Q2;Counts",100,-1,1);
  TH1F *hResoQ2_calo_jb = new TH1F("hResoQ2_calo_jb","JB method;#DeltaQ2/Q2;Counts",100,-1,1);
  TH1F *hResoQ2_calo_da = new TH1F("hResoQ2_calo_da","Double Angle method;#DeltaQ2/Q2;Counts",100,-1,1);
  TH1F *hResoQ2_calo_sigma = new TH1F("hResoQ2_calo_sigma","#Sigma method;#DeltaQ2/Q2;Counts",100,-1,1);
  TH1F *hResoQ2_calo_esigma = new TH1F("hResoQ2_calo_esigma","e-#Sigma method;#DeltaQ2/Q2;Counts",100,-1,1);

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
    auto& eleCollection = event.get<edm4eic::ReconstructedParticleCollection>("ScatteredElectronsEMinusPz");
    auto& hfsCollection = event.get<edm4eic::HadronicFinalStateCollection>("HadronicFinalState");

    auto& ecalClusters = event.get<edm4eic::ClusterCollection>("EcalClusters");

    // Store kinematics from InclusiveKinematics branches
    if (kin_truth.empty() || kin_electron.empty() || kin_jb.empty()) continue;
    if (eleCollection.empty()) continue;

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

    x_truth = kin_truth.x()[0];
    y_truth = kin_truth.y()[0];
    Q2_truth = kin_truth.Q2()[0];

    x_electron = elec_reco[0];
    x_jb = jb_reco[0];
    x_da = da_reco[0];
    x_sigma = sigma_reco[0];
    x_esigma = esigma_reco[0];

    y_electron = elec_reco[1];
    y_jb = jb_reco[1];
    y_da = da_reco[1];
    y_sigma = sigma_reco[1];
    y_esigma = esigma_reco[1];

    Q2_electron = elec_reco[2];
    Q2_jb = jb_reco[2];
    Q2_da = da_reco[2];
    Q2_sigma = sigma_reco[2];
    Q2_esigma = esigma_reco[2];

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

    // Replace electron momentum with energy from calo cluster
    // cout << eleCollection[0].getClusters().size() << endl;
    E = eleCollection[0].getClusters()[0].getEnergy();

    // Recalculate kinematics
    std::vector<float> calo_elec_reco = calc_elec_method(E, theta, pt_had, sigma_h, E_ebeam, E_pbeam);
    std::vector<float> calo_jb_reco = calc_jb_method(E, theta, pt_had, sigma_h, E_ebeam, E_pbeam);
    std::vector<float> calo_da_reco = calc_da_method(E, theta, pt_had, sigma_h, E_ebeam, E_pbeam);
    std::vector<float> calo_sigma_reco = calc_sig_method(E, theta, pt_had, sigma_h, E_ebeam, E_pbeam);
    std::vector<float> calo_esigma_reco = calc_esig_method(E, theta, pt_had, sigma_h, E_ebeam, E_pbeam);

    x_electron = calo_elec_reco[0];
    x_jb = calo_jb_reco[0];
    x_da = calo_da_reco[0];
    x_sigma = calo_sigma_reco[0];
    x_esigma = calo_esigma_reco[0];

    y_electron = calo_elec_reco[1];
    y_jb = calo_jb_reco[1];
    y_da = calo_da_reco[1];
    y_sigma = calo_sigma_reco[1];
    y_esigma = calo_esigma_reco[1];

    Q2_electron = calo_elec_reco[2];
    Q2_jb = calo_jb_reco[2];
    Q2_da = calo_da_reco[2];
    Q2_sigma = calo_sigma_reco[2];
    Q2_esigma = calo_esigma_reco[2];

    hResoX_calo_electron->Fill((x_electron-x_truth)/x_truth);
    hResoX_calo_jb->Fill((x_jb-x_truth)/x_truth);
    hResoX_calo_da->Fill((x_da-x_truth)/x_truth);
    hResoX_calo_sigma->Fill((x_sigma-x_truth)/x_truth);
    hResoX_calo_esigma->Fill((x_esigma-x_truth)/x_truth);

    hResoY_calo_electron->Fill((y_electron-y_truth)/y_truth);
    hResoY_calo_jb->Fill((y_jb-y_truth)/y_truth);
    hResoY_calo_da->Fill((y_da-y_truth)/y_truth);
    hResoY_calo_sigma->Fill((y_sigma-y_truth)/y_truth);
    hResoY_calo_esigma->Fill((y_esigma-y_truth)/y_truth);

    hResoQ2_calo_electron->Fill((Q2_electron-Q2_truth)/Q2_truth);
    hResoQ2_calo_jb->Fill((Q2_jb-Q2_truth)/Q2_truth);
    hResoQ2_calo_da->Fill((Q2_da-Q2_truth)/Q2_truth);
    hResoQ2_calo_sigma->Fill((Q2_sigma-Q2_truth)/Q2_truth);
    hResoQ2_calo_esigma->Fill((Q2_esigma-Q2_truth)/Q2_truth);
  }
  // Drawing the histograms
  auto canvas_x_1D = new TCanvas();
  canvas_x_1D->Divide(3,2);
  canvas_x_1D->cd(1); hResoX_electron->Draw("hist");
  hResoX_calo_electron->SetLineStyle(2); hResoX_calo_electron->SetLineColor(kRed); hResoX_calo_electron->Draw("hist same");
  canvas_x_1D->cd(2); hResoX_jb->Draw("hist");
  hResoX_calo_jb->SetLineStyle(2); hResoX_calo_jb->SetLineColor(kRed); hResoX_calo_jb->Draw("hist same");
  canvas_x_1D->cd(3); hResoX_da->Draw("hist");
  hResoX_calo_da->SetLineStyle(2); hResoX_calo_da->SetLineColor(kRed); hResoX_calo_da->Draw("hist same");
  canvas_x_1D->cd(4); hResoX_sigma->Draw("hist");
  hResoX_calo_sigma->SetLineStyle(2); hResoX_calo_sigma->SetLineColor(kRed); hResoX_calo_sigma->Draw("hist same");
  canvas_x_1D->cd(5); hResoX_esigma->Draw("hist");
  hResoX_calo_esigma->SetLineStyle(2); hResoX_calo_esigma->SetLineColor(kRed); hResoX_calo_esigma->Draw("hist same");
  canvas_x_1D->cd(6);
  auto legend_x = new TLegend(0.1,0.1,0.9,0.9);
  legend_x->AddEntry(hResoX_electron, "E_{e} from tracker", "l");
  legend_x->AddEntry(hResoX_calo_electron, "E_{e} from ECAL", "l");
  legend_x->Draw();
  
  auto canvas_y_1D = new TCanvas();
  canvas_y_1D->Divide(3,2);
  canvas_y_1D->cd(1); hResoY_electron->Draw("hist");
  hResoY_calo_electron->SetLineStyle(2); hResoY_calo_electron->SetLineColor(kRed); hResoY_calo_electron->Draw("hist same");
  canvas_y_1D->cd(2); hResoY_jb->Draw("hist");
  hResoY_calo_jb->SetLineStyle(2); hResoY_calo_jb->SetLineColor(kRed); hResoY_calo_jb->Draw("hist same");
  canvas_y_1D->cd(3); hResoY_da->Draw("hist");
  hResoY_calo_da->SetLineStyle(2); hResoY_calo_da->SetLineColor(kRed); hResoY_calo_da->Draw("hist same");
  canvas_y_1D->cd(4); hResoY_sigma->Draw("hist");
  hResoY_calo_sigma->SetLineStyle(2); hResoY_calo_sigma->SetLineColor(kRed); hResoY_calo_sigma->Draw("hist same");
  canvas_y_1D->cd(5); hResoY_esigma->Draw("hist");
  hResoY_calo_esigma->SetLineStyle(2); hResoY_calo_esigma->SetLineColor(kRed); hResoY_calo_esigma->Draw("hist same");
  canvas_y_1D->cd(6);
  auto legend_y = new TLegend(0.1,0.1,0.9,0.9);
  legend_y->AddEntry(hResoY_electron, "E_{e} from tracker", "l");
  legend_y->AddEntry(hResoY_calo_electron, "E_{e} from ECAL", "l");
  legend_y->Draw();
  
  auto canvas_Q2_1D = new TCanvas();
  canvas_Q2_1D->Divide(3,2);
  canvas_Q2_1D->cd(1); hResoQ2_electron->Draw("hist");
  hResoQ2_calo_electron->SetLineStyle(2); hResoQ2_calo_electron->SetLineColor(kRed); hResoQ2_calo_electron->Draw("hist same");
  canvas_Q2_1D->cd(2); hResoQ2_jb->Draw("hist");
  hResoQ2_calo_jb->SetLineStyle(2); hResoQ2_calo_jb->SetLineColor(kRed); hResoQ2_calo_jb->Draw("hist same");
  canvas_Q2_1D->cd(3); hResoQ2_da->Draw("hist");
  hResoQ2_calo_da->SetLineStyle(2); hResoQ2_calo_da->SetLineColor(kRed); hResoQ2_calo_da->Draw("hist same");
  canvas_Q2_1D->cd(4); hResoQ2_sigma->Draw("hist");
  hResoQ2_calo_sigma->SetLineStyle(2); hResoQ2_calo_sigma->SetLineColor(kRed); hResoQ2_calo_sigma->Draw("hist same");
  canvas_Q2_1D->cd(5); hResoQ2_esigma->Draw("hist");
  hResoQ2_calo_esigma->SetLineStyle(2); hResoQ2_calo_esigma->SetLineColor(kRed); hResoQ2_calo_esigma->Draw("hist same");
  canvas_Q2_1D->cd(6);
  auto legend_Q2 = new TLegend(0.1,0.1,0.9,0.9);
  legend_Q2->AddEntry(hResoQ2_electron, "E_{e} from tracker", "l");
  legend_Q2->AddEntry(hResoQ2_calo_electron, "E_{e} from ECAL", "l");
  legend_Q2->Draw();
  

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