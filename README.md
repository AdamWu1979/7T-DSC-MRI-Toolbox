# :mag_right: :bulb: 7T-DSC-MRI-Toolbox :flashlight: :wrench: </font>

<p align="center">
This Python-based toolbox provides functions and pipeline to process Dynamic Susceptibility Contrast (DSC) MRI data acquired at 7T. A dataset of DSC MRI acquired in the spinal cord of a healthy volunteer as published in the reference below (healthy volunteer HC1) is also provided. More particularly, this toolbox provides functions to extract acquisition times from physiologs (Siemens format) for cardiac-gated acquisitions, correct signal by effective TR and discard inconsistent TRs as well as to filter breathing frequencies in the signal based on the signal measured by any respiratory device in a repiration physiolog (Siemens format) are available.</p>

We thank you for choosing our toolbox! :heart: According to the Apache license 2.0, please cite the following reference:
> **Lévy S, Roche P-H, Callot V. Dynamic Susceptibility Contrast imaging at 7T for spinal cord perfusion mapping in Cervical Spondylotic Myelopathy patients, In: *Proc. Intl. Soc. Mag. Reson. Med. 28*. 2019;3195.**

This toolbox also provides functions and pipeline to fit a Gamma-variate function to the bolus and derive relative perfusion metrics. These functions and pipeline were adapted from the functions provided by the great toolbox [Dynamic Susceptibility Contrast MRI toolbox](https://github.com/marcocastellaro/dsc-mri-toolbox) and are therefore written in Matlab. Thank you for developing those very useful functions!

---

# Table of Contents

- [System requirements](#system-requirements)
- [Code description](#code-description)
- [Data description](#data-description)
- [Funding](#funding)

---

# System requirements

The Python programs work with Python3.6 and require the following libraries: nibabel, numpy, scipy.io, argparse, \_pickle, sys, os
The code has been developed under MacOSX and should easily work under Linux (although not tested yet) but similar performance cannot be promised so far under Windows systems.

---

# Code description

This toolbox includes Python scripts that be run directly from a bash environment as well as Python functions that are called by the scripts. Here is an overview of those functions:
  - `dsc_image2concentration.py`: this program converts a time-series 4D volume to contrast agent concentration values after voxel-wise signal processing
  - `dsc_process_signal_by_slice.py`: this program performs similar processing as `dsc_image2concentration` but on the mean signal in a given region of interest and by slice
  - `dsc_dataQuality.py`: this program provides temporal SNR maps and signal-time profile of a given region of interest for a 4D volume
  - `dsc_utils.py`: this function provides the main methods for DSC signal processing and filtering that are called by the scripts above
  - `dsc_pipelines.py`: this function proposes methods calling methods for signal processing from `dsc_utils`
  - `dsc_extract_physio.py`: this function provides methods to extract acquisition times of each repetition in times series acquired on Siemens systems provided a physiolog file.

As explained in the introduction, this toolbox also provides functions and pipelines to derive perfusion metrics, which were adapted from the great [Dynamic Susceptibility Contrast MRI toolbox](https://github.com/marcocastellaro/dsc-mri-toolbox) and are therefore written in Matlab. Here is an overview of those functions:
  - `dsc_fitGamma2imgConc.m`: this program takes a 4D volume (of ∆Concentration or ∆R2(\*) as produced by `dsc_image2concentration`) and fits a Gamma-variate function voxel-wise
  - `dsc_fitGamma2listConc.m`: this program takes .mat file with a list of time-signals (of ∆Concentration or ∆R2(\*) as produced by `dsc_process_signal_by_slice`) and fits a Gamma-variate function to each of these time-signals
  - `dsc_fitGamma.m`: this function takes a time-tignal as input and fit a Gamma-variate function to it (this function is called by `dsc_fitGamma2imgConc` and `dsc_fitGamma2listConc`)
  - `dsc_calculatePerfMetricsVoxelWise.m`: this program takes as input the .mat files generated by `dsc_image2concentration` and `dsc_fitGamma2imgConc` and calculate relative perfusion metrics voxel-wise
  - `saveAsNifti.m`: this functions is used to generate a Nifti file from 3D matrices
  - `dsc_map_rBV_rBF.m` and `dsc_map_BAT_TT_TTP_rTTP_PR.m` are called by `dsc_calculatePerfMetricsVoxelWise` to calcualte relative Blood Volume, relative Blood Flow, Bolus Arrival Time, Transit Time, Time-to-Peak from t=0, Time-to-Peak from bolus arrival time and Peak Ratio
  - `dsc_calculate_BAT_TT_TTP_rTTP.m`: this function is used in `dsc_map_BAT_TT_TTP_rTTP_PR` to calculate Bolus Arrival Time, Transit Time, Time-to-Peak from t=0, Time-to-Peak from bolus arrival time.

Some of the programs of the toolbox are further detailed below.

## dsc_image2concentration

This program takes an Nifti 4D image as input and converts it to the value of variation of the contrast agent concentration value (∆C). Below are the available options:
```
required arguments:
  -i IFNAME             Path to MRI data file.
  -m MASKFNAME          NIFTI volume defining the region of interest.
  -physio PHYSIOLOGFOLDER
                        Path to folder including the physiologs.
  -o OFNAME             Filename for output image and mat file.

optional arguments:
  -h, --help            show this help message and exit
  -param PARAMFILEPATH  Path to file giving specific parameters (injection
                        repetition, dicom path).
  -r2 R2GDINBLOOD       Transverve relaxivity (in s-1.mmol-1.L = s-1.mM-1) of
                        Gadolinium in blood. Default = 3.55 s-1.mmol-1.L (from
                        Proc. Intl. Soc. Mag. Reson. Med. 16 (2008) 1457)
```

Note that if you use `-r2 1`, the obtained value will correspond to the variation of relaxation rate (R2 or R2* depending on whether acquired data are spin-echo or gradient-echo) along time instead of the variation of contrast agent concnetration

## dsc_process_signal_by_slice

This program performs the same processing as `dsc_image2concentration` but on the mean signal within a given region of interest (input of `-m` flag), all slices averaged and slice-by-slice. The results are saved as a .mat file and a pickle file. Below are the available options:
```
required arguments:
  -i IFNAME             Path to MRI data file.
  -m MASKFNAME          NIFTI volume defining the region of interest.
  -l PHYSIOLOGFNAME     Basename of physio log for Pulse Ox and Respiration.
  -o OFNAME             Filename for the output plots and data.

optional arguments:
  -h, --help            show this help message and exit
  -inj INJECTIONREP     Number of the repetition when contrast agent injection
                        was launched.
  -s FIRSTPASSSTARTTIME
                        Start time (on original time grid) of first pass (in
                        seconds).
  -e FIRSTPASSENDTIME   Time (on original time grid) of first pass end (in
                        seconds).
  -te TE                Echo time in milliseconds.
  -r2 R2GDINBLOOD       Transverve relaxivity (in s-1.mmol-1.L = s-1.mM-1) of
                        Gadolinium in blood. Default = 3.55 s-1.mmol-1.L [from
                        Proc. Intl. Soc. Mag. Reson. Med. 16 (2008) 1457]
```

## dsc_dataQuality

This program plots ∆R2 along time, mean image and temporal SNR maps from a 4D time-series volume.

```
required arguments:
  -i IMGFNAME         Path to NII data file.
  -m MASKFNAME        Path to NII mask file.
  -dcm DCMFOLDER      Dicom folder path.
  -p PHYSIOLOGFOLDER  Physiologs folder path.
  -l DATALABEL        Label or title for the plot.

optional arguments:
  -h, --help          show this help message and exit
  -b BASELINEENDREP   Last repetition of the baseline.
 ```
 

---

# Data description

In the **data** you will find a dataset (*se_epi_hc1.nii.gz*) acquired with a spin-echo single-shot EPI sequence within the spinal cord of a healthy volunteer (HC1 in the reference above) at 7T and with contrast agent bolus injection (this dataset has already been processed to address Gibbs ringing artifacts, motion, noise and distortions using `mrdegibbs` from MRTrX toolbox, rigid transformations with ANTs, BM4D and FSL Topup respectively).

The file *maskForCalculations.nii.gz* is a simple 3D mask that can be used with `dsc_image2concentration` to restrict calculations around the spinal cord to reduce computation time.

You will also find a folder named **physiolog** containing the physiolog files of the pulse oximeter (*slr_Physiolog_hc1.puls*) and of the respiratory belt (*slr_Physiolog_hc1.resp*) which are used for effective TR normalization and breathing frequencies filtering.

Finally, the folder **results_you_should_get** provides the maps of Bolus Arrival Time (*BAT.nii.gz*), relative Blood Flow (*rBF.nii.gz*), relative Blood Volume (*rBV.nii.gz*) and Time-to-Peak (*rTTP.nii.gz*).


---

# Funding


*This work was performed within the [CRMBM-CEMEREM](http://crmbm.univ-amu.fr/) (UMR 7339, CNRS / Aix-Marseille University), which is a laboratory member of France Life Imaging network (grant #ANR-11-INBS-0006). The project received funding from the European Union’s Horizon 2020 research and innovation program (Marie Skłodowska-Curie grant agreement #713750), the Regional Council of Provence-Alpes-Côte d’Azur, A\*MIDEX (#ANR-11-IDEX-0001-02, #7T-AMI-ANR-11-EQPX-0001, #A\*MIDEX-EI-13-07-130115-08.38-7T-AMISTART) and CNRS (Centre National de la Recherche Scientifique).*

