name: centred
layout: true
class: center, middle, inverse
---
# Diffusion Weighted Imaging Analysis
---
## Outline
### Background and Sequences
### Pre-processing steps
### Analysis Tools/Methods
---
layout: true
name: columns
class: inverse, middle
---
.left-column[
## Background
]
.right-column[
Diffusion Weighted Imaging sequences are a process where by acquiring many different images with different gradients we can get a measure of the amount of diffusivity across the brain. We can then fit a tensor (diffusion ellipsoid) that sums up all the information from each gradient at each voxel.
]
---
.left-column[
## Background
### - Sequences
]
.right-column[
Varying sequence components for diffusion can consist of:

* the number of directions

* whether a reverse phase encoded sequence is run

* the number of b0 measurements/repeat measurements

* whether the sequence was multi-shell
]
---
.left-column[
## Background
### - Sequences
### - Sequence Setup
]
.right-column[
A good heuristic is to acquire opposite phase encoded directions to help correct for eddy currents, distortions.

At least 40 directions should be acquired for b-values over 1000. Then time constraints/patient comfort and other factors will determine the number of directions, repeated measures and other factors.

A sample set of measurements might look something like:

* 1 P>A b0 as a separate sequence

* A second sequence with A>P, 2 sets of 64 b3000 measurements interspersed with b0 measurements.emphatic[*]

.footnote[
.emphatic[*]
This would give multiple measures at each gradient direction to help correct for any movement, and an opposite direction to correct for eddy current distortions
]
]
---
.left-column[
## Background
### - Sequences
### - Sequence Setup
### - Sources of Noise
]
.right-column[
Certain things to look out for when acquiring data:

* Eddy current distortions

* Movement can have a particularly large impact

* Other traditional MR artifact sources also affect quality.
]
---
template: centred
## First steps with the data
---
template: columns
.left-column[
## Pre-processing
### - Conversion
]
.right-column[
First step after receiving the DICOM files from the scanner is to convert them to a format that is recognised by the software you intend to use for the rest of your analysis steps. 

There are many conversion tools out there. A lot of analysis packages contain a conversion tool, or you can use a standalone tool.

The most widely used format, particularly for open source or freely available software is the nifti file format. The freely available `dcm2niix` package is one popular choice for conversion.
]
---
.left-column[
## Pre-processing
### - Conversion
### - Correction
]
.right-column[
Most analysis toolchains for diffusion data will have some method for correcting for subject movement and eddy current distortions, and perhaps susceptibility distortions as well.

NOTE: In some instances it may be necessary to manually rotate the original bvecs (gradient vector information) after distortion/motion correction.

With FSL current versions recommend a combination of the `eddy` and `topup` tools.
]
---
.left-column[
## Pre-processing
### - Conversion
### - Correction
### - Diffusion Tensor Fitting
]
.right-column[
This is another steps that most of the analysis packages have an implementation of. It is this step that generates Fractional Anisotropy maps, Mean Diffusivity maps and more.

FSL has a processing step called `dtifit` that takes the eddy current corrected data, a whole brain mask, usually generated with it's brain extraction tool `bet` which strips the skull and external data from a b0 image, and then outputs a range of files, including those mentioned above.
]
---
template: centred
###A sample FA map, coloured based on the direction of diffusion
.center[<img src="resources/FA-colour.png" width="500" height="341">]
---
.left-column[
## Pre-processing
### - Conversion
### - Correction
### - Diffusion Tensor Fitting
### - Co-registration
]
.right-column[
This involves registering either a b0 image from the diffusion sequence or the mean b0 if multiple b0 were acquired to a structural image (MPRAGE or similar).

Depending upon the later analysis steps this pre-processing step may or may not be necessary, however it is generally advisable as it allows individual diffusion results to be displayed overlaid on a higher resolution image, allowing the viewer to visualise position in the brain for significant features.

The method that FSL uses is to run a linear registration with 6 degrees of freedom.

An additional steps can be to register each participants structural image to a standard brain (a popular choice is the [MNI brain](https://www.mcgill.ca/bic/resources/brain-atlases/human)). This then allows the results from each individual brain to be compared in a standard space.
]
---
template: centred
## Further Analysis
---
.left-column[
## Analysis
### - FSL (1)
]
.right-column[
.center[### BedpostX]

There are two primary streams of analysis that are prominent within the FSL suite of analysis tools for diffusion data. The first of these is probabilistic tractography (`ProbtrackX`) and a pre-requisite for that is `BedpostX`.

`BedpostX` uses Bayesian estimation to generate a folder with a large selection of files which are used in the probabilistic tractography. It is able to model crossing fibres using the Markov Chain Monte Carlo sampling to build up distributions at each voxel.

The script expects a data file containing a 4D series of brain volumes (including both with and without diffusion weighting) as well as a brain mask (separating skull and external regions from the brain). Finally it also requires a bvecs & bvals file that contain a list of gradient directions and b-values respectively applied during the acquisition (and in the same order as the data).
]
---
.left-column[
## Analysis
### - FSL (1)
]
.right-column[
.centre[### BedpostX continued...]

The output files are 3D and 4D volumes primarily samples and means of the distributions on theta, phi and fractional anisotropy, means of distributions on diffusivity, T2w baseline signal intensity and PDD distribution and finally uncertainty on the estimated fibre orientation.

Advanced option that can be fed into the script include changing the default number of fibres modelled per voxel, adding burn-in for the MCMC process and alternative models for multi-shell data and others.
]
---
.left-column[
## Analysis
### - FSL (1)
]
.right-column[
.center[### ProbtrackX]

This script runs probabilistic tractography with crossing fibres and takes as input the files created by the `bedpostx` script. The main premise of the script is to generate connectivity distributions from user-specified seed voxels, creating a single image in the space of the specified seed.

All brain voxels in the resulting image will have values representing the number of samples that pass through the voxel from the seed region.
]
---
.left-column[
## Analysis
### - FSL (1)
]
.right-column[
.centre[### ProbtrackX Seed Regions]

* If the seed region is in a non-diffusion space then transformation matrices (for linear registration) or warp-fields (for non-linear registration) need to be provided.

* Seeds can be specified as a single voxel, a single mask (one contiguous region) or multiple masks (tracts must pass through all supplied masks)

* .emphatic[Targets] can also be specified. The different types are waypoint masks (tracts that pass through either all masks or at least one mask are selected), exclusion masks (pathways that enter an exclusion mask are terminated) and classification masks (which classify seed voxels based on which mask tracts intersect.)

#### Options

There are too many options to discuss fully here, but options include changing the number of samples used, limiting the curvature of pathways, pathway loop checking and many more.
]
---
.left-column[
## Analysis
### - FSL (1)
]
.right-column[
.center[### Further steps]

If you have run a classification target `probtrackx` analysis then the FSL diffusion toolkit provides a couple more scripts:

* `proj_thresh`: which converts seeds_to\_{target} values from the total numbers of samples to the proportion of samples reaching any target mask that is over a specified threshold.

* `find_the_biggest`: performs hard segmentation of the seed region based on the outputs of classification targets.
]
---
.left-column[
## Analysis
### - FSL (1)
]
.right-column[
.center[### Other utilities]

FSL provides two more utilities that act on 3D/4D images files and are very powerful for extracting numbers out of the resulting files. These also can be very useful with the next analysis stream.

* `fslmaths`: Has a huge array of mathematical operations that can be performed on the data, including, but certainly not limited to, thresholding, adding/multiplying/subtracting image, spatial filtering and basic statistical operations.

* `fslstats`: Runs statistics on supplied images, including max/min/mean/standard deviation, outputting co-ordinates of max/min voxels, histograms, centres-of-gravity and more.
]
---
template: centred
### ProbtrackX tract representation in 3D
.center[
<video width="500" height="500" controls>
<source src="resources/animation.mp4" type="video/mp4">
<source src="resources/animation.ogg" type="video/ogg">
Your browser does not support the video tag.
</video>
]

.imlabel[The display threshold has been adjusted to clarify the representation]
---
.left-column[
## Analysis
### - FSL (1)
### - FSL (2)
]
.right-column[
.center[### Tract Based Spatial Statistics]

TBSS is an alternative to traditional voxel-wise statistical analysis of fractional anisotropy (FA) images. It aims to solve the issues of arbitrariness of spatial smoothing extent, and choice of standard registration algorithms. The steps in the analysis are:

The process takes FA images (from our `dtifit` step), erodes the data slightly to remove outliers and registers each image to a standard space (FMRIB58_FA, a user supplied target or the most representative from the set).

After registration each individuals FA has the transform applied, and they are combined to create a mean FA image. This is run through a skeletonisation step which creates an alignment-invariant tract representation. Finally TBSS thresholds the mean FA skeleton image and computes a projection from each individuals data onto the skeleton. This is then fed into voxel-wise statistics in the next section.
]
---
.left-column[
## Analysis
### - FSL (1)
### - FSL (2)
]
.right-column[
.center[### Voxel-wise Statistics and more]

TBSS results can be fed into the FSL `randomise` tool. This runs non-parametric permutation inference on the neuroimaging data, generally by applying a general linear model design matrix. Some examples are one-sample T-tests, two-sample Unpaired T-test with nuisance variables and repeated measures ANOVA. It generally outputs raw test statistics, uncorrected P statistics and family-wise error rate corrected P-values.

Other options for TBSS analysis are:

* Feeding non-FA images in TBSS (for instance mean diffusivity)

* Using crossing-fibre measures (for instance from the `bedpostx` tool)

* Warping TBSS results back to native space to use for tractography or validation.
]
---
template: centred
### TBSS results for significant (P<0.5) differences
.center[<img src="resources/tbss_results_filled.png" width="500" height="335">]
.imlabel[Group comparison showing regions of increased FA from one group to another]
---
There are multiple ways to generate masks used for tractography, including:

* Use masks from a predefined/default atlas. Fast. Lacks precision.

* Use masks from a structural analysis of our participants (Freesurfer or similar)

* Use masks from another group that have scanned regions of interest in higher field strengths, e.g. 7T
---
# Final Thoughts
1. Normalisation for comparing between subjects
2. Thresholding for throwing away values with low probability of being on tract.
