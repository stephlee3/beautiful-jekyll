---
layout: post
permalink: /posts/strucMRInotes
title: Intro to Neurohacking notes
---

> Neurohacking is the continuous process of using, improving and designing the simplest open source scripted software that depends on the minimum number of software platforms and is dedicated to improving the correctness, reproducibility, and speed of neuroimage data analysis.


# Theme 1: Overview and Set Up

- Structural MRI:
  - High spatial resolution
  - Reveals the anatomic structure of soft tissues
  - Used extensively in clinical and research practice
  - Versatile: different contrasts can target different tissue types : **FLAIR, T1, T2**
  - Sensitive to pathology (e.g. brain cancer, Multiple Sclerosis lesions)

---

# Theme 2: Data Structure and Operations
## 2.1 DICOM
DICOM format, two components:
- image data (in pixels, matrix)
- header (meta-data)

Note: One DICOM file = one "slice" of the brain.

Use the function `readDICOM` in `oro.dicom` package in R.

summary: Image data are stored as collection of 2D slice files for DICOM format.

## 2.2 NIfTI
NifTI:
- standardized representation of images
- 3D array: stacking individual slices on top of each other.

DICOM vs NIfTI

|        | DICOM  | NifTI  |
| ------ | ------ | ------ |
| File extenstion | .dcm | .nii.gz |
| File representation | one slice of the brain | 3D image of the brain|
| Header contains | Many information | image meta-data, no patient info|
| Storage | Different folders per subject, complex data structures | Different files can be in the same directory|

Some useful functions to process NifTI format:
- Use the function `dicom2nifti` in the `oro.dicom` package to convert DICOM format to NIfTI format.
- Use the `writeNifTI` and `readNIfTI` functions in the `oro.nifti` package to write and read NIfTI files.

## 2.3 Visualization
Use the following functions from `oro.nifti` packages to visualize the brain images.
- `image`: display NifTI data
- `orthographic`: display 3-planes of an image.
- `overlay`: display overlay of two images, NAs are not plotted. (We typically create a mask image before the overlay)

## 2.4 Data Manipulation
- masking
- basic operations (addition, subtraction, multiplication)

## 2.5 Transformation and Smoothing
### Transformation
- Linear/ spline transfer functions
- Used for better visualization, prediction and input into a standard software.

### Smoothing
- Use `AnalyzeFMRI::GaussSmoothArray` to apply a Gaussian smooth to the image.

## 2.6 Basic MRI contrasts
- T1
- T2
- FLAIR

---

# Theme 3: Preprocessing

## Overview:
- Pre-processing pipelines
- Inhomogeneity correction
- Skull stripping
- Registration: rigid/affine/nonlinear
- Intensity normalization

## 3.1 Preprocessing
### Pipeline
`DICOM` -> `NIfTI` -> `N3 Correction` -> `Skull Strip` -> `Registration`

### Inhomogeneity
The probability distribution function of tissue class intensities should not depend on the spatial localization of the tissue.

### Skull-stripping
Remove extra-cerebral voxels from the volume.

## 3.2 Image Registration
<span id="jump"> </span>
###  Registration
A spatial transformation of images with the goal of making locations (voxels, ROIs) have the same or similar interpretation.

### Co- Registration
Co-register volumes from different modalities to one another.
- Cross-sectional between-modalities
- Longitudunal within-modality
- Longitudinal between-modalities

### Registration to a template
- MNI template
- JHU_Eve template

### Complexity
- rigid (6df)
- affine (12df)
- nonlinear (>12df)

#### Rigid
- T_rigid (v)  = Rv + t
- R is rotation matrix, and t is translation vector.

#### Affine
- T_affine(v) = Av + t
- A is not constrained to be a rotation matrix.


## 3.3 fslr
- Virtual machine
- `fslr` package

## 3.4 Inhomogeneity correction using fslr
- Use function `fslr:: fsl_biascorrect`
```r
fast_img = fsl_biascorrect(nim, retimg = TRUE)
```

## 3.5 Brain extraction using fslr
- Use function `fslr:: fslbet`
```r
bet_fast = fslbet(infile = fast_img, retimg= TRUE)
```

## 3.6 Image registration using fslr
- From FSL: FLIRT(FMRIB's Linear Image Registration Tool) is an automated and robust tool for linear(rigid, affine) brian image registration.
- Use function `fslr:: flirt`

```r
## rigid
registered_fast = flirt(infile= bet_fast2, reffile = template, dof = 6, retimg = TRUE)

## affine
reg_fast_affine= flirt(infile= bet_fast2, reffile = template, dof = 12, retimg = TRUE)
```

Use function `fslr::fnirt_with_affine` to perform nonlinear image registration and affine registration before that.

```r
fnirt_fast= fnirt_with_affine(infile= bet_fast2, reffile = template, outfile= "FNIRT_to_Template", retimg = TRUE)
```

## 3.7 ANTsR
- `AnTsR`: port of `ANTS` into `R` using `Rcpp`
- focus on image Inhomogeneity correction (N3 and N4), image registration

## 3.8 Basic Data Manipulation with ANTsR
- reading images:
`antsImageRead`. Note: the extension of the filename (.nii.gz) must be specified; the dimension of the image must be supplied (2, 3, or 4)

- ANTsR images: the `antsImage` class
- Basic Statistics: `mean`, `as.array` etc
- ANTs image class: from `ANTsR` to `oro.nifti` class, using the package `extrantsr`.

## 3.9 Preprocessing with ANTsR
- Bias field correction (N4)
Use function `extrantsr:: bias_correct` for correction

```r
n3img = bias_correct(nim,correction = "N3", retimg = TRUE)
n4img = bias_correct(nim,correction = "N4", retimg = TURE)
```


- Registration
Use function `ants_regwrite` from package `extrantsr` to register the image to the template

```r
reg_n4 = ants_regwrite(filename= n4img, template.file = template, remove.warp = TRUE, typeofTransform = "Rigid")

```
---

# Theme 4:  Registration, ROI quantification, segmentation

## 4.1 Co-registration
See [section 3.2](#jump)

## 4.2 fslr co-registration
Use `flirt` function from `fslr` package to register the T2w(infile) to the T1 (reffile)

## 4.3 AnTsR co-registration
Use `ants_regwrite` from `extrantsr` package to register the T2(filename) to the T1 (template.file).

## 4.4 wrapper functions
- Use function `preprocess_mri_within` from `extrantsr` function to do the following steps:
  - Inhomogeneity Correction
  - Registration of the files to the first filename

## 4.5 Across-visit co-regisration of T1 images
- Visit 1 files are in the visit 1 T1 space and visit 2 files are in the visit 2 T1 space.
- Register the follow-up/visit 2 T1 scan to the visit 1/ baseline scan.

### Overview
- Registration within a subject
  - `ants_regwrite` wraps around the reading/writing of images and applying transformations
  - `double_ortho` and `ortho2` can provide basic visual checks.


- When images are registered in the space space, we can perform the following operations:
    - masking with a brain mask
    - transforming images to new spaces with one modality



## 4.6 Rigid Registration to T1

### Overall Framework
`FLAIR Image` + `Tumor ROI` -> `T1 weighted image` -> `Template Image`

The first is rigid registration, the second is nonlinear registration.



## 4.7 Affine registration of T1 to Template
Use function `ants_regwrite`, where `typeofTransform = "Affine"`.

## 4.8 Nonlinear registration of T1 to template
Use function `ants_regwrite`, where `typeofTransform = "SyN"`.

SyN: symmetric image normalization, a symmetric diffeomorphic registration technique.

## 4.9 Getting ROI anatomic info from non-linear registration
- Motivation: we have the ROI in the template space, and want to infer its location by leveraging the atlas labels.
- Dealing with non-binary ROI
  - Threshold the ROI
  - Use a weighted sum over the ROI

- Quantifiying the ROI engagement: tumor by region / region by Tumor



## 4.10 Tissue-Level Segmentation
We show how to use fslr to
- segment CSF,GM, and WM
- calculate the volume of CSF, GM, and WM.

### Functions
- readNIfTI
- bias_correct
- fslbet_robust: performs skull-stripping
- fast: calls fast from FSL, does Segmentation
- ortho2: does orthographic display

The function `fast` returns probability maps for corresponding tissue classes: pve_0, pve_1, pve_2
- `fast==1` represents CFS
- `fast==2` represents grey matter
- `fast==3` represents white matter


---
## Reference
[Imaging in R](http://johnmuschelli.com/imaging_in_r/index.html)

[Introduction to Neurohacking in R](https://www.coursera.org/learn/neurohacking)
