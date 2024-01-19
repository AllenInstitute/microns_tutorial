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

# Ultrastructure Resources

```{figure} img/ultrastructure.png
---
scale: 33%
align: center
---

An EM cutout showing numerous biological structures such as mitochondria, synapses, myelin, nuclei, and endoplasmic reticulum.
```

Electron microscopy allows one to see very high resolution features called the "ultrastructure" of the cell, such as organelles and microtubules.
However, when first seeing EM images, it can be difficult to know the biological meaning of what you are looking at.
Here are a couple of resources with several examples of particular structures imaged with EM:
* [SynapseWeb](https://synapseweb.clm.utexas.edu/tutorials). This website from the Harris lab at UT Austin offers a number of annotated examples of EM images, in particular the [Atlas of Ultrastructural Neurocytology](https://synapseweb.clm.utexas.edu/atlas).
* [Brain Ultrastructure: Putting the Pieces Together](https://www.brainfacts.org/brain-anatomy-and-function/anatomy/2012/brain-ultrastructure-putting-the-pieces-together). This short paper has some particularly clear pictures of less common structures like neurites undergoing autophagy.

However, note that EM imagery can look very different when imaged with different imaging techniques, sectioning methods or staining protocols, and thus the exact appearance can vary between datasets.
In addition, the same structure imaged in different planes can look different, particularly in anisotropic approaches like serial section EM where the x/y resolution is much finer than the z resolution.
