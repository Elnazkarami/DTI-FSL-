# Trigeminal Nerve Tractography Using FSL

This repository contains a comprehensive workflow for performing **trigeminal nerve tractography** using the **FMRIB Software Library (FSL)**. The goal of this project is to map the trigeminal nerve and analyze its structural connectivity using diffusion tensor imaging (DTI) data.

## Project Overview

Tractography is a powerful technique in neuroimaging for visualizing and analyzing the brain's white matter tracts. The **trigeminal nerve** is the fifth cranial nerve, critical for facial sensation and motor functions like chewing. This project utilizes DTI data to map the trigeminal nerve, providing insights into its connectivity and structural integrity. This is mainly used for projects studying orofacial chronic pain. 

### **Key Features:**

- **Preprocessing Pipelines**: Includes scripts for data preprocessing, including skull stripping and correction for eddy currents and motion artifacts.
- **Tractography Workflow**: Uses FSL tools (`dtifit`, `bedpostx`, `probtrackx2`) for diffusion modeling and probabilistic tractography.
- **Customizable Analysis**: Scripts can be adapted for different regions of interest (ROIs) or other cranial nerves.
- **Detailed Visualization**: Provides options for visualizing tractography results using FSL tools (`FSLView`, `FSLeyes`).

##  Repository Contents

- **`scripts/`** : Bash scripts for preprocessing, tensor fitting, and tractography.
- **`masks/`** : Seed and exclusion masks used for tractography.
- **`sample_data/`** : Example diffusion data (if applicable or allowed).
- **`results/`** : Sample outputs, including tracts and diffusion metrics.
- **`README.md`** : Project documentation and instructions.
- **`LICENSE`** : Licensing information.

##  Requirements

To run this project, you will need:

- **FSL (FMRIB Software Library)**: [Installation Guide](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FslInstallation)
- A Linux or macOS environment
- Basic knowledge of Bash scripting and FSL tools

##  Workflow

### 1. **Preprocessing the DTI Data**

Start by preprocessing the DTI data, including skull stripping and correcting for eddy currents and motion artifacts:


# Extract the b0 image and perform brain extraction
fslroi dwi_data.nii.gz nodif 0 1
bet nodif nodif_brain -m -f 0.3

# Correct for eddy currents and motion
eddy_correct dwi_data.nii.gz dwi_data_ec 0

# Create a brain mask
bet dwi_data_ec.nii.gz dwi_data_ec_brain -m


### 2. Fit the diffusion tensor model
dtifit -k dwi_data_ec_brain.nii.gz \   # Corrected DWI data
       -o dti \                        # Output prefix for DTI metrics (FA, MD, etc.)
       -m dwi_data_ec_brain_mask.nii.gz \  # Brain mask
       -r bvecs \                      # Diffusion gradient directions
       -b bvals                        # b-values file


### 3. Run Bedpostx for Fiber Orientation Modeling
 
    Prepare data for probabilistic tractography using `bedpostx`.

mkdir bedpostx_data
cp dwi_data_ec.nii.gz bedpostx_data/data.nii.gz
cp bvals bedpostx_data/bvals
cp bvecs bedpostx_data/bvecs
cp dwi_data_ec_brain_mask.nii.gz bedpostx_data/nodif_brain_mask.nii.gz

bedpostx bedpostx_data

### 4. **Define the Seed Region**

Create a seed mask for the trigeminal nerve using `FSLView` or `FSLeyes`.

fslmaths trigeminal_seed_mask.nii.gz -bin trigeminal_seed_mask_bin.nii.gz

### 5. **Perform Probabilistic Tractography**

Run `probtrackx2` to perform probabilistic tractography from the trigeminal nerve seed region.

probtrackx2 -x trigeminal_seed_mask_bin.nii.gz -l --onewaycondition --omatrix1 \
--targetmasks=target_masks_list.txt --os2t --s2tastext --avoid=exclusion_mask.nii.gz \
--dir=tractography_output --forcedir --opd --samples=bedpostx_data.bedpostX/merged \
--mask=bedpostx_data.bedpostX/nodif_brain_mask --xfm=bedpostx_data.bedpostX/xfm \
--invxfm=bedpostx_data.bedpostX/inverse_xfm


##  Results

- **Tractography Outputs**: The output includes connectivity matrices, probability maps, and tract density images.
- **Quantitative Metrics**: Extracts diffusion metrics along tracts, such as FA, MD, and axial/radial diffusivity.

## Visualization

Visualize the results using FSL tools:

- **FSLView**: Legacy tool for visualization.
- **FSLeyes**: A modern viewer with enhanced features.

## Example commands to visualize tracts:

fsleyes dwi_data.nii.gz tractography_output/fdt_paths.nii.gz





