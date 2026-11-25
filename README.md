<h1 align="center">anatomix-t1w-dwi-registration</h1>

<p align="center">
  Registers a DWI series to a T1-weighted image by aligning its mean b0 with anatomix deep features and FireANTs deformable registration, outputting the registered b0 and an ANTs-convention warp.
</p>

<p align="center">
  <a href="https://github.com/brainlife/abcd-spec"><img src="https://img.shields.io/badge/ABCD_Spec-v1.1-green.svg" alt="ABCD Spec v1.1"></a>
  <a href="https://doi.org/10.25663/brainlife.app.972"><img src="https://img.shields.io/badge/Brainlife-brainlife.app.972-blue.svg" alt="Run on brainlife.io"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-see_LICENSE-lightgrey.svg" alt="License"></a>
</p>

## At a glance

<!-- brainlife-app:begin (generated from the App manifest; edit the manifest, not this block) -->

> **Registers a DWI series to a T1-weighted image by aligning its mean b0 with anatomix deep features and FireANTs deformable registration, outputting the registered b0 and an ANTs-convention warp.**

Averages the DWI volumes at or below b0_max_bval into a b0 image, registers it onto the T1w with anatomix-register.py (modality-agnostic anatomix features driving FireANTs, affine then deformable by default) and writes the resampled b0 plus the forward and optional inverse displacement fields in ITK/ANTs convention. Use the warp to bring FA/MD or other DWI-space maps into T1w space, or the inverse to bring T1w-space parcellations into DWI space. A GPU is strongly recommended; the transform must end in a deformable stage.

| | |
|---|---|
| **Inputs** | `neuro/anat/t1w` · `neuro/dwi` |
| **Outputs** | `neuro/dwi/b0` · `neuro/transform/nifti` |
| **Container** | `gamorosino/anatomix-fireants:latest` |
| **Resources** | 32 GB memory · 4 h walltime · GPU |
| **Run it** | [doi.org/10.25663/brainlife.app.972](https://doi.org/10.25663/brainlife.app.972) |

#### Inputs

| Input | Datatype | Required | Description |
|---|---|---|---|
| `t1` | `neuro/anat/t1w` | yes | T1-weighted anatomical image used as the fixed image. |
| `dwi` | `neuro/dwi` | yes | DWI series with bvals and bvecs; its mean b0 is the moving image. At least one volume with b-value at or below b0_max_bval. |

#### Outputs

| Output | Datatype | Description |
|---|---|---|
| `b0_reg` | `neuro/dwi/b0` | Mean b0 volume extracted from the DWI series, registered (resampled) onto the T1w grid. |
| `warp` | `neuro/transform/nifti` | FireANTs transform (ITK/ANTs LPS-mm displacement field convention, not FSL) that resamples the DWI/b0 (moving) image onto T1w (fixed) space -- apply it to other DWI-space volumes (e.g. FA, MD maps) to bring them into T1w space. With save_inverse enabled, inverse-warp.nii.gz does the reverse (T1w -> native DWI space). |

<details>
<summary><b>Parameters</b> (4)</summary>

| Parameter | Type | Default | Description |
|---|---|---|---|
| `b0_max_bval` | number | `50` | b-value (s/mm^2) at or below which a DWI volume is treated as a b0 and averaged to build the moving image. |
| `transform` | string | `affine,deformable` | Comma-separated registration stages, in order, from rigid/affine/deformable (see anatomix-register.py --help). Must end in a deformable stage: this app's warp output is a NIfTI displacement field, which anatomix-register.py only writes when the last stage is deformable. affine,deformable handles gross T1w/DWI misalignment plus local distortion; use deformable alone if the images are already roughly aligned. |
| `save_inverse` | boolean | `true` | Also compute the moving-to-fixed (DWI/b0 -> T1w) inverse transform, e.g. to bring a T1w-space parcellation into native DWI space for tractography ROIs. |
| `device` | string | `auto` | Compute device for FireANTs: auto, cpu, cuda, or cuda:N. |

</details>

<!-- brainlife-app:end -->

## Running the App

### On brainlife.io

Open the App at [doi.org/10.25663/brainlife.app.972](https://doi.org/10.25663/brainlife.app.972), choose `neuro/anat/t1w`, `neuro/dwi` from your project, set the parameters and press **Execute**. The results are saved to your project as `neuro/dwi/b0`, `neuro/transform/nifti`.

### Locally

You need [Singularity/Apptainer](https://apptainer.org/) (the App runs in `gamorosino/anatomix-fireants:latest`), [`jq`](https://jqlang.github.io/jq/).

1. Clone this repository.
2. Create `config.json` with the paths to your inputs and the parameters:

   ```json
   {
       "t1": "/path/to/t1/t1",
       "dwi": "/path/to/dwi/dwi",
       "bvals": "/path/to/dwi/bvals",
       "bvecs": "/path/to/dwi/bvecs",
       "b0_max_bval": 50,
       "transform": "affine,deformable",
       "save_inverse": true,
       "device": "auto"
   }
   ```

3. Run `./main`.

## Troubleshooting

- **no volumes with b-value <=**: Raise b0_max_bval or check the bvals file.
- **expected warp not found**: Make the transform parameter end in a deformable stage, e.g. affine,deformable.

## Citing

If you use this App, please cite it, the software it builds on and brainlife.io:

- anatomix-t1w-dwi-registration. brainlife.io App. [doi:10.25663/brainlife.app.972](https://doi.org/10.25663/brainlife.app.972)
- Avants et al. A reproducible evaluation of ANTs similarity metric performance in brain image registration. *NeuroImage* 54:2033-2044 (2011). [doi:10.1016/j.neuroimage.2010.09.025](https://doi.org/10.1016/j.neuroimage.2010.09.025)
- Hayashi S, Caron BA, Heinsfeld AS, et al. brainlife.io: a decentralized and open-source cloud platform to support neuroscience research. *Nature Methods* 21:809–813 (2024). [doi:10.1038/s41592-024-02237-2](https://doi.org/10.1038/s41592-024-02237-2)
- Avesani P, McPherson B, Hayashi S, et al. The open diffusion data derivatives, brain data upcycling via integrated publishing of derivatives and reproducible open cloud services. *Scientific Data* 6:69 (2019). [doi:10.1038/s41597-019-0073-y](https://doi.org/10.1038/s41597-019-0073-y)

## Authors

Registration is performed by
[anatomix](https://github.com/neel-dey/anatomix) +
[FireANTs](https://github.com/rohitrango/FireANTs) (via
[this fork](https://github.com/neel-dey/FireANTs)). See anatomix's own
[registration README](https://github.com/neel-dey/anatomix/blob/main/anatomix/registration/README.md)
for the underlying method and citations.

- Gabriele Amorosino

## License

See [LICENSE](LICENSE).
