---
title:  "Making pyramidal TIF masks for segmentation"
excerpt: "Convert polygons in GeoJSON to a TIF whole slide image mask."
---

![Image of nuclei labels overlaid on histology.](https://raw.githubusercontent.com/kaczmarj/geojson2tif/main/sample.png)

This post will explain how I created multi-resolution (pyramidal) TIF images of binary
masks for use with histology. **The goal of this**
is to create a whole slide image of segmentations, which can be accessed in the same
way as the original histology image. The segmentation image is the same size (width, length)
as the original histology image, and they have the same physical units (i.e., microns per pixel).

The segmentations can be of many different things, including nuclei, cell boundaries, tumor regions, stroma, etc.

As part of this, I created a command-line tool to convert
a GeoJSON representation of polygons to a binary mask TIF image. Find it here:
https://github.com/kaczmarj/geojson2tif/

The main workhorse is the [ASAP whole slide image viewer](https://computationalpathologygroup.github.io/ASAP/)
and the [`wholeslidedata` Python package](https://github.com/DIAGNijmegen/pathology-whole-slide-data).
These packages implement the conversion of a set of polygons to a whole slide image format.

## GeoJSON

GeoJSON is a powerful format to represent geometries, like polygons. Below I show an
example of GeoJSON with one polygon. This is simply a JSON object that conforms to a schema.
As such, it is highly flexible. You can use common JSON tools (like Python's `json` module in the standard library)
with GeoJSON. Additionally, QuIP is able to consume GeoJSON (just drag and drop into the viewer).

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          [
            [0, 0],
            [1, 0],
            [1, 1],
            [0, 1],
            [0, 0]
          ]
        ]
      }
    }
  ]
}
```

## GeoJSON to TIF

Once we have our data in GeoJSON format, we can use it with other tools. For example,
we can use it directly with the `geojson2tif` command-line tool I wrote. Below, I demonstrate
how to use it with `apptainer` (formerly Singularity).

```
apptainer pull docker://kaczmarj/geojson2tif:latest
apptainer exec --pwd $(pwd) geojson2tif_latest.sif \
    --wsi image.tif --geojson polygons.json --output mask.tif
```

Here is an explanation of the arguments:
- `--wsi image.tif` : the original whole slide image with which the labels are associated.
This will often be a hematoxylin and eosin slide.
- `--geojson polygons.json` : the GeoJSON file with the polygon coordinates. This can
be gzipped.
- `--output mask.tif` : the path of the output TIF image containing the binary mask.


This process can take some time... perhaps 20 to 30 minutes. Perhaps more. Maybe less. But probably more.


## Using the images in a machine learning pipeline

TODO.
