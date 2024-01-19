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

# Dataset Basics

One of the more complicated things about using connectomics data is that it's both highly multimodal — there are images, tables, meshes, and more — and that it has been designed to be dynamic, as the segmentation changes with proofeading and carries changes to synaptic connectivity and more.
This section will cover what types of data exist and how to access them.

The types of data we will describe here are:
- **Imagery**: The 3d array depicting the ultrastructure, with grayscale values.
- **Segmentation**: A 3d array where each voxel has the root id of the segmentation at that location (see next section).
- **Meshes**: A representation of the 3d surface of a neuron.
- **Skeletons** : A representation of the tree-like structure of a neuron.
- **Tables**: Database tables that describe the various types of annotations, such as cell types, synapses, and more.

In addition, there are three primary ways to access the data:
- **Neuroglancer** : Neuroglancer is a web-based viewer to explore the 3d data, and offers access to imagery, segmentations, meshes, and annotations.
Its graphical interface, ability to quickly explore data, and share links make it convenient for fast exploration.
- **Python** : A collection of tools based around the CAVEclient offer a programmatic interface to the data.
- **Dash web apps**: Web apps that allow fast querying of tables and synaptic connectivity that can also be used to generate complex Neuroglancer links.
Not as flexible as programmatic access, but it can be used to quickly explore certain features of the data and visualize in Neuroglancer.

## Dataset Information Hub

If you need to look up any particular values or links for the dataset — how to open a Neuroglancer link for it, what annotation tables exist, how to find Dash Apps —
the public MICrONs data datastack has a [useful information page](https://global.daf-apis.com/info/datastack/minnie65_public). Go there if you need to look up basic details about the dataset.

Particularly useful links here are the [Neuroglancer link](https://neuroglancer.neuvue.io/#!%7B%22jsonStateServer%22:%22https://global.daf-apis.com/nglstate/api/v1/post%22,%22navigation%22:%7B%22pose%22:%7B%22position%22:%7B%22voxelSize%22:%5B4.0,4.0,40.0%5D%7D%7D,%22zoomFactor%22:2.0%7D,%22showSlices%22:false,%22layout%22:%22xy-3d%22,%22perspectiveZoom%22:2000.0,%22layers%22:%5B%7B%22type%22:%22image%22,%22source%22:%22precomputed://https://bossdb-open-data.s3.amazonaws.com/iarpa_microns/minnie/minnie65/em%22,%22name%22:%22img%22,%22shader%22:%22#uicontrol%20float%20black%20slider(min=0,%20max=1,%20default=0.0)%5Cn#uicontrol%20float%20white%20slider(min=0,%20max=1,%20default=1.0)%5Cnfloat%20rescale(float%20value)%20%7B%5Cn%20%20return%20(value%20-%20black)%20/%20(white%20-%20black);%5Cn%7D%5Cnvoid%20main()%20%7B%5Cn%20%20float%20val%20=%20toNormalized(getDataValue());%5Cn%20%20if%20(val%20%3C%20black)%20%7B%5Cn%20%20%20%20emitRGB(vec3(0,0,0));%5Cn%20%20%7D%20else%20if%20(val%20%3E%20white)%20%7B%5Cn%20%20%20%20emitRGB(vec3(1.0,%201.0,%201.0));%5Cn%20%20%7D%20else%20%7B%5Cn%20%20%20%20emitGrayscale(rescale(val));%5Cn%20%20%7D%5Cn%7D%5Cn%22%7D,%7B%22type%22:%22segmentation_with_graph%22,%22source%22:%22graphene://https://minnie.microns-daf.com/segmentation/table/minnie65_public%22,%22name%22:%22seg%22,%22selectedAlpha%22:0.3,%22objectAlpha%22:1.0,%22notSelectedAlpha%22:0.0%7D,%7B%22type%22:%22annotation%22,%22filterBySegmentation%22:false,%22bracketShortcutsShowSegmentation%22:true,%22annotationSelectionShowsSegmentation%22:true,%22name%22:%22ann%22%7D%5D,%22selectedLayer%22:%7B%22layer%22:%22ann%22,%22visible%22:true%7D%7D), the [Dash Apps](https://minnie.microns-daf.com/dash/datastack/minnie65_public), and the [list of all annotation tables](https://minnie.microns-daf.com/materialize/views/datastack/minnie65_public/version/661) associated with the most recent release version (v661).

## How to reference a neuron

The dynamic nature of the data means that there is more than one way to reference a neuron, and each way has a slightly different meaning.

The most frequent way to reference a specific neuron is its **root id**, a unique integer that identifies a specific segmentation, and can be used in numerous places.
However, as proofreading occurs, the segmentation of a cell can change, and each edit creates a new "version" of a given neuron that is given its own unique root id.
Root ids thus refer to not only a specific cell, but a specific version of that cell.
When downloading meshes, querying the database, or browsing in Neuroglancer, we use the root id of a cell.
However, when using data from different points in time, however, root ids can change and the data might not be consistent across two time points.
One way to make this easier is to only use data from one moment in time.
This is the case for the public database that will be used in the course — all data from the public database is synched up to a single moment in time, but it does mean that the analysis cannot respond to any changes in segmentation.
In some cases, in some cases, you might see the root id called a "segment id" or an "object id," but they all refer to the same thing.

For segmentations with a cell body in the volume, we also have a **cell id** (or **soma id**), which is a unique integer that identifies each nucleus in the dataset.
The advantage of the soma id is that it remains constant across all versions of a cell, so it can be used to track a cell across time.
The disadvantage is that to look up the id at any given time or to load into Neuroglancer, you need to look up the root id.
Cell ids can be found in the `nucleus_detection_v0` table in the database, as well as many of the tables that reference cell types.
