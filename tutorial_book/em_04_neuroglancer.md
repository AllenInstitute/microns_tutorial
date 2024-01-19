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

```{code-cell}
:tags: ["remove-cell"]
import os
import caveclient
try:
    os.makedirs('~/.cloudvolume/secrets')
except:
    pass
cglob = caveclient.CAVEclient(global_only=True)
cglob.auth.save_token(token=os.environ.get('API_SECRET'), overwrite=True)
del cglob
```

# Neuroglancer
```{note}
Neuroglancer works best in Chrome and Firefox but does not always work as expected in Safari.
```

```{figure} img/neuroglancer.png
---
align: center
---

Neuroglancer can show multiple layers of data, including imagery, segmentation, and more.
```

[Neuroglancer](https://github.com/google/neuroglancer) is a WebGL-based viewer developed by Jeremy Maitin-Shephard at the Google Connectomics team to visualize very large volumetric data, designed in large part for connectomics.
We often use Neuroglancer to quickly explore data, visualize results in context, and share data.

To look at the MICrONS data in Neuroglancer, [click this link](https://neuroglancer.neuvue.io/#!%7B%22jsonStateServer%22:%22https://global.daf-apis.com/nglstate/api/v1/post%22,%22navigation%22:%7B%22pose%22:%7B%22position%22:%7B%22voxelSize%22:%5B4.0,4.0,40.0%5D%7D%7D,%22zoomFactor%22:2.0%7D,%22showSlices%22:false,%22layout%22:%22xy-3d%22,%22perspectiveZoom%22:2000.0,%22layers%22:%5B%7B%22type%22:%22image%22,%22source%22:%22precomputed://https://bossdb-open-data.s3.amazonaws.com/iarpa_microns/minnie/minnie65/em%22,%22name%22:%22img%22,%22shader%22:%22#uicontrol%20float%20black%20slider(min=0,%20max=1,%20default=0.0)%5Cn#uicontrol%20float%20white%20slider(min=0,%20max=1,%20default=1.0)%5Cnfloat%20rescale(float%20value)%20%7B%5Cn%20%20return%20(value%20-%20black)%20/%20(white%20-%20black);%5Cn%7D%5Cnvoid%20main()%20%7B%5Cn%20%20float%20val%20=%20toNormalized(getDataValue());%5Cn%20%20if%20(val%20%3C%20black)%20%7B%5Cn%20%20%20%20emitRGB(vec3(0,0,0));%5Cn%20%20%7D%20else%20if%20(val%20%3E%20white)%20%7B%5Cn%20%20%20%20emitRGB(vec3(1.0,%201.0,%201.0));%5Cn%20%20%7D%20else%20%7B%5Cn%20%20%20%20emitGrayscale(rescale(val));%5Cn%20%20%7D%5Cn%7D%5Cn%22%7D,%7B%22type%22:%22segmentation_with_graph%22,%22source%22:%22graphene://https://minnie.microns-daf.com/segmentation/table/minnie65_public%22,%22name%22:%22seg%22,%22selectedAlpha%22:0.3,%22objectAlpha%22:1.0,%22notSelectedAlpha%22:0.0%7D,%7B%22type%22:%22annotation%22,%22filterBySegmentation%22:false,%22bracketShortcutsShowSegmentation%22:true,%22annotationSelectionShowsSegmentation%22:true,%22name%22:%22ann%22%7D%5D,%22selectedLayer%22:%7B%22layer%22:%22ann%22,%22visible%22:true%7D%7D).
Note that you will need to authenticate with the same Google-associated account that you use to set up CAVEclient.

## Interface Basics

The Neuroglancer interface is divided into panels.
In the default view, one panel shows the imagery in the X/Y plane (left), one shows a 3d view centered at the same location, and the narrow third panel provides information about the specific layer.
Note that at the center of the each panel is a collection of axis-aligned red, blue and, green lines. The intersection and direction of each of these lines is consistent across all panels.

Along the top left of the view, you can see tabs with different names.
Neuroglancer organizes data into *layers*, where each layer tells Neuroglancer about a different aspect of the data.
The default view has three layers:
* `img` describes how to render imagery.
* `seg` describes how to render segmentation and meshes.
* `ann` is a manual annotation layer, allowing the user to add annotations to the data.

You can switch between layers by *right clicking* on the layer tab.
You will see the panel at the right change to provide controls for each layer as you click it.

The collection of all layers, the user view, and all annotations is stored as a JSON object called the **state**.


The basic controls for navigation are:
* `single click/drag` slides the imagery in X/Y and rotates the 3d view.
* `scroll wheel up/down` moves the imagery in Z.
* `right click` jumps the 3d view to the clicked location in either the imagery or on a segmented object.
* `double click` selects a segmentation and loads its mesh into the 3d view. Double clicking on a selected neuron deselects it.
* `control-scrool` zooms the view under the cursor in or out.
* `z` snaps the view to the closest right angle.

You can paste a position into Neuroglancer by clicking the x, y, z coordinate in the upper left corner and pasting a space or comma-separated list of numbers and hitting enter.
Note that Neuroglancer always works in voxel units, and you can see the resolution of the voxels in the extreme upper left corner.

## Selecting objects

The most direct way to select a neuron is to double click in the imagery to select the object under your cursor.
This will load all the voxels associated with that object and also display its mesh in the 3d view.

To see the list of selected objects, you can select the segmentation tab (right click on the `seg` tab).
Underneath the list of options, there is a list of selected root ids and the color assigned to them in the view.
You can change colors of all neurons randomly by pressing `l` or individually change colors as desired.
In addition, you can press the checkbox to hide a selected object while keeping it in the list, or deselect it by clicking on the number itself.
You can also copy a root id by pressing the clipboard icon next to its number, or copy all selected root ids by pressing the clipboard icon above the list.

This selection list also allows you to select objects by pasting one or more root ids into the text box at the top of the list and pressing enter.

## Annotations

Annotations are stored in an annotation layer.
The default state has an annotation layer called `ann`, but you can always add new annotation layers by command-clicking the `+` button to the right of the layer tabs.

To create an annotation, select the layer (right click on the tab), and then click the icon representing the type of annotation you want to use.
The most basic annotation is a point, which is the icon to the left of the list.
The icon will change to having a green background when selected.

Now if you **control-click** in either the imagery or the 3d view, you will create a point annotation at the clicked location.
The annotation will appear in the list to the right, with its coordinate (in voxels, not nanometers) displayed.
Clicking any annotation in the list will jump to that annotation in 3d space.
Each annotation layer can have one color, which you can change with the color picker to the left of the annotation list.

## Saving and sharing states

Like many other websites that require logins, you cannot simply send your URL ot another person to have them see the view.
Instead, to save the current state and make it available to yourself or others in the future, you need to save the state with the `Share` button at the top right corner.
This will then give you a URL that you can copy and share with others or paste yourself.
A typical sharing URL looks like the following:
```
https://neuroglancer.neuvue.io/?json_url=https://global.daf-apis.com/nglstate/api/v1/4684616269037568
```
The first part is the URL for the Neuroglancer viewer, while the part after the `?json_url=` is a URL that points to a JSON file that contains the state.
The number at the end of the URL is used to uniquely identify the state and can be used programatically to retrieve information.

```{warning}
If a URL contains `?local_id=` instead of `?json_url`, that means that it cannot be viewed by anyone else or even in another browser on your own computer.
```

## Navigating Annotations

Annotations in an annotation layer can be right-clicked on to jump to them, but can also be navigated as a list.

To see the list of annotations, select the annotation layer (right click on the tab).
Each annotation is listed as its location in Neuroglance voxel coordinates.
Clicking on an row in this annotation list will not only jump to it in the view, but also select it.
You can see the information about the selected neuron in the lower right corner.

```{warning}
The following sections are not accurate for spelunker version of neuroglancer, but do represent planned feature addtions.
```
Once an annotation is selected, any associated root ids are loaded.
The keys `[` and `]` will jump to the previous and next annotations in the list, respectively.

Each annotation can have a full-text description associated with it for adding notes.
This can be added in the lower right corner.

```{figure} img/neuroglancer_tags.png
---
align: center
---

Annotation tags after populating them manually.
```

However, the most convenient way to label data quickly is through Tags.
To add tags, click on the `Shortcuts` tab within the Annotation widget on the right, and then click on the `+` button to add a new tag.
Each tag gets a text label and a key command to activate or deactivate it for a given annotation.
By default, the first tag is activated by pressing `shift-q`, the second by pressing `shift-w`, and so on down the qwerty line.

```{figure} img/neuroglancer_tags_example.png
---
align: center
---

Annotation tags applied to annotations. The labels with with the `#` symbol indicate a tag is present.
```

Now when you select an annotation, you can press the key command to attach that tag to it.
Pressing the same key command will remove the tag.
Any number of tags can be added to each annotation.
Together with the `[` and `]` keys to navigate the list, this allows you to quickly label a large number of annotations.

