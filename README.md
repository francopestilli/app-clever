[![Abcdspec-compliant](https://img.shields.io/badge/ABCD_Spec-v1.1-green.svg)](https://github.com/brain-life/abcd-spec)
[![brainlife.io/app](https://img.shields.io/badge/brainlife.io-app-green.svg)](https://brainlife.io/app/5b2975d616fe38002748e79a)

# app-clever

<!-- brainlife-app:begin (generated from the App manifest; edit the manifest, not this block) -->

> **Flags outlying timepoints in an fMRI run with the clever R package (PCA leverage, robust distance and DVARS) for scrubbing and quality control, and outputs an outlier plot and table.**

Masks the BOLD time series into a timepoints-by-voxels matrix, robustly normalises it, keeps high-variance and/or high-kurtosis principal components and measures leverage, robust distance and optionally DVARS for each timepoint, then thresholds them to flag outliers. Use it to choose volumes to scrub or to check fMRI data quality. Needs a 4D BOLD image and a brain mask in the same space. Note: main.R reads the images from undefined variables bold and mask (not input$bold / input$mask) and the parameter kurt_quantile is not in the list of names passed to clever (it expects kurt_quant), so the App may fail or ignore that parameter.

| | |
|---|---|
| 📥 **Inputs** | `neuro/func/task` · `neuro/mask` |
| 📤 **Outputs** | `raw` |
| 🐳 **Container** | `brainlife/clever:2.0` |
| ⚙️ **Resources** | 8 CPUs · 31 GB memory · 15 h walltime |
| 🔗 **Run it** | [doi.org/10.25663/bl.app.71](https://doi.org/10.25663/bl.app.71) |

#### 📥 Inputs

| Input | Datatype | Required | Description |
|---|---|---|---|
| `bold` | `neuro/func/task` | yes | The brain volume time-course to look for outlying timepoints in. |
| `mask` | `neuro/mask` | yes | The whole brain mask. Voxels to be retained for analysis should be 1; voxels to be discarded should be 0. |

#### 📤 Outputs

| Output | Datatype | Description |
|---|---|---|
| `clever` | `raw` | Outlier detection for high-dimensional data. |

<details>
<summary><b>⚙️ Parameters</b> (16)</summary>

| Parameter | Type | Default | Description |
|---|---|---|---|
| `projection` | enum | `variance` | `clever` works by first choosing PCs of the data which are likely to contain information about outliers. `kurtosis` selects PCs of high kurtosis among the PCs which explain the top 90% of variance. It is based on the observation that the presence of outliers often induces high kurtosis. `variance` selects PCs of greater-than-average variance, and is based on the observation that the larger PCs often contain outlier information, whereas smaller PCs often represent typical variation in the data. We recommend the `kurtosis` method over the `variance` method. `PCATF` (PCA Trend Filtering) works differently from the other two projection methods: it computes a variation of PCA which minimizes the zero norm of first-differences in each PC score. This has the effect of making PC scores constant within neighborhoods of little change, thus highlighting regions of artifact presence. It is based on the observation that timepoints with artifacts tend to be adjacent (due to an artifact source persisting or otherwise affecting the fMRI across multiple scans). If multiple projections are chosen, each projection will be used with each compatible measure of outlyingness. Options: `kurtosis`, `variance`, `PCATF`, `kurtosis, variance`, `kurtosis, PCATF`, `variance, PCATF`, `kurtosis, variance, PCATF`. |
| `out_meas` | enum | `leverage` | After projecting the data, `clever` computes an outlyingness score for each observation. `leverage` is based on a regression model using the PC scores as a design matrix. `robdist` is a robust measure of the minimum covariance determinant (MCD) distance. It is currently being revamped; for now, we recommend the `leverage` option. Options: `leverage`, `robdist`, `leverage, robdist`. |
| `DVARS` | enum | `TRUE` | DVARS is another common measure of outlyingness for fMRI data based on the standard deviation of the voxels' change from the previous timepoint. Unlike the other methods, it is not based on first projecting the data. Options: `FALSE`, `TRUE`. |
| `detrend` | enum | `TRUE` | Should the PCs be detrended before measuring the outlyingness of each observation? Detrending is highly recommended for fMRI data for two reasons. First, temporal trends induce positive or negative kurtosis, contaminating the connection between high kurtosis and outlier presence. Second, trends tend to reduce the size of the in-MCD subset for the robust distance method causing many false positives. In addition to `TRUE` and `FALSE`, a third option `kurtosis` can be used to only detrend the PCs for the purpose of measuring kurtosis, and not for the actual outlyingness measurement. This option will not affect the PCATF projection, which is never detrended. Options: `TRUE`, `FALSE`, `kurtosis`. |
| `kurt_quantile` | string | `0.95` | A number between 0 and 1. Only applicable for the `kurtosis` projection method. PCs of kurtosis greater than this quantile will be retained. The quantile will be based on the sampling distribution of kurtosis for Normal, independent data (estimated by simulation or with the theoretical asymptotic distribution). The default value of 0.95 was chosen because the kurtosis of PCs with outliers should easily surpass this quantile, and 0.05 is a reasonable rate for false positives (retaining PCs without outlier information) under the null hypothesis of no outliers. Note that at least one PC will be retained in each projection, regardless of the value of `kurt_quantile`. |
| `id_outliers` | enum | `TRUE` | Should outliers be identified? Options: `TRUE`, `FALSE`. |
| `lev_cutoff` | string | `4` | The outlier cutoff value for leverage, as a multiple of the median leverage. |
| `rbd_cutoff` | string | `0.9999` | The outlier cutoff quantile for MCD distance. |
| `lev_images` | string | `FALSE` | Leverage images are currently not supported by the `clever` BrainLife app. |
| `verbose` | enum | `FALSE` | Should occasional updates be printed? Default is `FALSE`. Options: `TRUE`, `FALSE`. |
| `main` | string |  | Override title for output plot(s). |
| `sub` | string |  | Override subtitle for output plot(s). |
| `xlab` | string |  | Override x-axis label for output plot(s). |
| `ylab` | string |  | Override y-axis label for output plot(s). |
| `png` | string | `cleverPlot.png` | Name of file to save plot to. |
| `csv` | string | `cleverTable.csv` | Name of file to save table to. |

</details>

📚 **References used by this App:**
- Afyouni S, Nichols TE. Insight and inference for DVARS. NeuroImage (2018). [doi:10.1016/j.neuroimage.2017.12.098](https://doi.org/10.1016/j.neuroimage.2017.12.098)

<!-- brainlife-app:end -->

`clever` computes principal Components LEVERage (“CLEVER”) and other measures of outlyingness for high-dimensional data such as fMRI data. In BrainLife, it can be used for flagging timepoints which are likely to contain artifacts ("scrubbing") as well as general quality-control of fMRI data.

### How it works

As input, `clever` takes an fMRI volume `"bold.nii.gz"` and brain mask `"mask.nii.gz"`, and applies the mask to form a ***T*** timepoints by ***V*** voxels matrix, ***Y***. In our case, ***Y*** represents an fMRI run: each row is a vectorized volume, and each column represents one timepoint. Next, the algorithm performs the following steps: 

1. Normalize ***Y*** by centering and scaling its columns robustly.

2. Perform PCA on the normalized ***Y*** matrix using the singular value decomposition (SVD) in order to obtain the ***T **x** T*** PC score matrix, ***U***.

3. Some PCs are removed from ***U*** to obtain the ***Q*** *x* ***T*** (***Q*** *<* ***T***) matrix, ***U'***. The PCs which are retained are those likley to contain outlier information: PCs with greater-than-average variance, or PCs with both greater-than-average variance and high kurtosis. (The removal of at least one PC is also a theoretical requirement for leverage; additionally, the robust distance method requires ***Q*** to be appropriately small relative to ***T***.)

4. Next, leverage and/or robust distance is measured. The output of each is a length ***T*** vector representing the "outlyingness" of each time point.  

5. The outlyingness measures are thresholded to identify the set of outlying timepoints. 

We also include the DVARS outlier detection method in `clever`. It normalizes the data as explained in step 1, but otherwise follows the algorithm as described in the [paper (Afyouni and Nichols, 2018)](doi.org/10.1016/j.neuroimage.2017.12.098) and implemented in the [MATLAB code](https://github.com/asoroosh/DVARS) provided by its authors.

### Authors

- [Amanda Mejia](afmejia@iu.edu)
- [John Muschelli](jmusche1@jhu.edu)
- [Damon Pham](ddpham@iu.edu)

### Funding Acknowledgement

brainlife.io is publicly funded and for the sustainability of the project it is helpful to Acknowledge the use of the platform. We kindly ask that you acknowledge the funding below in your publications and code reusing this code.

[![NSF-BCS-1734853](https://img.shields.io/badge/NSF_BCS-1734853-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1734853)
[![NSF-BCS-1636893](https://img.shields.io/badge/NSF_BCS-1636893-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1636893)
[![NSF-ACI-1916518](https://img.shields.io/badge/NSF_ACI-1916518-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1916518)
[![NSF-IIS-1912270](https://img.shields.io/badge/NSF_IIS-1912270-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1912270)
[![NIH-NIBIB-R01EB029272](https://img.shields.io/badge/NIH_NIBIB-R01EB029272-green.svg)](https://grantome.com/grant/NIH/R01-EB029272-01)

### Citations

We kindly ask that you cite the following articles when publishing papers and code using this code. 

1. Avesani, P., McPherson, B., Hayashi, S. et al. The open diffusion data derivatives, brain data upcycling via integrated publishing of derivatives and reproducible open cloud services. Sci Data 6, 69 (2019). [https://doi.org/10.1038/s41597-019-0073-y](https://doi.org/10.1038/s41597-019-0073-y)

# Running the App 

### On Brainlife.io

You can submit this App online at [https://doi.org/10.25663/bl.app.71](https://doi.org/10.25663/bl.app.71) via the "Execute" tab.

### Running Locally (on your machine)

For local use, we recommend using the R package directly. See the README at https://github.com/mandymejia/clever/tree/2.0 for installation instructions and a tutorial.

### Sample Datasets

`clever` includes two sample datasets, `Dat1` and `Dat2`. These are fMRI time series slices which have been vectorized (are ***T*** **x** ***V***).

### Output

All output files will be generated under the current working directory. The outputs of this App are `product.json` which plots the timecourse of each measure of outlyingness interactively using `plotly`, `cleverPlot.png` which plots the timecourse of each measure of outlyingness using the original plot method included with `clever`, and `cleverTable.csv` which stores each measure and outlier indication along the columns.

### Dependencies

This App only requires [singularity](https://www.sylabs.io/singularity/) to run. If you don't have singularity, you will need to install following dependencies.  

  - R: https://www.r-project.org/
  - The `clever` package itself: https://github.com/mandymejia/clever  
  - R package "Imports" listed here: https://github.com/mandymejia/clever/blob/master/DESCRIPTION
  - The `glmgen` R package: https://github.com/glmgen/glmgen
  - These other R packges: `oro.nifti`, `rjson`, `ggplot2`, and `cowplot`.

### GPL-v3 Copyright 