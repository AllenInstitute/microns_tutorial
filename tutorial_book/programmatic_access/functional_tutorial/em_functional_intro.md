---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: allensdk
---

(em:functional-data)=
# MICrONS Functional Data

The MICrONs mouse went through a battery of functional imaging experiments before be prepared for electron microscopy.
Excitatory neurons were imaged with 2-photon calcium imaging using GCaMP6s.
Because of the size of the volume, different populations of neurons were imaged in different sessions, spanning several days.
Each neuronal {term}`ROI`` from the functional data is uniquely determined based on the combination of image session, scan index, and ROI unit id.
During each imaging session, the mouse was head-fixed and presented a variety of visual stimuli to the left visual field, including both natural movies and synthetic parametric movies.
In addition to functional 2p imaging, the treadmill rotation, left eye position, and left pupil diameter were recorded to provide behavioral context.

## Stimuli and "Digital twin"

The visual stimuli presented to the mouse included many natural movies, as well as parametric stimuli that were designed to emphasize specific visual features, such as orientation and spatial frequency.
Because imaging sessions were performed over several days and imaging time in any one location was limited, comparisons between cells in different sessions are not straightforward.
To get around this, the principle goal of this collection of stimuli was to be used as training data for deep neural networks that were trained to predict the response of each cell to an arbitrary stimulus.
These so-called "digital twins" were designed to be used to probe functional responses to cells to stimuli that were outside the original training set.
More details about this digital twin approach can be found in [Wang et al. 2023](https://www.biorxiv.org/content/10.1101/2023.03.21.533548v2).

As a measure of the reliability of visual responses, a collection of six movies totalling one minute were presented to the mouse ten times per imaging session.
An **oracle score** was computed based on the signal correlation of a given cell to the oracle stimuli across the imaging session, where higher numbers indicate more reliable responses.
Specifically, the oracle score is the mean signal correlation of the response of each presentation to the average of the other nine presentations.


## Coregistration

The process of aligning the location of cells from the functional imaging with the same cells in the EM imaging is called **coregistration**.
It is a challenging problem due to the need for micron-scale alignment of image volumes, despite signifiant differences in the imaging modalities, tissue deformations under different conditions and potential distortions introduced by sample preparation.

The dataset contains two approaches to coregistration, one semi-manual and one fully automatic.
In the semi-manual approach, a transform between the EM data and the 2p data was generated based on fiducual points such as blood vessels and cell bodies.
This transform was then applied to the functional data to identify a location in the EM space, and a human annotator then identified the cell body in the EM data that was a best match to the functional ROI based on location and context.
In the fully automatic approach, blood vessels were segmented in both 2p and EM volumes, and a transform was generated based on matching this 3d structure.
More details can be found in the [MICrONS dataset preprint](https://www.biorxiv.org/content/10.1101/2021.07.28.454025v3).

### Coregistration Quality Metrics

```{figure} img/coreg-metrics.png
---
align: center
---
Cartoon illustrating the coregistration metrics.
```

Two values are available to help assess the quality of the coregistration.
The **residual** indicates the distance between the location of an ROI after transformation to the EM space and the location of the matched cell body in the EM data.
The **score** (or separation score) is the distance between the matched cell body and the nearest other cell body in the EM data.
This attempts to measure how the residual compares with the distance to other potential matches in the data.
Larger values indicate fewer potential matches and therefore a more confident match, in general.

When using the automated coregistration, it is important to filter the data based on assignment confidence.
A guide for this can be found by comparing the subset of cells matched by both the automated and manual coregistration methods.

```{figure} img/coreg-agreement.png
---
align: center
---
Relationship between separation threshold (left) and residual (right) and the accuracy of automated coregistration compared to manual coregistration. Orange curves depict the fraction of cells that remain after filtering out those matches beyond the threshold indicated on the x-axis.
```

### Matching EM to Function

The combination of session index, scan index, and ROI unit id uniquely identifies each ROI in the functional data.
The annotation database contains tables with the results of each of the coregistration methods.
Each row in each table contains the nucleus id, centroid, and root ID of an EM cell as well as the scan/session/unit indices required to match it.
In additoin, the residual and score metrics for each match are provided to filter by quality.
For manual coregistration, the table is called `coregistration_manual_v3` and for automated coregistration, the table is called `apl_functional_coreg_forward_v5`.

Full column definitions can be [found on Annotation Tables page](em:functional-coreg).

## Functional data

A collection of *in silico* model responses of neurons to a variety of visual stimuli has been precomputed.
The data can be found in a collection of files:

:::{list-table}
:header-rows: 1
* - Filename
  - Format
  - Info
* - `nat_movie.npy`
  - `numpy.ndarray`
  - dimensions=`[clip, frame, height, width]`
* - `nat_resp.npy`
  - `numpy.ndarray`
  - dimensions=`[unit, bin]`
* - `nat_unit.csv`
  - importable into `pandas.DataFrame`
  - columns=`[animal_id, scan_session, scan_idx, unit_id, row_idx]`
* - `monet_resp.npy`
  - `numpy.ndarray`
  - dimensions=`[unit, direction, trial]`
* - `monet_dir.npy`
  - `numpy.ndarray`
  - dimensions=`[direction]`
* - `monet_unit.csv`
  - importable into `pandas.DataFrame`
  - columns=`[animal_id, scan_session, scan_idx, unit_id]`
:::

The model response data is organized into two sets of files, one for natural movies and one for Monet stimuli, a type of parametric stimulus that measures orientation tuning.
The `.npy` files can be read with the numpy function `np.load` and the `.csv` files with the pandas function `pd.read_csv`.

```{figure} img/function-stimulus.png
---
align: center
---
Example images from a variety fo the stimuli used to probe functional responses. Natural movies include scenes from cinema, POV nature videos, and rendered 3d scenes. Monet stimuli are a parametic textured stimulus that varies in orientation and spatial frequency of correlated motion.
```

The natural movie data is organized into three files:

* `nat_movie` contains 250 10-sec clips of natural movies. Movies were shown to the model at a resolution of 72 * 128 pixels at 30 hz. This is downsampled from the *in vivo* presentation resolution of 144 * 256 pixels.
* `nat_resp` contains 104,171 units' binned responses to natural movies. Responses were binned into 500 msec non-overlapping bins. This results in 250 clips * 10 sec / 500 msec = 5000 bins, with every binned response corresponding to 15 frames of natural movies.
* `nat_unit` contains the mapping from rows in nat_resp to functional unit keys. Since there is only a single animal for the EM data, use the session, scan, and unit id values to match to the EM data via one of the coregistration tables described above.

The Monet stimulus data is similarly organized into three files:

* `monet_resp` contains 104,171 units' mean responses to Monet stimuli. 120 trials of 16-direction Monet stimuli were shown to the model. The 120 trials were generated with different random seeds.

* `monet_dir` contains the direction (in units of degrees) for the Monet stimulus shown to the model. The order of directions matches the order in the direction dimension of monet_resp. Here, 0 degrees is vertical feature moving leftwards, with degrees increases in the anti-clockwise direction.

* `monet_unit` contains the functional unit keys of the 104,171 units. The order of the unit keys matches the order in the unit dimension of `monet_resp`.  Since there is only a single animal for the EM data, use the session, scan, and unit id values to match to the EM data via one of the coregistration tables described above.
